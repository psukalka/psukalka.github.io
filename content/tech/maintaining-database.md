+++
date = '2025-05-28T23:53:13+05:30'
draft = false
title = 'Maintaining A Database'
+++
We use Django Framework in our system. Django comes with its own ORM for relational databases. There are few tables which are being used from 7+ years now, so we decided to look into them. 

To check how much space a table is using, we can run following Postgres query: 
```
     SELECT
        schemaname,
        tablename,
        pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
        pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
        pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) as indexes_size
    FROM pg_tables
    WHERE tablename = '{table_name}';
```

It turned out that the table is using 1.5GB space and its indexes are using 6.5GB space. This came as a surprise to me as I was always under the impression that indexes are smaller than table. Yes, indeed they are smaller but how many indexes you create also matters. It turns out that we had 50+ indexes on our table. Obviously not all of them are being used. So, I checked indexes and their usage with this query: 
```
    SELECT
        indexrelname as indexname,
        idx_tup_read as reads,
        idx_tup_fetch as fetches,
        pg_size_pretty(pg_relation_size(indexrelid)) as size,
        CASE
            WHEN idx_tup_read = 0 THEN 'UNUSED'
            WHEN idx_tup_read < 100 THEN 'RARELY USED'
            ELSE 'ACTIVELY USED'
        END as status
    FROM pg_stat_user_indexes
    WHERE relname = '{table_name}';
```

Obviously it turned out that more than half of the indexes were such that we didn't use them. So, they became candidate to drop indexes. To drop the index, we can use 
```
DROP INDEX CONCURRENTLY {index_name};
```
This would drop index without blocking the table. 

For Django, I just had to remove `db_index=True` for these fields and indexes got removed on running migration.

Another interesting thing was that multiple indexes were created for some field. For ex, `slug` field had 3 indexes. One regular index, one index with `_like` prefix and one index with `_key` prefix. Turns out that for `CharField` fields, 2 indexes are created. One for exact match and one for `like` queries. The reason is something related to `byte-comparison` and `collation-comparison` (based on language rules). You can google further. The third index `_key` was created due to `unique=True` constraint. 

After cleaning up the table (by removing about 15+ indexes) I again checked the size. But surprisingly the size didn't decrease even by 1kB. I ran `VACUUM ANALYZE` too hoping that it would reclaim the space. But that didn't happen. Turns out that Postgres doesn't ever return space back to OS until you run the `VACUUM FULL` (which would mean downtime). But no need to worry, all next insertions in the table will use the recently cleared space. So, table size won't increase for some time till the cleared space is used. 

In the end, the more indexes that you create the more expensive your write becomes. Over the years we had added around 50+ indexes on the table so each write would have to go and update these indexes (meaning 50+ writes). So, if writes become slow it is the database that we blame rather than understanding our follies. 