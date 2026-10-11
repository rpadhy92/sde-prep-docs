# 07 — NoSQL (Types) & Cassandra Deep-Dive

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **3 h** · 2-month plan **5 h** (see [README](README.md)).

**Learn in this order**
1. **Why/when NoSQL + the 4 types** with one example each (§1, §3 table) — 30 min
2. **CAP, eventual consistency, quorum (`R + W > RF`), consistent hashing** (§2) — 45 min
3. **Cassandra core:** masterless ring, partition vs clustering key, replication factor, write path (commit log → memtable → SSTable) (§5) — 45 min
4. **Cassandra data modeling:** query-first, one table per query, bucketing; consistency levels (§6) — 1 h
5. (2-month) **Redis patterns** (cache-aside, rate limiter, cache problems) + Mongo embed vs reference (§4) — 1–2 h

**Done when:** you pick a store for 5 scenarios with reasons, and model a Cassandra table from a list of queries in 5 minutes.
**Skip if short on time:** compaction strategies detail, LWT/Paxos, Mongo sharding, Redis Cluster slots, graph/OLAP stores.

> RDBMS fundamentals → [06_DATABASE_SQL.md](06_DATABASE_SQL.md) · Design context → [11_HLD_SYSTEM_DESIGN.md](11_HLD_SYSTEM_DESIGN.md)
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Day 1:** NoSQL landscape + CAP/consistency theory (§1–2). Be able to pick the right store for a scenario and justify.
2. **Day 2:** MongoDB + Redis essentials (§3–4) — 1 hour hands-on each (Docker).
3. **Day 3–4:** **Cassandra** (§5–6): architecture, **query-first data modeling**, consistency. Run a 3-node cluster in Docker Compose; create tables; test `QUORUM` reads/writes.
4. **Day 5:** Spring Data Cassandra/Mongo/Redis (§7), question bank (§8).
Time: 3 h (1-month: ⭐ box only) / 5 h (2-month: add Redis/Mongo §4 and the Docker Cassandra lab).

---

## 1. Why NoSQL & When
| Reason | Explanation |
|---|---|
| Horizontal scale | Built to partition/shard across commodity nodes |
| Flexible schema | Evolving documents / sparse attributes |
| Specialised access pattern | key-value lookups, time-series, graph traversals |
| High write throughput / availability | Cassandra-style masterless, tunable consistency |
| Trade-off | Weak/absent joins, limited ad-hoc queries, no/limited multi-row ACID, **you model data around queries** |

**Answer pattern for "SQL or NoSQL?":** list access patterns, consistency needs, scale (QPS/data size), query flexibility, transactions, team skill → choose; often **polyglot persistence** (RDBMS for transactions + Redis cache + Elasticsearch search + Cassandra for time-series/events).

## 2. Theory: CAP, PACELC, Consistency
- [ ] **CAP:** under a network **P**artition, choose **C**onsistency or **A**vailability. (CP: HBase, MongoDB (default primary), Zookeeper; AP: Cassandra, DynamoDB (default), Couchbase). Don't say "pick 2 of 3" — partitions are inevitable; the choice is during partitions.
- [ ] **PACELC:** *else* (no partition) trade **L**atency vs **C**onsistency.
- [ ] Consistency models: strong, eventual, causal, read-your-writes, monotonic reads
- [ ] **Quorum:** with replication factor `RF`, read `R` + write `W` replicas; **R + W > RF ⇒ strong consistency** (reads see latest write)
- [ ] Replication: leader-follower vs leaderless; conflict resolution (last-write-wins, vector clocks, CRDTs)
- [ ] Partitioning: hash vs range; **consistent hashing** (+ virtual nodes); rebalancing; hot partitions
- [ ] Gossip protocol, anti-entropy/repair, hinted handoff, Merkle trees (Cassandra/Dynamo-style)

