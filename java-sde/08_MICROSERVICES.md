# 08 — Microservices

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **4 h** · 2-month plan **6 h** (see [README](README.md)).

**Learn in this order**
1. **Monolith vs microservices trade-offs; when *not* to use; service boundaries (DDD bounded context)** (§1) — 45 min
2. **Sync vs async communication; API gateway; idempotency & versioning** (§2) — 45 min
3. **Data consistency:** database-per-service, **Saga** (choreography vs orchestration), **Outbox**, idempotent consumer (§3) — 1.5 h
4. **Resilience:** timeout, retry + jitter, circuit breaker, bulkhead; cascading-failure story (§4) — 1 h
5. **Observability:** logs/metrics/traces, correlation IDs (§6) — 30 min

**Done when:** you explain Saga + Outbox + idempotent consumer as one end-to-end flow and draw a resilient call chain.
**Skip if short on time:** service mesh, event sourcing depth, multi-region DR, Spring Cloud Config/Eureka specifics.

> Related: [09_KAFKA.md](09_KAFKA.md) · [04_SPRING_BOOT.md](04_SPRING_BOOT.md) · [11_HLD_SYSTEM_DESIGN.md](11_HLD_SYSTEM_DESIGN.md)
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Understand the *why* before patterns:** be able to argue *when microservices are the wrong answer* (small team, unclear domain, no DevOps maturity).
2. **Learn patterns as problem → solution → trade-off triplets** (§3–4). For each, draw a sequence diagram.
3. **Build it:** mini-project — two services (Order, Payment) + Kafka + circuit breaker + outbox. Break things deliberately (kill payment, delay responses) and observe.
4. **Prepare a real story:** one microservices problem you faced (latency cascade, data consistency, versioning) and how you fixed it.
5. Rehearse the question bank (§8). Time: 4 h (1-month: ⭐ box + §8 questions) / 6 h (2-month: add the §9 practice tasks).

---

## 1. Fundamentals
- [ ] **Definition:** small, independently deployable services, organized around **business capabilities** (bounded contexts), each owning its data.
- [ ] **Monolith vs modular monolith vs microservices** — trade-offs

