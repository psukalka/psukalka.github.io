+++
date = '2025-10-04T23:29:55+05:30'
draft = false
title = 'Memory Full'
+++
On Sunday morning, I got lot of messages that production was down. Users are not able to login. Naturally, I went to Sentry to check if any alerts were raised. And there seemed to be a connection issue with Postgres database. I went to the db dashboard and it flagged that the RDS memory had exhausted. It had 1TB memory provisioned and that was exhausted. To resolve the problem immediately I provisioned more memory to it. 

After few weeks, I looked into the issue yesterday. Where is 1TB memory going ? We have only around 10k users and that shouldn't take so much memory. I started debugging DB stats. 
```
from django.db import connection
def check_db_size():
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT 
                schemaname AS database,
                ROUND(SUM(pg_total_relation_size(schemaname||'.'||tablename)) / 1024 / 1024 / 1024::numeric, 2) AS size_gb
            FROM pg_tables
            WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
            GROUP BY schemaname
            ORDER BY size_gb DESC
        """)
        results = cursor.fetchall()
        for row in results:
            print(f"Schema: {row[0]}, Size: {row[1]} GB")
```

DB size seemed quite normal. It came below 1GB. That is interesting. Claude suggested to check WAL size. 

```
def check_wal_size():
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT 
                pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0')) as wal_size,
                count(*) as wal_files 
            FROM pg_ls_waldir()
        """)
        result = cursor.fetchone()
        print(f"WAL Size: {result}")
```

To my surprise, it showed that 258GB was being hogged by one replication slot that was inactive. That got us more into it. We dug deeper into WAL stats. 

```
def deep_wal_investigation_rds():
    with connection.cursor() as cursor:
        # Check ALL replication slots
        cursor.execute("""
            SELECT 
                slot_name,
                slot_type,
                active,
                xmin,
                restart_lsn,
                pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as retained
            FROM pg_replication_slots
            ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) DESC NULLS LAST
        """)
        print("ALL REPLICATION SLOTS:")
        for row in cursor.fetchall():
            print(f"  {row[0][:30]}... Active={row[2]}, Retained={row[5]}")
        # Check WAL retention settings
        cursor.execute("""
            SELECT 
                name, 
                setting,
                unit
            FROM pg_settings 
            WHERE name IN (
                'wal_keep_segments', 
                'wal_keep_size', 
                'max_wal_size', 
                'min_wal_size',
                'checkpoint_timeout',
                'checkpoint_completion_target',
                'archive_mode',
                'archive_timeout'
            )
            ORDER BY name
        """)
        print("\nWAL CONFIGURATION:")
        for row in cursor.fetchall():
            print(f"  {row[0]}: {row[1]} {row[2] if row[2] else ''}")
        # Check current WAL position
        cursor.execute("""
            SELECT 
                pg_current_wal_lsn() as current_lsn,
                pg_walfile_name(pg_current_wal_lsn()) as current_wal_file
        """)
        wal_pos = cursor.fetchone()
        print(f"\nCURRENT WAL POSITION:")
        print(f"  LSN: {wal_pos[0]}")
        print(f"  File: {wal_pos[1]}")
        # Check oldest WAL required
        cursor.execute("""
            SELECT 
                MIN(restart_lsn) as oldest_required_lsn,
                pg_walfile_name(MIN(restart_lsn)) as oldest_required_file
            FROM pg_replication_slots
            WHERE restart_lsn IS NOT NULL
        """)
        oldest = cursor.fetchone()
        if oldest[0]:
            print(f"\nOLDEST WAL REQUIRED BY SLOTS:")
            print(f"  LSN: {oldest[0]}")
            print(f"  File: {oldest[1]}")
```

From the investigation it came that, current WAL position was `38E/A801DF30` and the oldest required position by replication slots was `34E/8008F00`. So, the oldest WAL was around 40 (hex) segments behind, i.e. 64 segments behind. Each slot occupying around 16GB, total came out around 4TB.

For the context, we have CDC pipeline setup on the RDS using DMS service of AWS. For some reason an old replication had gone down, so the maintainer created another replication slot without cleaning the older one. This resulted in memory being hogged by the inactive replication slot. While we actually used a GB of memory, we were paying for TB of memory due to this cleanup issue.