## 3. NoSQL Types
| Type | Examples | Data model | Strengths | Typical use | Watch-outs |
|---|---|---|---|---|---|
| **Key-Value** | Redis, DynamoDB, Riak | key → opaque value | O(1) get/put, very fast, simple | cache, sessions, counters, rate limits, leaderboards | no rich queries |
| **Document** | MongoDB, Couchbase, Firestore | JSON/BSON docs in collections | flexible schema, rich queries, nested data | catalogs, CMS, profiles, user content | joins weak (`$lookup`), 16 MB doc limit, multi-doc tx cost |
| **Wide-column** | **Cassandra**, HBase, ScyllaDB, Bigtable | rows with partition key + clustering columns; sparse columns | massive write throughput, linear scale, multi-DC | time-series, IoT, messaging, activity feeds, event logs | query-first modeling, no joins/ad-hoc, tombstones |
| **Graph** | Neo4j, Neptune | nodes + edges | relationship traversal | social graph, fraud, recommendations | scaling writes, niche |
| **Search / Time-series (related)** | Elasticsearch/OpenSearch, InfluxDB, TimescaleDB | inverted index / time buckets | full-text, aggregations / metrics | search, logs, metrics | not a primary source of truth (ES) |
| **Column-oriented OLAP (related)** | ClickHouse, BigQuery, Redshift | columnar storage | analytics scans | reporting | not for OLTP |

## 4. MongoDB & Redis Essentials
### MongoDB
- [ ] Document model; **embed vs reference** (embed for 1-few & read together; reference for many/unbounded/shared)
- [ ] Indexes: single, compound (ESR rule: Equality, Sort, Range), TTL, text, geospatial; `explain()`
- [ ] Aggregation pipeline (`$match`, `$group`, `$lookup`, `$project`, `$sort`); schema design patterns (bucket, outlier, subset)
- [ ] Replica sets (primary/secondary, elections, read/write concern), sharding (shard key choice, chunks, balancer), multi-document transactions (cost)
### Redis
- [ ] Data structures: String, Hash, List, Set, **Sorted Set (leaderboards)**, Bitmap, HyperLogLog, Stream, Geo
- [ ] Single-threaded command execution (+ IO threads), in-memory, persistence (RDB snapshot vs AOF), replication, **Sentinel** vs **Cluster** (16384 hash slots)
- [ ] TTL & eviction policies (`allkeys-lru`, `volatile-ttl`, …); memory fragmentation
- [ ] **Patterns:** cache-aside, write-through/behind, distributed lock (`SET NX PX`, Redlock debate, fencing tokens), rate limiter (INCR + EXPIRE / sliding window with ZSET / token bucket Lua), pub/sub, streams
- [ ] **Cache problems:** stampede (mutex/early refresh/jitter), penetration (cache nulls / Bloom filter), avalanche (randomized TTL), hot key, stale data/invalidation, big keys