| Aspect | Monolith | Microservices |
|---|---|---|
| Deploy | single unit | independent per service |
| Scale | whole app | per service |
| Tech | uniform | polyglot (with cost) |
| Data | one DB, ACID | DB per service, eventual consistency |
| Complexity | in code | in operations/network |
| Team | one big | autonomous teams (Conway's law) |
| Failure | all-or-nothing | partial failures, cascading risks |

- [ ] **When NOT to:** small team/product, unclear boundaries, low ops maturity. Start with a **modular monolith**; extract services when pain justifies.
- [ ] **12-factor app** principles (config in env, stateless processes, logs as streams, disposability, dev/prod parity).
- [ ] **Domain-Driven Design (DDD) basics:** bounded context, aggregate, entity/value object, ubiquitous language, context map; how DDD drives service boundaries.
- [ ] **Service sizing:** by business capability/aggregate, not by tables; avoid "nano-services" and "distributed monolith" (services that must deploy together).
- [ ] **Fallacies of distributed computing:** network is reliable, latency is zero, etc.

## 2. Communication
- [ ] **Synchronous:** REST/HTTP+JSON, gRPC (HTTP/2, protobuf, streaming, contracts), GraphQL (BFF/aggregation)
- [ ] **Asynchronous:** messaging/events (Kafka, RabbitMQ, SQS/SNS) — decoupling, buffering, resilience; **commands vs events**
- [ ] **Choose:** sync for queries needing immediate answer; async for workflows/notifications/integration; avoid long sync chains (latency multiplies, availability multiplies: 99.9%⁵ ≈ 99.5%)
- [ ] **Idempotency** (idempotency keys), retries, timeouts, deadlines propagation
- [ ] **API design & versioning:** backward compatible changes (additive), URL/header versioning, consumer-driven contracts (Pact), deprecation policy; **Tolerant Reader**
- [ ] **Service discovery:** client-side (Eureka, Consul) vs server-side (K8s Services/DNS, load balancers)
- [ ] **API Gateway:** routing, auth, rate limiting, aggregation, TLS termination (Spring Cloud Gateway, Kong, Apigee); **BFF** pattern; avoid putting business logic in gateway
- [ ] **Service mesh** (Istio/Linkerd): mTLS, retries, traffic shifting, observability sidecars (overview)

## 3. Data Management & Consistency (Most Probed)
- [ ] **Database per service**; no shared DB; data ownership; sharing via APIs/events
- [ ] **No distributed ACID** → eventual consistency; why 2PC/XA is avoided (blocking, availability, coupling)
- [ ] **Saga pattern** — sequence of local transactions + **compensating actions**
  - **Choreography:** services react to events (simple, decoupled; harder to follow/test; risk of cyclic deps)
  - **Orchestration:** central orchestrator drives steps (clear flow, easier monitoring; orchestrator is a coupling point) — Temporal/Camunda/Axon/custom state machine
  - Design: compensation must be **idempotent**, handle partial failure, timeouts, pivot transactions
  - Example: Order → Reserve Inventory → Charge Payment → Confirm; failure at payment ⇒ release inventory, cancel order
- [ ] **Transactional Outbox:** write business data + outbox event in **same local DB transaction**; a relay (poller or **CDC/Debezium**) publishes to Kafka → solves **dual-write problem**; consumers must be idempotent (at-least-once)
- [ ] **Inbox/idempotent consumer:** store processed message IDs (unique constraint) to dedupe
- [ ] **CQRS:** separate write model and read model(s) (denormalized projections); eventual consistency between them; use when read/write shapes or loads differ — not by default
- [ ] **Event Sourcing:** store state changes as events; rebuild state; audit/time-travel; complexity (versioning, snapshots) — use selectively
- [ ] **Data duplication & caching:** local read replicas via events; stale data tolerance; ownership
- [ ] **Distributed IDs & ordering:** UUID/ULID/Snowflake; per-aggregate ordering via Kafka key
- [ ] **Reporting across services:** data lake/warehouse via CDC or event streams; API composition; CQRS views
- [ ] **Consistency spectrum:** read-your-writes, session consistency, UI strategies (optimistic UI, polling, status endpoints)

## 4. Resilience Patterns
| Pattern | Problem | Notes (Resilience4j) |
|---|---|---|
| **Timeout** | hanging calls consume threads | always set connect + read timeouts; budget end-to-end deadline |
| **Retry** | transient faults | exponential backoff + **jitter**; only idempotent ops; cap attempts; avoid **retry storms** (retry at one layer only) |
| **Circuit Breaker** | stop hammering a failing dependency | states CLOSED → OPEN → HALF_OPEN; failure-rate threshold, sliding window, wait duration; fallback response |
| **Bulkhead** | one slow dependency exhausts all threads | separate pools/semaphores per dependency |
| **Rate limiter / throttling** | protect self/downstream | token bucket; 429 + `Retry-After` |
| **Fallback / graceful degradation** | partial functionality | cached/default response |
| **Load shedding / backpressure** | overload | reject early; bounded queues |
| **Idempotency** | duplicate requests/retries | idempotency key + stored result |
| **Health checks** | unhealthy instances | liveness vs readiness probes; remove from LB |
| **Chaos testing** | verify resilience | inject latency/failures |

- [ ] **Cascading failure** anatomy: slow dependency → threads blocked → pool exhausted → upstream times out → retries amplify load → system-wide outage. Mitigation = timeouts + bulkheads + circuit breakers + load shedding + capacity limits.
- [ ] **Order of decorators:** `Retry( CircuitBreaker( RateLimiter( TimeLimiter( Bulkhead( call )))))` — know why the timeout sits inside the retry (each attempt gets its own time budget).

## 5. Security
- [ ] Edge authN at gateway (OIDC/JWT), **authZ in each service**; propagate identity (token relay)
- [ ] Service-to-service: mTLS (mesh), OAuth2 client credentials
- [ ] Secrets management (Vault/K8s Secrets/cloud KMS), least privilege, network policies
- [ ] Input validation; rate limiting; audit logging; OWASP API Top 10 basics

## 6. Observability & Operations
- [ ] **Three pillars:** logs (structured JSON, correlation/trace IDs), metrics (RED: Rate, Errors, Duration; USE for resources; golden signals), traces (OpenTelemetry, Zipkin/Jaeger/Tempo)
- [ ] **Centralized logging** (ELK/OpenSearch, Splunk), dashboards (Grafana), alerting on SLO burn rate, on-call/runbooks
- [ ] **SLI/SLO/SLA**, error budgets
- [ ] **Deployment:** containers, Kubernetes (Deployment, Service, Ingress, HPA, ConfigMap/Secret, probes, resource requests/limits), **CI/CD**, blue-green, **canary**, rolling, feature flags, backward-compatible DB migrations
- [ ] **Config management:** externalized config, Spring Cloud Config/K8s ConfigMaps, dynamic refresh with care
- [ ] **Testing strategy:** unit → component → **contract tests** → few E2E; testing in production (canaries, shadow traffic)
- [ ] **Monolith → microservices migration:** **Strangler Fig**, anti-corruption layer, branch by abstraction, extract by seams, data decoupling first

## 7. Spring Cloud / Java Ecosystem Mapping
| Need | Tooling |
|---|---|
| Gateway | Spring Cloud Gateway |
| Discovery | Eureka / Consul / K8s DNS |
| Config | Spring Cloud Config / K8s ConfigMap |
| Resilience | Resilience4j (`@CircuitBreaker`, `@Retry`, `@Bulkhead`, `@RateLimiter`, `@TimeLimiter`) |
| HTTP clients | OpenFeign / `RestClient` / `WebClient` |
| Messaging | Spring Kafka / Spring Cloud Stream |
| Tracing | Micrometer Tracing + OpenTelemetry |
| Saga/workflow | Temporal, Camunda, Axon, custom state machine |
| CDC / Outbox | Debezium + Kafka Connect |

*(Many cloud-native shops use K8s + Istio and skip Eureka/Config Server — know both.)*

## 8. Question Bank
1. 🟢 Monolith vs microservices — pros/cons; when not to use microservices?
2. 🟢 How do services communicate? Sync vs async — when each?
3. 🟡 How do you define service boundaries? (DDD bounded contexts; business capability)
4. 🟡 What is an API Gateway; what shouldn't go there? BFF?
5. 🟡 Circuit breaker states and config; how does it differ from retry? what fallback?
6. 🟡 Retry pitfalls: retry storms, non-idempotent calls, multiplied retries across layers.
7. 🟡 How do you version APIs and avoid breaking consumers?
8. 🟡 How do you trace one request across 6 services? (trace/correlation IDs, OpenTelemetry)
9. 🟡 Database-per-service: how do you do reporting/joins across services?
10. 🟡 How do you handle config & secrets across environments?
11. 🔴 **How do you maintain data consistency across services?** Saga (choreography vs orchestration), compensation, idempotency.
12. 🔴 **Dual-write problem** (DB + Kafka) and the Outbox solution; how do you guarantee at-least-once and dedupe?
13. 🔴 CQRS & Event Sourcing — benefits, costs, when to use?
14. 🔴 Design the order → payment → inventory flow with failures at each step (timeouts, partial success, duplicate messages).
15. 🔴 Cascading failure incident — what happened and how to prevent (timeouts, bulkheads, load shedding)?
16. 🔴 How do you deploy with zero downtime when DB schema changes? (expand/contract, backward compat)
17. 🔴 How would you migrate a monolith to microservices? (Strangler Fig, ACL, data split, risks)
18. 🔴 Distributed monolith — symptoms & fixes?
19. 🔴 How do you ensure ordering and exactly-once-*effect* in event-driven flows?
20. 🔴 Contract testing vs E2E for service integration?
21. 🔴 Multi-region / DR for microservices — active-active vs active-passive; RPO/RTO; data replication conflicts.
22. 🔴 Service latency doubled after adding a new dependency — investigate and fix.

## 9. Practice Tasks
- [ ] Draw sequence diagrams for: Saga (orchestrated) with compensation; Outbox + Debezium; Circuit breaker transitions
- [ ] Implement Order/Payment services with Resilience4j (timeout + retry w/ jitter + circuit breaker + fallback); use WireMock to inject failures
- [ ] Implement outbox table + poller publisher; make the consumer idempotent
- [ ] Add trace/correlation IDs through HTTP + Kafka; view in Zipkin/Jaeger
- [ ] Write a one-page "service boundary decision" for an e-commerce domain (Catalog, Cart, Order, Payment, Inventory, Notification, User)

## 10. Self-Check
- [ ] Explain Saga + Outbox + idempotent consumer as one end-to-end flow without notes
- [ ] Draw a resilient call chain with timeouts/retries/circuit-breaker and explain the failure modes it handles
- [ ] State at least 3 reasons to *not* choose microservices
- [ ] Tell one real production story (incident or design) using these concepts
