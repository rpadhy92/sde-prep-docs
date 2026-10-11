# 09 — Apache Kafka

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **4 h** · 2-month plan **6 h** (see [README](README.md)).

**Learn in this order**
1. **Core concepts:** topic, partition, offset, broker, consumer group; Kafka vs RabbitMQ (§1) — 30 min
2. **Internals:** replication, leader/ISR, ordering only within a partition, key → partition (§2) — 45 min
3. **Producer:** `acks`, retries, idempotent producer, batching (§3) — 45 min
4. **Consumer:** groups, offsets & commit strategy, rebalancing, lag (§4) — 1 h
5. **Delivery semantics:** at-most/at-least/exactly-once + **idempotent consumer** (§5) — 45 min
6. **Failure handling:** retries, `@RetryableTopic`, DLT (§6) — 45 min

**Done when:** you state a no-data-loss config (RF=3, `min.insync.replicas=2`, `acks=all`), explain per-key ordering, and diagnose growing lag with a checklist.
**Skip if short on time:** Kafka Streams/Connect, MirrorMaker, KRaft internals, transactions detail (know what exactly-once covers and doesn't).

> Related: [08_MICROSERVICES.md](08_MICROSERVICES.md) (Outbox/Saga) · [04_SPRING_BOOT.md](04_SPRING_BOOT.md) · [05_JUNIT_MOCKITO.md](05_JUNIT_MOCKITO.md) (testing)
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Day 1:** concepts & architecture (§1–2). Draw topic → partitions → replicas → consumer group from memory.
2. **Day 2:** producer deep-dive (§3) — run a local Kafka (Docker/KRaft), produce with different `acks`, keys; observe partition assignment.
3. **Day 3:** consumer deep-dive (§4) — groups, rebalances, offset commit modes; kill consumers and watch rebalancing.
4. **Day 4:** delivery semantics, idempotency, transactions, error handling/retry/DLT (§5–6) — implement in Spring Kafka.
5. **Day 5:** operations, design scenarios, question bank (§7–10).
Time: 4 h (1-month: ⭐ box + one hands-on producer/consumer) / 6 h (2-month: add §7–8 scenarios and Testcontainers tests). Always explain *trade-offs* (throughput vs latency vs durability).

---

## 1. Why Kafka & Core Concepts
- [ ] **Distributed, partitioned, replicated commit log**; durable (disk), high-throughput (sequential I/O, batching, zero-copy), horizontally scalable; **pull-based** consumers; messages retained by time/size (not deleted on consume).
- [ ] **Kafka vs traditional MQ (RabbitMQ/SQS):** log retention & **replay**, consumer groups with offsets, ordering per partition, very high throughput; vs RabbitMQ's smart broker/routing, per-message ack, priority queues.
- [ ] **Use cases:** event-driven microservices, log/metrics pipelines, CDC, stream processing, async integration, audit/event sourcing, buffering spikes.
- [ ] **Terminology:** broker, cluster, **topic**, **partition**, **offset**, record (key, value, headers, timestamp), producer, consumer, **consumer group**, leader/follower replica, ISR, controller, **KRaft** (ZooKeeper-less, default in new versions).

## 2. Architecture Internals
- [ ] **Topic → partitions** (unit of parallelism & ordering). Ordering guaranteed **only within a partition**.
- [ ] **Replication:** each partition has 1 **leader** + followers (replication factor, typ. 3). Producers/consumers talk to leader (follower fetch optional). **ISR** (in-sync replicas) set; **high watermark** = latest offset replicated to all ISR (consumers read up to it).
- [ ] **Durability settings:** `replication.factor=3`, `min.insync.replicas=2`, producer `acks=all` → tolerate 1 broker loss without data loss; with `unclean.leader.election.enable=false`.
- [ ] **Storage:** partition = ordered **segments** on disk (+ index/timeindex); retention (`retention.ms`/`bytes`); **log compaction** (keep latest value per key — changelog/state topics, `cleanup.policy=compact`).
- [ ] **Why fast:** sequential I/O, page cache, batching, compression, zero-copy `sendfile`.
- [ ] **Controller** election, leader election on failure; **KRaft** quorum.
- [ ] **Partition count considerations:** max consumer parallelism = #partitions; more partitions = more throughput but more open files/leader elections/memory; can increase (not decrease) → changes key→partition mapping (breaks per-key ordering for existing data).
- [ ] **Partitioner:** key present → `hash(key) % numPartitions` (murmur2); no key → sticky/round-robin batches. **Same key → same partition → ordering per key**. Hot key → hot partition.

## 3. Producer
- [ ] Flow: `send()` → serializer → partitioner → **batch** (per partition, `batch.size`, `linger.ms`) → compression (`compression.type=lz4/snappy/zstd`) → network → broker ack.
- [ ] **`acks`:** `0` (fire & forget), `1` (leader only), **`all`** (all ISR; with `min.insync.replicas`) — durability vs latency.
- [ ] **Retries & ordering:** `retries`, `delivery.timeout.ms`, `max.in.flight.requests.per.connection` (≤5 with idempotence keeps ordering).
- [ ] **Idempotent producer** (`enable.idempotence=true`, default in recent versions): PID + sequence numbers dedupe producer retries per partition/session.
- [ ] **Transactions:** `transactional.id`, atomic writes across partitions/topics + consumer offset commit (**exactly-once** for consume-process-produce within Kafka); consumers use `isolation.level=read_committed`.
- [ ] Callbacks & error handling: `send().get()` vs callback; `RetriableException` vs fatal; `buffer.memory`, `max.block.ms`.
- [ ] Serialization: String/JSON/**Avro/Protobuf** + **Schema Registry** (compatibility: backward/forward/full); avoid Java serialization.
- [ ] Message design: key choice (aggregate ID), small payloads (reference large blobs), headers (trace ID), event envelope/versioning.

```java
// Spring Kafka producer
@Service
class OrderEventPublisher {
    private final KafkaTemplate<String, OrderCreated> kafka;
    OrderEventPublisher(KafkaTemplate<String, OrderCreated> kafka) { this.kafka = kafka; }

    void publish(OrderCreated e) {
        kafka.send("orders.created", e.orderId(), e)          // key = orderId → ordering per order
             .whenComplete((res, ex) -> {
                 if (ex != null) log.error("publish failed {}", e.orderId(), ex);
             });
    }
}
```
```yaml
spring:
  kafka:
    producer:
      acks: all
      properties: { enable.idempotence: true, linger.ms: 10, compression.type: lz4 }
```

## 4. Consumer
- [ ] **Consumer group:** partitions divided among consumers in a group (each partition → exactly one consumer in the group); different groups each get all messages (pub-sub); consumers > partitions ⇒ idle consumers.
- [ ] **Offsets:** stored in `__consumer_offsets`; **committed offset = next message to read**. Commit modes: auto (`enable.auto.commit` — risky), **manual** after processing (`ack-mode=MANUAL`/`RECORD`/`BATCH` in Spring), sync vs async.
- [ ] **`auto.offset.reset`:** `earliest` / `latest` / `none` (applies when no committed offset).
- [ ] **Rebalancing:** triggered by join/leave/crash/`max.poll.interval.ms` exceeded/partition change. Eager (stop-the-world) vs **cooperative sticky** assignor; **static membership** (`group.instance.id`) to avoid rebalances on restarts. Rebalance storms = slow processing exceeding `max.poll.interval.ms` → reduce `max.poll.records`, offload work, tune timeouts.
- [ ] Key configs: `max.poll.records`, `max.poll.interval.ms`, `session.timeout.ms`, `heartbeat.interval.ms`, `fetch.min.bytes`, `fetch.max.wait.ms`, `isolation.level`.
- [ ] **Concurrency:** one consumer = one thread per instance (KafkaConsumer isn't thread-safe); scale by partitions: `concurrency` in Spring listener container, more instances; for heavy processing → worker pool but manage offset commits carefully.
- [ ] **Lag:** `log-end-offset − committed offset`; monitor (Burrow, Kafka Exporter, Cruise Control, Confluent Control Center); lag growth → consumers too slow/stuck.
- [ ] **Ordering:** per partition only; parallel processing within a partition breaks it.

```java
@KafkaListener(topics = "orders.created", groupId = "inventory-service", concurrency = "3")
public void onOrderCreated(ConsumerRecord<String, OrderCreated> rec, Acknowledgment ack) {
    if (processed.exists(rec.value().eventId())) { ack.acknowledge(); return; }   // idempotency
    inventory.reserve(rec.value());                  // local tx: business change + processed-event record
    ack.acknowledge();                               // commit after success
}
```

## 5. Delivery Semantics & Exactly-Once
| Semantics | How | Risk |
|---|---|---|
| **At-most-once** | commit offset *before* processing | message loss on crash |
| **At-least-once** (typical) | commit *after* processing; producer `acks=all` + retries | **duplicates** → consumers must be **idempotent** |
| **Exactly-once** | idempotent producer + transactions + `read_committed` (Kafka→Kafka, e.g., Kafka Streams `processing.guarantee=exactly_once_v2`) | external side effects (DB/HTTP) not covered |

- [ ] **Practical "exactly-once effect" outside Kafka:** at-least-once + **idempotent consumer** (dedupe table keyed by event ID with unique constraint, upserts, natural idempotency, conditional updates) — often combined with **Transactional Outbox** on the producer side.
- [ ] **Poison pill handling:** deserialization failure (use `ErrorHandlingDeserializer`), business failure → retry w/ backoff → **Dead Letter Topic (DLT)**.

## 6. Error Handling, Retry & DLT (Spring Kafka)
- [ ] `DefaultErrorHandler` + `BackOff` (fixed/exponential) + `DeadLetterPublishingRecoverer` → `<topic>.DLT`; classify retryable vs non-retryable exceptions
- [ ] **Non-blocking retries** with retry topics: `@RetryableTopic(attempts = "4", backoff = @Backoff(delay = 1000, multiplier = 2))` → `topic-retry-0/1/...` + `-dlt`; avoids blocking the partition (ordering trade-off)
- [ ] **Blocking retries** preserve order but stall the partition
- [ ] DLT operations: alert, inspect, **replay/redrive** tool, fix root cause; never silently drop
- [ ] Idempotent handlers; timeouts to downstream; circuit breaker to pause listener (`KafkaListenerEndpointRegistry` pause/resume)
- [ ] Batch listeners, `RecordInterceptor`, MDC/trace propagation via headers

```java
@RetryableTopic(attempts = "4", backoff = @Backoff(delay = 1000, multiplier = 2.0),
                dltStrategy = DltStrategy.FAIL_ON_ERROR,
                exclude = { ValidationException.class })              // non-retryable
@KafkaListener(topics = "payments.requested")
public void handle(PaymentRequested e) { paymentService.process(e); }

@DltHandler
public void dlt(PaymentRequested e, @Header(KafkaHeaders.EXCEPTION_MESSAGE) String msg) {
    log.error("DLT {} : {}", e, msg);  // alert + store for redrive
}
```

## 7. Operations, Performance & Security
- [ ] **Throughput tuning:** producer batching (`linger.ms`, `batch.size`), compression, more partitions, consumer `fetch.*`, parallelism; **latency tuning:** `linger.ms=0`, `acks=1`.
- [ ] **Capacity:** partitions per broker, disk/retention math, network, page cache; replication overhead.
- [ ] **Monitoring:** under-replicated partitions, ISR shrink, consumer lag, request latency, disk usage, controller count, broker CPU/network.
- [ ] **Security:** TLS, SASL (SCRAM/OAuth), ACLs; encrypt PII; schema governance.
- [ ] **Multi-DC:** MirrorMaker 2 / Cluster Linking; active-passive vs active-active; offset translation.
- [ ] **Kafka Connect** (source/sink connectors, Debezium CDC), **Kafka Streams** (stateful stream processing, KTable, windowing, exactly-once), ksqlDB — overview.
- [ ] **Topic design:** naming conventions, one event type per topic vs multiple, compaction for state, retention, partition count by target throughput ÷ per-partition throughput and consumer parallelism; keys by aggregate ID.
- [ ] **Schema evolution:** additive fields with defaults; Schema Registry compatibility; versioned event types.

## 8. Design Scenarios (Practice Aloud)
1. **Order events:** `OrderCreated` → inventory, payment, notification. Topics/keys, consumer groups, idempotency, DLT, ordering per order.
2. **Outbox → Kafka (Debezium)** with consumer dedupe; how to handle replays.
3. **Exactly-once-ish money movement:** idempotency key, transactions, reconciliation job, DLT for manual handling.
4. **Notification fan-out:** one topic, multiple groups (email/SMS/push), retry topics per channel, rate limiting.
5. **Lag spike in production:** diagnose (slow consumer, GC, downstream slowness, rebalance storm, hot partition, poison message), mitigate (scale consumers ≤ partitions, increase partitions carefully, pause/skip bad messages to DLT, backpressure).
6. **Ordering requirement + parallelism:** key by entity ID; per-key serialization; avoid multi-threading within partition or use key-based worker queues.
7. **Reprocessing/backfill:** new consumer group from `earliest`; seek by timestamp; replay DLT; idempotent handlers.
8. **Schema change** without breaking consumers (Schema Registry, backward compat, dual-publish, deprecation).

## 9. Question Bank
1. 🟢 What is Kafka? Topic, partition, offset, broker, consumer group?
2. 🟢 Kafka vs RabbitMQ/SQS — when each?
3. 🟡 How does Kafka guarantee ordering? What if you need global order?
4. 🟡 How does a producer choose a partition? What if the key is null? Hot partition?
5. 🟡 Explain `acks=0/1/all`, `min.insync.replicas`, replication factor — what combination gives no data loss?
6. 🟡 Consumer groups & partition assignment; what if consumers > partitions?
7. 🟡 Offset commit strategies; consequences of auto-commit; commit before vs after processing?
8. 🟡 What triggers a rebalance; how to minimize impact (cooperative, static membership, tune `max.poll.*`)?
9. 🟡 How do you handle duplicates? How do you make a consumer idempotent?
10. 🟡 Retry & DLT design in Spring Kafka; blocking vs non-blocking retries & ordering trade-off?
11. 🟡 What is consumer lag; how do you monitor and fix it?
12. 🟡 Log compaction — what & use cases?
13. 🔴 Explain idempotent producer & transactions; what does exactly-once cover and *not* cover?
14. 🔴 ISR, high watermark, leader election; what happens when the leader dies / ISR shrinks below `min.insync.replicas`?
15. 🔴 How do you increase partitions safely given key-based ordering?
16. 🔴 Why is Kafka fast? (sequential I/O, page cache, zero-copy, batching, compression)
17. 🔴 How do you avoid the dual-write problem when saving to DB and publishing an event? (Outbox/CDC)
18. 🔴 Design a pipeline processing 1M events/sec — partitions, brokers, consumers, schema, retention, monitoring.
19. 🔴 A consumer is stuck on one bad message and lag is growing — what do you do (immediate + long-term)?
20. 🔴 How do you do zero-downtime schema evolution on events?
21. 🔴 Kafka Streams vs a plain consumer; when to use stateful stream processing?
22. 🔴 How do you test Kafka producers/consumers (`@EmbeddedKafka` or Testcontainers Kafka + Awaitility to wait for the side effect)? (Spring test basics: [05 Part D](05_JUNIT_MOCKITO.md); Kafka specifics are covered here)
23. 🔴 What happens during a broker failure / network partition for producers & consumers?
24. 🔴 What are the trade-offs of Kafka as a source of truth / event store?

## 10. Practice Tasks
- [ ] Run Kafka locally (Docker, KRaft); use `kafka-topics`, `kafka-console-producer/consumer`, `kafka-consumer-groups --describe` to see lag
- [ ] Spring Boot producer + consumer: keyed messages, manual ack, idempotent consumer (dedupe table), DLT
- [ ] Demonstrate rebalance by starting/stopping consumers; demonstrate ordering per key
- [ ] Demonstrate duplicate delivery (crash before commit) and de-duplicate
- [ ] Implement outbox → Kafka (poller first; Debezium optional)
- [ ] Write 2 integration tests with Testcontainers Kafka (happy path + poison message → DLT)

## 11. Self-Check
- [ ] Whiteboard: producer → partitions/replicas → consumer groups, with offsets
- [ ] State a no-data-loss config and explain each setting
- [ ] Explain at-least-once + idempotent consumer vs exactly-once transactions
- [ ] Diagnose lag growth with a concrete checklist
