---
title: "Day 29 - Amazon ElastiCache"
weight: 4
chapter: false
pre: "<b> 1.6.4. </b>"
---

**Date:** 2025-10-16 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## Amazon ElastiCache

Managed in-memory caching service for **Redis** and **Memcached** to reduce latency and offload databases.

- Microsecond reads, Multi-AZ with failover, simple scaling, encryption/auth, automated ops.
- **Redis**: rich data structures, backups, replication, cluster mode.
- **Memcached**: simple, horizontally scalable cache with auto-discovery.

Common uses: web/mobile acceleration, DB query caching, session stores, leaderboards, pub/sub, queues.

**ElastiCache for Redis Features:**

- **Data Structures:** Strings, lists, sets, sorted sets, hashes, bitmaps, hyperloglogs
- **Persistence:** Snapshots and AOF (Append-Only File)
- **Replication:** Primary-replica with automatic failover
- **Cluster Mode:** Partition data across multiple shards
- **Pub/Sub:** Real-time messaging
- **Lua Scripting:** Server-side scripting
- **Geospatial:** Location-based queries

**ElastiCache for Memcached Features:**

- **Multi-threaded:** Utilize multiple cores
- **Auto Discovery:** Automatic node discovery
- **Horizontal Scaling:** Add/remove nodes easily
- **Simple:** Easy to use, no persistence

**Redis vs Memcached:**

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data Structures | Rich (lists, sets, etc.) | Simple (key-value) |
| Persistence | Yes | No |
| Replication | Yes | No |
| Multi-AZ | Yes | No |
| Backup/Restore | Yes | No |
| Pub/Sub | Yes | No |
| Multi-threaded | No | Yes |

---

## Caching Strategies

### Cache-Aside (Lazy Loading)

- Application checks cache first
- On miss, load from database and populate cache
- **Pros:** Only requested data is cached
- **Cons:** Cache miss penalty, stale data possible

### Write-Through

- Write to cache and database simultaneously
- **Pros:** Data always fresh, no cache misses on reads
- **Cons:** Write penalty, unused data may be cached

### Write-Behind (Write-Back)

- Write to cache immediately, async write to database
- **Pros:** Fast writes, reduced database load
- **Cons:** Risk of data loss, complexity

---

## Use Cases

**Session Store:**

```python
# Store user session in Redis
redis.setex(f"session:{user_id}", 3600, session_data)

# Retrieve session
session = redis.get(f"session:{user_id}")
```

**Leaderboard:**

```python
# Add score to sorted set
redis.zadd("leaderboard", {user_id: score})

# Get top 10
top_10 = redis.zrevrange("leaderboard", 0, 9, withscores=True)
```

**Rate Limiting:**

```python
# Increment counter with expiry
pipe = redis.pipeline()
pipe.incr(f"rate:{user_id}")
pipe.expire(f"rate:{user_id}", 60)
count = pipe.execute()[0]

if count > 100:
    raise RateLimitExceeded()
```

---

# **Hands-On Labs**

## Lab 43 – AWS Database Migration Service (DMS) (Part 3)

12. Inspect S3 → *43-12*  
13. Create Serverless Migration → *43-13*  
14. Create Event Notification → *43-14*  
15. Logs → *43-15*  
16. Troubleshoot: Memory Pressure → *43-16*  
17. Troubleshoot: Table Error → *43-17*
