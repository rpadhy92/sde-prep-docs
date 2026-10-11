# 11 — High-Level Design (System Design)

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **6 h** · 2-month plan **9 h** (see [README](README.md)).

**Learn in this order**
1. **The 6-step interview framework** (Part B) — 30 min, then use it every time
2. **Core building blocks:** load balancer, caching (+ stampede), SQL vs NoSQL, replication/sharding, queues/Kafka, CAP, API design (C1) — 1.5 h
3. **Estimation basics:** QPS from DAU, storage/year, latency numbers (C2) — 30 min
4. **Practice designs:** URL shortener (worked example, C4) → rate limiter → notification system → (2-month) ticket booking, news feed, chat, payments (C3) — 3.5 h (1-month: 3–4 designs; 2-month: 7–8)

**Done when:** you present a design in 45 min with requirements, estimates, diagram, one deep-dive, a failure mode and trade-offs.
**Skip if short on time:** video streaming, geo-indexing, distributed lock/consensus depth, multi-region active-active.

> Two parts: **(A) how to prepare** and **(B) how to approach it in the interview**, then content (building blocks, problems, worked example, question bank).
> Related: [07_NOSQL_CASSANDRA.md](07_NOSQL_CASSANDRA.md) · [08_MICROSERVICES.md](08_MICROSERVICES.md) · [09_KAFKA.md](09_KAFKA.md) · [06_DATABASE_SQL.md](06_DATABASE_SQL.md)
> Legend: 🟡 intermediate · 🔴 senior-depth

---

# PART A — How to Prepare for HLD

## A1. What interviewers evaluate (senior level)
| Signal | Looks like |
|---|---|
| Structured thinking | follows a framework, drives the conversation |
| Requirements | functional + **non-functional** (scale, latency, availability, consistency) |
| Estimation | sensible back-of-envelope math that *drives decisions* |
| Building-block fluency | right use of cache, queue, DB type, LB, CDN, etc. |
| Trade-offs | explains *why X over Y*, states downsides |
| Depth | deep-dives 1–2 components (data model, scaling, consistency, failure) |
| Operability | monitoring, failure handling, deployment, security |
| Communication | diagrams are clear; adapts to hints |

## A2. Preparation roadmap (6 h in the 1-month plan / 9 h in the 2-month plan — the table below is the *ideal order*; compress phases for the 1-month plan)
| Phase | Days | Activity |
|---|---|---|
| 1. Fundamentals | 1–5 | Building blocks (§C1): scaling, LB, caching, DB choice/replication/sharding, queues, CAP, consistent hashing, API design. Make **one-page notes** per block with "when to use / trade-offs / numbers". |
| 2. Estimation skills | 6 | Latency numbers, QPS math, storage math (§C2). Do 5 estimation drills. |
| 3. Classic designs | 7–15 | 1 design per day (§C3): URL shortener → rate limiter → notification → chat → feed → ticket booking → payments … For each: draw diagram, write schema + APIs, identify bottleneck, discuss failures. |
| 4. Read & compare | continuous | After each attempt, compare with a reference (Alex Xu vol 1/2, ByteByteGo, High Scalability); record what you missed. |
| 5. Mocks | 16–21 | 45-min timed mocks, talk aloud, record; ask a peer to interrupt with "what if X fails / 10× traffic?". |

**Practice discipline**
1. Always practice on a **whiteboard / blank diagram tool** (Excalidraw/draw.io), talking aloud.
2. Use the **same framework every time** (Part B).
3. For each design write: *APIs, data model (tables/keys), diagram, scaling plan, failure modes, trade-offs*.
4. Tie to **your own production systems**: "In my service we used X because…". Senior interviews reward lived experience.
5. Learn **numbers** and typical **component limits** (a single Postgres ~ thousands–10k+ QPS depending; Redis ~100k ops/s; Kafka partition ~ tens of MB/s).
6. Prepare 2–3 *real* designs you've done deeply (be ready to whiteboard and defend).

