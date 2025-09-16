+++
date = '2025-09-16T10:32:51+05:30'
draft = false
title = 'Redis Valkey'
+++
Redis used to be open source with BSD license. Berkeley Software Distribution (BSD) license gives full permission to use the software. It is like `do whatever you want with this code, just don't blame us if it doesn't work`.

Cloud providers used Redis and made lot of money over it, but nothing was shared back to Redis Ltd. 
Hence, they moved to a more restrictive licensing on March 2024. Their source code is still available but not for commercial hosting / service use. For commercial use you need to take their permission. 

Valkey is a fork of Redis. Valkey is open source. Till now they both are almost similar but going forward they will diverge. Redis will be managed by Redis Ltd and Valkey will be managed by the Linux Foundation with backing from cloud providers (AWS, Google, Oracle, etc.) Thus, features in Valkey will be community driven. 
Since Valkey is open source it is cheaper (on cloud).

Currently migrating from Redis to Valkey is trivial: 
```
# Before
connection = redis.connect("redis://localhost:6379")

# After  
connection = redis.connect("valkey://localhost:6379")
```

Valkey and redis can be swapped with minimal differences:
```
# Redis
redis-server
redis-cli

# Valkey  
valkey-server
valkey-cli

# Commands, data structures, protocols - all the same (for now)
```

If you are using Redis on cloud, switching to valkey is a cheaper alternative as of now.