## 5. Cassandra — Architecture
- [ ] **Masterless (peer-to-peer) ring:** every node equal; no single point of failure; linear scalability; multi-datacenter replication.
- [ ] **Partitioner:** hashes the **partition key** (Murmur3) to a token; token ranges owned by nodes; **vnodes** for even distribution.
- [ ] **Replication factor (RF)** and **replication strategy:** `SimpleStrategy` (dev) vs `NetworkTopologyStrategy` (prod, per-DC RF e.g. 3).
- [ ] **Coordinator node:** any node receiving the request; forwards to replicas; snitch (topology awareness).
- [ ] **Write path:** client → coordinator → replicas: **commit log** (sequential append, durability) + **memtable** (in-memory sorted) → ack. Memtable flush → immutable **SSTable** on disk. **No read-before-write**, so writes are very fast.
- [ ] **Read path:** memtable + row cache/key cache → **Bloom filter** (skip SSTables that don't contain key) → partition index/summary → SSTable(s) → **merge** by timestamp. Reads may touch multiple SSTables → **compaction** keeps this in check.
- [ ] **Compaction:** merges SSTables, drops tombstones/overwritten data. Strategies: **STCS** (size-tiered; write-heavy), **LCS** (leveled; read-heavy, low read amplification), **TWCS** (time-window; time-series with TTL).
- [ ] **Tombstones:** deletes/TTL expiry write markers; retained for `gc_grace_seconds` (default 10 days) so deletes propagate → **too many tombstones = slow reads/timeouts** (anti-patterns: queue-like deletes, frequent nulls).
- [ ] **Anti-entropy:** read repair, hinted handoff (store writes for downed node, up to hint window), **`nodetool repair`** (Merkle trees) run regularly (within `gc_grace_seconds`).
- [ ] **Gossip** for cluster membership/failure detection; **seed nodes**.
- [ ] **Lightweight transactions (LWT):** `INSERT … IF NOT EXISTS`, `UPDATE … IF` via Paxos — linearizable, ~4× slower, use sparingly.
- [ ] Batches (logged vs unlogged) are for **atomicity across tables for one entity**, not performance; counters; collections (`list/set/map`), UDTs, `TTL`, materialized views (experimental caution), secondary indexes (avoid except low-cardinality + partition-restricted), SAI (newer).

## 6. Cassandra — Data Modeling (the heart of interviews)
### Principles
1. **Query-first design:** list your queries → design **one table per query pattern**. Denormalize & duplicate data; no joins.
2. **Primary key = partition key + clustering columns.**
   - **Partition key** → which node/partition (data distribution & single-partition query).
   - **Clustering columns** → sort order *within* partition (enables range queries/`ORDER BY` on clustering).
3. **Queries must specify the full partition key** (equality). Avoid `ALLOW FILTERING` & multi-partition scans.
4. **Partition sizing:** aim < ~100 MB / < ~100k rows per partition; avoid **unbounded partitions** → add a **bucket** (e.g., day/month) to the partition key. Avoid **hot partitions** (single key receiving most traffic).
5. Write-optimised: duplicates across tables are fine; keep them in sync from app (or logged batch/event stream).

### Example: user's recent orders (query: "latest N orders for a user")
```sql
CREATE KEYSPACE shop WITH replication =
  {'class': 'NetworkTopologyStrategy', 'dc1': 3};

CREATE TABLE shop.orders_by_user (
  user_id    uuid,
  order_ts   timeuuid,          -- clustering → time-ordered
  order_id   uuid,
  status     text,
  total      decimal,
  PRIMARY KEY ((user_id), order_ts)
) WITH CLUSTERING ORDER BY (order_ts DESC);

-- Query
SELECT * FROM shop.orders_by_user WHERE user_id = ? LIMIT 20;
```

### Example: time-series with bucketing (avoid unbounded partition)
```sql
CREATE TABLE metrics_by_device_day (
  device_id  text,
  day        date,                   -- bucket
  ts         timestamp,
  value      double,
  PRIMARY KEY ((device_id, day), ts) -- composite partition key
) WITH CLUSTERING ORDER BY (ts DESC)
  AND compaction = {'class': 'TimeWindowCompactionStrategy'}
  AND default_time_to_live = 2592000;  -- 30 days
```

### Same data, different query → different table
`orders_by_user` (user → orders) · `orders_by_id` (order_id → details) · `orders_by_status_day` (status+day → orders). Write to all three.

### Consistency levels (per request)
| Level | Meaning | Use |
|---|---|---|
| `ONE` / `LOCAL_ONE` | 1 replica | fastest, lowest consistency |
| `QUORUM` / `LOCAL_QUORUM` | ⌊RF/2⌋+1 replicas | **common default for strong-ish consistency** (`R+W>RF` ⇒ strong) |
| `ALL` | all replicas | strongest, least available |
| `ANY` (writes) | even hinted handoff | max availability, risky |
| `SERIAL`/`LOCAL_SERIAL` | LWT reads/Paxos | linearizable |

With RF=3: `QUORUM` write (2) + `QUORUM` read (2) → 4 > 3 ⇒ read-your-writes.

### Cassandra anti-patterns
- Using it like an RDBMS (joins, normalization, ad-hoc queries, secondary indexes everywhere)
- Unbounded/hot partitions; huge collections; large rows/blobs
- Queue pattern (insert + delete) → tombstone storms
- `ALLOW FILTERING`, `SELECT COUNT(*)` on big tables, `IN` with many partitions
- Skipping repairs; mismatched `gc_grace_seconds`; wrong compaction strategy; `SimpleStrategy` in prod
- Large unlogged batches across partitions
- Strong cross-row transactions/need ACID → use RDBMS

### When to choose Cassandra
✔ write-heavy, time-series/event logs/IoT, messaging/activity feeds, high availability across DCs, predictable key-based queries, huge scale
✘ complex ad-hoc queries/joins, strong multi-row transactions, small data sets, heavy analytics/aggregations

## 7. Spring Integration (quick)
- [ ] `spring-boot-starter-data-cassandra` (`CassandraTemplate`/`CassandraRepository`, `@Table`, `@PrimaryKeyColumn(type=PARTITIONED/CLUSTERED)`), or DataStax Java driver directly (prepared statements, async, token-aware routing, retry/speculative execution policies, `LOCAL_QUORUM`, local datacenter config)
- [ ] `spring-boot-starter-data-mongodb`, `spring-boot-starter-data-redis` (`RedisTemplate`, `@Cacheable` with Redis cache manager, Lettuce client)
- [ ] Testing with Testcontainers (Cassandra/Mongo/Redis) — same pattern as the Postgres `@DataJpaTest` example in [05 Part D](05_JUNIT_MOCKITO.md) (`@Container` + `@ServiceConnection`, module-specific container class)
- [ ] Idempotent writes (Cassandra upserts are naturally idempotent) → pairs well with at-least-once Kafka consumers ([09](09_KAFKA.md))

## 8. Question Bank
### NoSQL general
1. 🟢 SQL vs NoSQL — differences; four NoSQL families with examples?
2. 🟡 Explain CAP & PACELC with Cassandra/MongoDB as examples.
3. 🟡 What does `R + W > N` give you? Eventual vs strong consistency?
4. 🟡 MongoDB: embed vs reference; compound index design (ESR)?
5. 🟡 Redis: data structures & when to pick each; persistence modes; cache stampede mitigation?
6. 🔴 Design a distributed rate limiter on Redis; discuss atomicity (Lua) & failure modes.
7. 🔴 Consistent hashing — why/how (virtual nodes, rebalancing)?
8. 🔴 How would you migrate from RDBMS to NoSQL for part of your data without downtime (dual-write/CDC, backfill, verify, cut-over)?

### Cassandra
9. 🟢 What is Cassandra? Architecture in brief (masterless ring, RF, partition key).
10. 🟡 Partition key vs clustering key; what decides which node stores a row?
11. 🟡 Explain the **write path** and why writes are fast (commit log + memtable + SSTable; append-only).
12. 🟡 Explain the **read path** & role of Bloom filters / compaction.
13. 🟡 Consistency levels; what is `LOCAL_QUORUM` and why used in multi-DC?
14. 🟡 Why no joins & why denormalize? How do you model "orders by user" *and* "orders by status"?
15. 🟡 What are tombstones; why can they hurt; `gc_grace_seconds`?
16. 🔴 Hot partition / unbounded partition — detect and fix (bucketing, salting, better key)?
17. 🔴 STCS vs LCS vs TWCS — choose for time-series with TTL vs read-heavy workloads?
18. 🔴 Why/when run `nodetool repair`; hinted handoff vs read repair vs anti-entropy repair?
19. 🔴 LWT/Paxos — what & cost; alternatives for uniqueness/conditional updates?
20. 🔴 Design a chat/messaging or IoT time-series store on Cassandra (tables, keys, TTL, compaction, consistency).
21. 🔴 A Cassandra read query is timing out — step-by-step diagnosis (tombstones, partition size, compaction backlog, GC, wrong consistency, `ALLOW FILTERING`, node health via `nodetool tpstats/cfstats`).
22. 🔴 Why avoid secondary indexes & `ALLOW FILTERING`? What's the alternative?
23. 🔴 How does Cassandra stay available when nodes fail; what do clients (token-aware driver, retry policies) do?

## 9. Practice Tasks
- [ ] Spin up Cassandra (Docker, 3 nodes); create `orders_by_user`; write/read at `ONE` and `QUORUM`; kill a node, observe
- [ ] Model 3 query patterns for a chat app (messages by conversation, conversations by user, unread counts)
- [ ] Design bucketing for a high-volume time-series; justify partition size math
- [ ] Redis: implement rate limiter (INCR/EXPIRE then Lua token bucket) and cache-aside with stampede protection
- [ ] MongoDB: model a blog with embed/reference; add compound index; read `explain()`
- [ ] Write a one-page "SQL vs NoSQL decision" cheat-sheet for your domain

## 10. Self-Check
- [ ] Pick the right store for 5 scenarios and justify with access patterns
- [ ] Model a Cassandra table from a list of queries (key choice, ordering, bucketing) in 5 min
- [ ] Explain the write/read paths and consistency (`R+W>RF`) without notes
- [ ] List 5 Cassandra anti-patterns and their fixes