## A3. Pitfalls
- Jumping to boxes (Kafka! Redis! K8s!) before requirements
- Ignoring non-functional requirements/estimates
- Buzzword soup without trade-offs; every problem solved with "add a cache + Kafka"
- Never stating assumptions or checking with the interviewer
- Skipping failure modes (what if DB/queue/region fails?), data consistency, security, observability
- Spending all time on breadth; no deep-dive

---

# PART B — How to Approach HLD in the Interview (45 min Framework)

| Step | Time | What to do / say |
|---|---|---|
| **1. Requirements** | 5 min | **Functional** (core features, what's out of scope), **Non-functional** (users, QPS, latency targets, availability e.g. 99.9%, consistency needs, durability, global/regional, compliance). Ask: *read-heavy or write-heavy? peak vs average? data retention?* Summarize back. |
| **2. Estimation** | 4–5 min | DAU → QPS (avg & peak ≈ 2–10×), read:write ratio, payload size → **storage/yr**, bandwidth, cache size (80/20 rule). Use results to justify decisions (e.g., "10k QPS → need caching/sharding"). |
| **3. API & data model** | 5–7 min | Key endpoints (REST/gRPC) with params & idempotency; entities + schema + **access patterns** → pick DB type & keys/indexes. |
| **4. High-level architecture** | 8–10 min | Client → CDN/LB → API gateway → services → cache → DB / queue / object store; async workers; draw & name flows (write path, read path). Keep services at the right granularity. |
| **5. Deep dive** | 10–12 min | Pick the 1–2 hardest parts (or let interviewer choose): scaling DB (sharding key, replication), caching strategy/invalidation, consistency/ordering, fan-out, hot keys, ID generation, concurrency (seat locking), rate limiting. Show numbers & trade-offs. |
| **6. Bottlenecks, failures, trade-offs** | 5 min | SPOFs, failure scenarios (node/AZ/region, dependency slow), retries/idempotency, backpressure, data loss, monitoring (SLIs/alerts), security, cost, evolution ("what changes at 10×?"). |

### Phrases that signal seniority
- "Before designing, let me confirm the scale and consistency requirements…"
- "Given ~5k writes/s peak and 50:1 read ratio, I'd separate read path with cache + replicas…"
- "Trade-off: strong consistency here costs latency; for a feed we accept eventual consistency, but for payments we need…"
- "Failure mode: if the queue is down, we buffer in the outbox and alert; consumers are idempotent…"
- "Let me deep-dive on the part with the highest risk: …"

### Interaction tips
- Propose **MVP design first**, then scale it up (don't start with Netflix-scale).
- State what you'd **not** do and why.
- When the interviewer pushes ("what if 10× traffic?"), re-run the bottleneck analysis rather than defending blindly.
- Keep the diagram tidy; label data flows & sync/async arrows.

---

# PART C — Content

## C1. Building Blocks (checklist)
- [ ] **Scalability:** vertical vs horizontal, stateless services, session externalization, autoscaling
- [ ] **Load balancing:** L4 vs L7, algorithms (round-robin, least-conn, consistent hashing), health checks, sticky sessions (avoid)
- [ ] **Caching:** client/CDN/app (Redis/Memcached)/DB cache; cache-aside / write-through / write-behind / read-through; TTL, eviction (LRU/LFU), invalidation, **stampede/penetration/avalanche**, hot keys
- [ ] **Databases:** SQL vs NoSQL selection ([06](06_DATABASE_SQL.md), [07](07_NOSQL_CASSANDRA.md)); indexing; **replication** (leader-follower, lag), **sharding** (hash/range/directory; shard key; hotspots; resharding), partitioning, read/write splitting
- [ ] **CAP/PACELC**, consistency models, quorum, eventual consistency; idempotency; exactly-once *effect*
- [ ] **Messaging/streaming:** queues vs logs, Kafka ([09](09_KAFKA.md)), delivery semantics, ordering, DLQ, backpressure, fan-out/fan-in
- [ ] **API design:** REST/gRPC/GraphQL, pagination (cursor), idempotency keys, rate limiting, versioning, long-polling/WebSocket/SSE
- [ ] **Microservice patterns:** gateway, discovery, Saga, Outbox, CQRS, circuit breaker, bulkhead ([08](08_MICROSERVICES.md))
- [ ] **Storage:** object store (S3), block, file; blob + metadata pattern; CDN for static/media; pre-signed URLs
- [ ] **Search:** inverted index (Elasticsearch), indexing pipeline via CDC
- [ ] **Distributed primitives:** consistent hashing, leader election (ZK/etcd/Raft), distributed locks (and fencing), unique ID generation (Snowflake/ULID/UUIDv7), Bloom filters, heartbeats/gossip, consensus basics
- [ ] **Reliability:** timeouts, retries with backoff+jitter, graceful degradation, multi-AZ/region, DR (**RPO/RTO**), backups, chaos testing
- [ ] **Security:** authN/Z (OAuth2/JWT), TLS, secrets, encryption at rest, PII/GDPR, abuse prevention
- [ ] **Observability:** logs/metrics/traces, SLI/SLO/SLA, alerting, dashboards
- [ ] **Deployment:** CI/CD, blue-green/canary, feature flags, containers/K8s, autoscaling, config mgmt

## C2. Estimation Cheat-Sheet
| Item | Rule of thumb |
|---|---|
| Seconds/day | ~86,400 (≈ 10⁵) |
| 1M req/day | ≈ 12 QPS avg; peak ≈ 2–5× |
| 1B req/day | ≈ 12k QPS avg |
| Memory access | ~100 ns · SSD read ~100 µs · HDD seek ~10 ms · same-DC round trip ~0.5 ms · cross-region ~100+ ms |
| Storage units | 1 KB = 10³, 1 MB = 10⁶, 1 GB = 10⁹, 1 TB = 10¹² bytes |
| Typical sizes | tweet/message ~ 200 B–1 KB; image ~ 200 KB–2 MB; video minute ~ 10–50 MB |
| Cache sizing | cache ~20% of hot data (80/20 rule) |
| Availability | 99.9% ≈ 8.8 h downtime/yr; 99.99% ≈ 52 min; 99.999% ≈ 5 min |

**Drill formula:** `QPS = DAU × actions/user/day ÷ 86,400`; `storage/yr = writes/day × size × 365 × replication`.

## C3. Practice Problems
| Level | Problem | Key topics to cover |
|---|---|---|
| 🟡 | **URL Shortener** | ID generation (base62/counter/hash+collision), read-heavy cache, DB choice, redirect latency, analytics, expiry |
| 🟡 | **Rate Limiter (distributed)** | token bucket/sliding window, Redis+Lua, per-user/IP/API, gateway placement, failure mode (fail open/closed) |
| 🟡 | **Notification System** | queues, fan-out, channels (email/SMS/push), retries, dedupe, user prefs, templates, rate limits |
| 🟡 | **Pastebin / File sharing** | object storage + metadata DB, expiry, CDN, pre-signed URLs |
| 🟡 | **Key-Value Store / Cache** | partitioning, consistent hashing, replication, eviction, quorum |
| 🟡 | **Job Scheduler / Task Queue** | delayed jobs, retries, idempotency, leader election, exactly-once-ish |
| 🔴 | **E-commerce (cart, inventory, order, payment)** | consistency, Saga/Outbox, inventory reservation, idempotent payments, search |
| 🔴 | **Ticket Booking (BookMyShow)** | seat locking w/ TTL, high contention, queueing, overbooking prevention |
| 🔴 | **Ride Booking / Food Delivery** | geo-indexing (geohash/H3), matching, real-time location, ETA, surge |
| 🔴 | **News Feed / Timeline** | fan-out on write vs read (hybrid for celebrities), ranking, cache |
| 🔴 | **Chat / Messaging** | WebSockets, message ordering/delivery receipts, presence, storage (Cassandra), group chat |
| 🔴 | **Video Streaming (basics)** | upload pipeline, transcoding, CDN, adaptive bitrate |
| 🔴 | **Payment Gateway / Wallet** | ACID ledger, double-entry, idempotency, reconciliation, PCI considerations |
| 🔴 | **Logging / Metrics Pipeline** | Kafka ingest, stream processing, time-series storage, retention tiers |
| 🔴 | **Search Autocomplete / Typeahead** | trie/prefix index, top-K per prefix, caching, update pipeline |
| 🔴 | **Distributed Lock / Leader election service** | consensus, leases, fencing tokens |

## C4. Worked Example — URL Shortener (template to emulate)
1. **Requirements:** shorten URL; redirect; optional custom alias & expiry; analytics (clicks). NFR: 100M new URLs/month, 100:1 read:write, redirect p99 < 50 ms, 99.99% available, links never lost.
2. **Estimates:** writes ≈ 100M/month ≈ 40 QPS (peak ~ 200); reads ≈ 4k QPS (peak ~ 20k); 5 yrs × 100M × 12 = 6B records × ~500 B ≈ 3 TB → easily shardable; cache top ~20% hot.
3. **API:** `POST /v1/urls {longUrl, alias?, ttl?} → {shortUrl}` (idempotent via client key) · `GET /{code} → 301/302`.
4. **ID generation:** (a) base62 of a **distributed counter/ID range** per app server (no collisions, predictable → add salting if needed), (b) hash(long URL) + collision check, (c) pre-generated key service (KGS). Choose (a)/(c) for no collisions.
5. **Data model:** `urls(code PK, long_url, user_id, created_at, expires_at)` → key-value access pattern → **DynamoDB/Cassandra/sharded SQL** by `code`.
6. **Architecture:** Client → CDN/LB → stateless app → **Redis cache (cache-aside)** → DB; async click events → Kafka → analytics store (ClickHouse/Cassandra).
7. **Deep dive:** read path caching (TTL, hot keys, negative caching), 301 vs 302 (analytics vs caching), shard by `code`, replication, expiry cleanup job, abuse/spam, rate limiting.
8. **Failures/trade-offs:** cache down → DB takes load (protect with rate limit + replicas); counter service SPOF → ID ranges per node; multi-region active-active with conflict-free IDs.

## C5. Question Bank
1. 🟡 How do you scale a service from 1K → 1M users? What breaks first at each stage?
2. 🟡 SQL vs NoSQL for a given use case — justify with access patterns.
3. 🟡 Cache invalidation strategies; how to prevent stampede?
4. 🟡 How do you generate unique IDs in a distributed system? Trade-offs (UUID, Snowflake, DB sequence ranges)?
5. 🟡 Explain consistent hashing and why virtual nodes.
6. 🟡 How do you design idempotent payment/order APIs?
7. 🟡 Fan-out on write vs read — when each? Celebrity problem?
8. 🔴 Saga vs 2PC for order→payment→inventory; failure handling & compensation.
9. 🔴 How do you design for zero data loss and exactly-once-*effect* with Kafka? ([09](09_KAFKA.md))
10. 🔴 How do you handle hot partition/hot key (cache, key salting, splitting, rate limit)?
11. 🔴 Design 99.99% availability — list SPOFs & mitigations (multi-AZ, health checks, redundancy, graceful degradation).
12. 🔴 Multi-region architecture: active-active vs active-passive; data replication & conflict resolution; RPO/RTO.
13. 🔴 How would you migrate a monolith to microservices incrementally? ([08](08_MICROSERVICES.md))
14. 🔴 Real-time updates: polling vs long-polling vs SSE vs WebSocket — trade-offs & scaling WebSockets.
15. 🔴 Prevent overselling/double booking under high concurrency (DB lock, optimistic version, distributed lock + TTL hold, queue).
16. 🔴 Design for traffic spikes (autoscaling, queue buffering, load shedding, rate limiting, caching, pre-warming).
17. 🔴 Cost vs performance trade-offs: where would you save cost without hurting SLOs?

## C6. Self-Check
- [ ] Apply the 6-step framework to a new problem in 45 min with a tidy diagram
- [ ] Do QPS/storage estimates in <3 min and use them in a decision
- [ ] For each of 8 problems: state data model, main bottleneck, and one failure mode + mitigation
- [ ] Present 2 designs from your own work with trade-offs and lessons learned
