# Java Senior Engineer Interview Prep — START HERE

**For:** Senior Java Engineer (5–10 yrs) interviews at service-based to mid-product companies (practical coding, strong Java/Spring/DB/Kafka/Microservices fundamentals, LLD + HLD, project discussion, "how do you use AI").

**This folder = 1 main guide (this file) + 12 topic files.** You do **not** read everything. This file tells you *what to read, in what order, for how long*.

---

## 0. The 3 Rules That Keep This Manageable

1. **Fundamentals first, depth later.** Every topic file starts with a **⭐ Fundamentals First** box (the ~20% that covers ~80% of questions). Finish those boxes for *all* topics before going deep on any one.
2. **30–60 min of coding every day.** Coding is a skill, not knowledge — daily small reps beat weekend marathons.
3. **Practice aloud.** Interviews test *explaining*. Answer every question out loud; log misses in `mistakes-log.md` (create it) and re-try after 3 and 10 days.

> Not overwhelmed = small daily blocks, one topic at a time, one rest day per week, one buffer week-end, and permission to **skip the 🔴 items** on your first pass.

---

## 1. Pick Your Plan (Decision Table)

| Time until interview | Use | Weekly effort | Total |
|---|---|---|---|
| **< 2 weeks** | [Triage plan](#triage-plan-under-2-weeks) | 1–2 h/day extra | ~25 h |
| **~1 month** | [1-Month Plan](#6-one-month-plan-4-weeks--84-hours) | ~21 h/wk (3 h weekdays, 6 h Sat, Sun off) | **84 h** |
| **~2 months** | [2-Month Plan](#7-two-month-plan-8-weeks--120-hours) | ~15 h/wk (2 h weekdays, 5 h Sat, Sun off) | **120 h** |
| **Only 1 h/day available** | [Minimum Viable Day](#5-daily-routine) + stretch the 2-month plan to 12 weeks | 7 h/wk | ~85 h |

Not sure? **Default to the 2-month plan** and compress if the interview date moves up.

---

## 2. Document Map

| # | File | What's inside | Priority |
|---|---|---|---|
| 01 | [01_CODING.md](01_CODING.md) | How to improve + interview approach, patterns, **LinkedList / HashMap-HashSet / PriorityQueue** templates & questions, OOP-flavoured, concurrency, machine-coding | P0 |
| 02 | [02_JAVA_CORE.md](02_JAVA_CORE.md) | OOP, collections internals, multithreading, Java 8→21, 40 Q&A | P0 |
| 03 | [03_GC_JVM.md](03_GC_JVM.md) | JVM memory, GC (G1/ZGC…), tuning, OOM playbook | P0/P1 |
| 04 | [04_SPRING_BOOT.md](04_SPRING_BOOT.md) | Core, Boot, REST, JPA, transactions, AOP, Security + questions per topic | P0 |
| 05 | [05_JUNIT_MOCKITO.md](05_JUNIT_MOCKITO.md) | 4 simple parts: **JUnit 5** concepts → **Mockito** concepts → mocking **static & private** → **Spring Boot tests** (application context for controller / service / JPA) | P0 |
| 06 | [06_DATABASE_SQL.md](06_DATABASE_SQL.md) | SQL, indexing, transactions, tuning | P0 |
| 07 | [07_NOSQL_CASSANDRA.md](07_NOSQL_CASSANDRA.md) | NoSQL types, CAP, Redis/Mongo, **Cassandra** | P1 |
| 08 | [08_MICROSERVICES.md](08_MICROSERVICES.md) | Principles, Saga/Outbox/CQRS, resilience, observability | P0/P1 |
| 09 | [09_KAFKA.md](09_KAFKA.md) | Architecture, producer/consumer, semantics, DLT | P0/P1 |
| 10 | [10_LLD_DESIGN_PATTERNS.md](10_LLD_DESIGN_PATTERNS.md) | SOLID, patterns, LLD problems, prepare + approach | P0/P1 |
| 11 | [11_HLD_SYSTEM_DESIGN.md](11_HLD_SYSTEM_DESIGN.md) | System design basics, problems, prepare + approach | P1 |
| 12 | [12_AI_IN_DAILY_WORK.md](12_AI_IN_DAILY_WORK.md) | "How do you use AI?" answer + stories + guardrails | P1 |
| 13 | [13_JAVA8_STREAMS_CODING.md](13_JAVA8_STREAMS_CODING.md) | **Java 8+ Streams coding:** sorting, grouping, **2-level grouping**, `joining`, strings, `flatMap`, maps, **date/time**, pitfalls, Top-10/Top-25 | P0 |

**P0** = almost certainly asked · **P1** = expected at senior level · **P2** = differentiator.
Legend in files: 🟢 easy ≤10 min · 🟡 medium ≤25 min · 🔴 hard ≤40 min (senior differentiator).

---

## 3. Time Budget & How to Cover Each Topic

Hours = *total focused hours for that topic across the plan* (reading + hands-on + questions). Both columns include the topic's share of practice.

| # | Topic | 1-Month | 2-Month | How to cover (in this order) | You're done when… |
|---|---|---|---|---|---|
| 01 | Coding (excl. Streams) | **17 h** | **24 h** | (1) read §1–2 once (approach); (2) per pattern: learn template → 3–5 problems with the 4-pass method; (3) timed mixed sets at the end | You solve 🟢 in ≤10 min & 🟡 in ≤25 min aloud; can write reverse-list, LRU, top-K, Two Sum, sliding window from memory |
| 02 | Java Core | **7 h** | **11 h** | ⭐ box → §3 collections (HashMap!) → §4 concurrency → §5 Java 8+ → question bank | Explain HashMap internals, `ThreadPoolExecutor`, `volatile` vs `synchronized`, streams without notes |
| 03 | GC / JVM | **3 h** | **5 h** | ⭐ box → memory areas → generations/collectors → read one GC log → troubleshooting table | Draw heap, compare Parallel/G1/ZGC, describe an OOM investigation |
| 04 | Spring Boot | **8 h** | **13 h** | ⭐ box → build mini-project REST → JPA (N+1) → transactions → security → question bank | Explain bean lifecycle, auto-config, `@Transactional` pitfalls, N+1 fix |
| 05 | JUnit / Mockito | **3 h** | **5 h** | Parts A→D in order (JUnit → Mockito → static/private → Spring tests), practising each on your mini-project → `@WebMvcTest` + `@DataJpaTest` → *(2-month only)* one Testcontainers test | Write a Mockito service test (success + failure + verify) in <10 min |
| 06 | Database / SQL | **5 h** | **8 h** | ⭐ box → 2 SQL problems/day hands-on → indexing + `EXPLAIN` → isolation levels → questions | All 🟢/🟡 SQL problems solved; explain composite index & isolation levels |
| 07 | NoSQL / Cassandra | **3 h** | **5 h** | ⭐ box → NoSQL types & CAP → Cassandra model from queries → questions | Choose a store for 5 scenarios; model a Cassandra table from a query list |
| 08 | Microservices | **4 h** | **6 h** | ⭐ box → communication → data consistency (Saga/Outbox) → resilience → questions | Explain Saga + Outbox + idempotent consumer end-to-end |
| 09 | Kafka | **4 h** | **6 h** | ⭐ box → partitions/consumer groups → producer acks → delivery semantics → retry/DLT | State a no-data-loss config; explain at-least-once + idempotency |
| 10 | LLD + Patterns | **6 h** | **9 h** | ⭐ box → SOLID → 8 key patterns (code skeletons) → 3–6 LLD problems using the 7-step template | Do a new LLD problem in 45 min with clean code |
| 11 | HLD | **6 h** | **9 h** | ⭐ box → building blocks → estimation → 4–8 designs using the 6-step framework | Present a design in 45 min with estimates, trade-offs, failure modes |
| 12 | AI usage | **1 h** | **2 h** | Read §1 + §3 → write 3 STAR stories with *your* real examples | 90-second answer delivered smoothly with 1 safeguard & 1 metric |
| 13 | Java 8+ Streams coding | **3 h** | **6 h** | ⭐ box → toolbox → grouping → **2-level grouping** → `joining` → sorting → date/time → Top-10 (1-month) / Top-25 (2-month) | You write 2-level `groupingBy`, `mapping`+`joining`, `maxBy` per group, `YearMonth` report in ≤10 min each |
| — | Mocks, revision, project stories | **14 h** | **11 h** | Mistakes-log re-tries, 3 full mock rounds, STAR stories | You can run a full 3-round mock without notes |
| | **TOTAL** | **84 h** | **120 h** | | |

### Suggested "three-pass" reading method (avoids overload)
| Pass | What | When |
|---|---|---|
| **Pass 1 — Fundamentals** | Only the **⭐ Fundamentals First** box in each file + its must-do items | First ~40% of your time (all topics) |
| **Pass 2 — Depth** | Rest of each file's checklist + hands-on tasks | Middle ~40% |
| **Pass 3 — Questions & mocks** | Question banks, 🔴 items, mock rounds | Last ~20% |

(The weekly plans below already follow this: early weeks lean on ⭐ boxes, later weeks on depth/mocks.)

---

## 4. Answer Format (use for every theory question)
**Definition → how it works internally → tiny example → trade-off/pitfall → where I used it.**
Tie answers to your own projects: senior interviews are ~50% "how did *you* do X?".

---

## 5. Daily Routine

| Block | Weekday | Saturday | Sunday |
|---|---|---|---|
| Topic study | per plan (1–2 h) | lab/hands-on or mini-project (2–3 h) | **OFF (rest)** or 30 min light mistakes-log |
| Coding | 45–60 min | 60–120 min | — |
| Recall (mistakes-log, flashcards) | 10–15 min | 30 min weekly self-quiz | — |

### Minimum Viable Day (≈ 60 min — use on busy days; never skip fully)
- 35 min: 1 coding problem (timer on, aloud)
- 20 min: read one ⭐ section or one question-bank block, answer aloud
- 5 min: log mistakes

### Rules for staying on track
- **Missed a day?** Don't double up. Use the weekly buffer (Sat) or shift the plan by a day.
- **Behind schedule?** Drop 🔴 items first, then topic depth (Pass 2); never drop daily coding or mocks.
- **Week-end checkpoint:** if you can't pass the topic's "done when…" check, spend 1 extra hour next Saturday, not a new week.

---

## 6. One-Month Plan (4 Weeks · 84 Hours)

**Schedule:** Mon–Fri 3 h · Sat 6 h · Sun off → 21 h/week.
**Philosophy:** ⭐ fundamentals only for most topics; depth where interviews hit hardest (Java, Spring, coding, design). Skip 🔴 items except in your strongest areas.

### Week 1 — Core Java, GC & SQL (21 h)
| Day | Study | Coding |
|---|---|---|
| Mon | Java: OOP, exceptions, generics (2 h) → 02 §1–2 | 1 h Strings/Arrays |
| Tue | Java: **collections internals / HashMap** (2 h) → 02 §3 | 1 h Arrays + HashMap basics |
| Wed | Java: **concurrency** (2 h) → 02 §4 | 1 h LinkedList |
| Thu | GC: memory areas, generations, collectors (2 h) → 03 §1–3 | 1 h LinkedList/HashSet |
| Fri | GC: troubleshooting (1 h) → 03 §5 · SQL joins/window functions (2 h) → 06 §1–2 | — |
| Sat | Java 8+ & quiz (1 h) → 02 §5, 7 · SQL indexing & isolation (3 h) → 06 §3–4 | 2 h mixed (Strings/LinkedList/HashMap) |
**Hours:** Java 7 · GC 3 · SQL 5 · Coding 6. **Checkpoint:** explain HashMap & ThreadPool; solve 10+ coding problems.

### Week 2 — Spring, Testing, NoSQL, Kafka intro (21 h)
| Day | Study | Coding |
|---|---|---|
| Mon | Spring Core + Boot (2 h) → 04 §1–2 | 1 h HashMap/HashSet |
| Tue | REST, validation, exception handling (2 h) → 04 §3 | 1 h HashMap/HashSet |
| Wed | JPA + N+1 + transactions (2 h) → 04 §4–5 | 1 h PriorityQueue |
| Thu | JUnit + Mockito (2 h) → 05 Parts A–C | 1 h PriorityQueue/Stack |
| Fri | NoSQL types + CAP + Cassandra basics (2 h) → 07 §1–3, 5–6 | 1 h Stack/Queue |
| Sat | Spring Security + AOP quick pass, build mini-project core (2 h) · Spring Boot test slices → 05 Part D (1 h) · NoSQL questions (1 h) · Kafka basics (2 h) → 09 §1–4 | — |
**Hours:** Spring 8 · JUnit 3 · NoSQL 3 · Kafka 2 · Coding 5. **Checkpoint:** REST+JPA+tests running; reproduce & fix an N+1.

### Week 3 — Microservices, Kafka, LLD, HLD start (21 h)
| Day | Study | Coding |
|---|---|---|
| Mon | Microservices: communication + consistency (2 h) → 08 §1–3 | 1 h Streams Top-10 (items 1–4) → [13](13_JAVA8_STREAMS_CODING.md) §15 |
| Tue | Microservices: resilience + observability (2 h) → 08 §4, 6 | 1 h Streams Top-10 (items 5–7, incl. 2-level grouping) |
| Wed | Kafka: semantics, retries, DLT (2 h) → 09 §5–6 | 1 h Streams Top-10 (items 8–10) |
| Thu | LLD: SOLID + patterns (2 h) → 10 §C1–C2 | 1 h Concurrency basics |
| Fri | LLD: practice problem #1–2 with the 7-step template (2 h) | 1 h OOP-flavoured / machine-coding |
| Sat | LLD: problem #3 (2 h) · HLD fundamentals + framework (3 h) → 11 · AI answer + STAR stories (1 h) → 12 | — |
**Hours:** Micro 4 · Kafka 2 · LLD 6 · HLD 3 · AI 1 · Coding 5. **Checkpoint:** 3 LLD problems done; Saga/Outbox explained.

### Week 4 — Design, Revision, Mocks (21 h)
| Day | Activity |
|---|---|
| Mon | HLD: 2 problems (URL shortener, rate limiter) 2 h · Coding 1 h |
| Tue | HLD: 1 more problem 1 h · Revision of mistakes-log 2 h |
| Wed | Coding 1 h · Revise weak topics (⭐ boxes) 2 h |
| Thu | Coding 1 h · Project architecture + STAR stories + intro 2 h |
| Fri | Coding 1 h · **Mock 1:** Java + Spring + DB Q&A (2 h) |
| Sat | **Mock 2 (3 h):** coding + LLD/HLD + behavioral → **review (3 h)** |
**Hours:** HLD 3 · Coding 4 · Mocks/revision 14. **Final:** day before → [Final Checklist](#10-final-checklist).

---

## 7. Two-Month Plan (8 Weeks · 120 Hours)

**Schedule:** Mon–Fri 2 h · Sat 5 h · Sun off → 15 h/week.
**Philosophy:** Pass 1 (⭐ fundamentals) in Weeks 1–4, Pass 2 (depth) in Weeks 5–7, Pass 3 (questions & mocks) in Week 8. Coding every day. Build the mini-project in parallel.

| Wk | Theme | Study hours (topic: h) | Coding (h) · focus | Saturday lab (the 5 h block) | Checkpoint |
|---|---|---|---|---|---|
| **1** | Core Java + SQL basics | Java 6 · SQL 5 | 4 · Strings/Arrays | Java collections & concurrency demos; SQL on a seeded DB | Explain HashMap internals; 8 coding problems |
| **2** | Java depth + GC | Java 5 · GC 5 · SQL 1 | 4 · LinkedList | Producer–consumer + deadlock demo; run a GC log & heap dump lab | Read a GC log; LRU cache (both ways) |
| **3** | Spring fundamentals + testing start | Spring 7 · SQL 2 · JUnit 2 | 4 · HashMap/HashSet | Mini-project: REST + validation + exception handler; unit tests | REST app + tests green |
| **4** | JPA, transactions, security, testing | Spring 6 · JUnit 3 · NoSQL 2 | 4 · PriorityQueue + Stack/Queue | JPA entities, reproduce/fix N+1, JWT security, `@WebMvcTest` | N+1 fixed; `@Transactional` pitfalls demoed |
| **5** | Microservices + NoSQL + Kafka intro | Micro 6 · NoSQL 3 · Kafka 2 | 4 · Streams + OOP | Resilience4j circuit breaker + WireMock; Cassandra table modeling | Draw Saga + Outbox; model a Cassandra table |
| **6** | Kafka + LLD start + AI | Kafka 4 · LLD 5 · AI 2 | 4 · Concurrency | Spring Kafka producer/consumer + DLT + idempotency; first 2 LLD problems | Kafka no-loss config; 2 LLD problems |
| **7** | LLD + HLD + first mocks | LLD 4 · HLD 6 · Mock 2 | 3 · machine-coding | 2 HLD designs (aloud, diagram) + 1 LLD build + 1 mock | 4 LLD + 4 HLD done; first mock |
| **8** | Questions, revision, mocks | HLD 3 · Mocks/revision 9 | 3 · timed mixed | **3 full mock rounds** (coding + Java/Spring + design) | Run full mock without notes; final checklist |

**Hours per week:** 15 each → 8 × 15 = **120**.
**Weekday template (2 h):** 1 h topic study + 45 min coding + 15 min recall. **Saturday (5 h):** 3 h lab/hands-on + 1 h coding timed set + 1 h review/mistakes-log.

---

## Triage Plan (Under 2 Weeks)

Only ~25 h. Priority order, ⭐ boxes only:
1. **Days 1–3 (8 h):** 02 ⭐ (collections, concurrency, streams) + 01 templates (LinkedList, HashMap, PQ, sliding window) + 13 ⭐ box & Top-10 — 1 coding problem/day
2. **Days 4–6 (7 h):** 04 ⭐ (Spring DI/Boot/REST/JPA/`@Transactional`) + 06 ⭐ (SQL + indexing)
3. **Days 7–9 (5 h):** 08 ⭐ + 09 ⭐ (Saga/Outbox, Kafka semantics) + 05 ⭐ (Mockito)
4. **Days 10–12 (4 h):** 10 ⭐ (SOLID + 5 patterns) + 11 ⭐ (framework + URL shortener)
5. **Days 13–14 (1 h + mock):** 12 (AI answer) + project stories + one mock

Skip: 03, 07 depth (read the ⭐ only), all 🔴 items.

---

## 8. Mini-Project (Hands-On Glue for Topics 04–09)

**"Order Management Service"** — Spring Boot 3, Java 17/21. Build it incrementally; each step gives interview stories.

| Step | Add | Topic file |
|---|---|---|
| 1 | REST CRUD, validation, global exception handler, profiles | 04 |
| 2 | JPA (Order/Item/Customer), Flyway, reproduce + fix an N+1, pagination | 04, 06 |
| 3 | JUnit/Mockito + `@WebMvcTest` + `@DataJpaTest` (Testcontainers) | 05 |
| 4 | JWT auth + roles | 04 |
| 5 | Publish `OrderCreated` to Kafka; second service consumes idempotently + DLT | 09 |
| 6 | Resilience4j circuit breaker/retry to a stub payment; Redis cache; outbox | 08 |
| 7 | Actuator, Micrometer metrics, correlation-ID logging, Dockerfile | 04 |
| 8 *(optional)* | Audit/event history in Cassandra or Mongo | 07 |

---

## 9. Project Discussion & Behavioral (Prepare Alongside)

Prepare **5–6 STAR stories** (Situation, Task, Action, Result — with numbers):
- [ ] Architecture of your current project (draw it; data flow; *why* these choices)
- [ ] A hard production issue (symptom → investigation → root cause → fix → prevention)
- [ ] A performance optimization (before/after metrics)
- [ ] A design decision with trade-offs you owned (and one you'd change)
- [ ] Conflict / disagreement / mentoring / code-review culture
- [ ] Delivering under ambiguity or deadline; handling tech debt
- [ ] AI usage story ([12_AI_IN_DAILY_WORK.md](12_AI_IN_DAILY_WORK.md))
- [ ] 3–4 questions to ask the interviewer

**Self-intro (60–90 s):** current role → key stack → 2 achievements with numbers → why change.

---

## 10. Final Checklist

**Progress tracker (tick when the "done when…" in Section 3 passes):**
- [ ] 01 Coding · [ ] 13 Streams coding · [ ] 02 Java · [ ] 03 GC · [ ] 04 Spring · [ ] 05 Testing · [ ] 06 SQL
- [ ] 07 NoSQL/Cassandra · [ ] 08 Microservices · [ ] 09 Kafka · [ ] 10 LLD · [ ] 11 HLD · [ ] 12 AI

**Day before:**
- [ ] Review mistakes-log + cheat-sheets (HashMap, concurrency, `@Transactional`, N+1, indexing, SOLID, Kafka semantics, GC)
- [ ] Re-code from memory: LRU, producer–consumer, thread-safe singleton, reverse list, top-K heap
- [ ] Project diagram + STAR stories reviewed
- [ ] IDE/environment ready (Java version, Maven, Spring Boot skeleton) for live coding
- [ ] Sleep; join 10 min early

> **Golden rules:** (1) clarify before coding, (2) think aloud, (3) simple solution first then optimize, (4) state trade-offs, (5) test with examples + edge cases, (6) "I don't know, here's my reasoning / how I'd find out" beats bluffing.

## 11. Resources
| Area | Resource |
|---|---|
| Java | *Effective Java*, *Java Concurrency in Practice*, Baeldung, Jenkov; *Modern Java in Action* |
| Spring | Official Spring docs/guides, Baeldung |
| Kafka | Confluent developer docs, *Kafka: The Definitive Guide* |
| DB / Cassandra | *Use The Index, Luke*, *DDIA*, DataStax docs |
| LLD | Refactoring.Guru, *Head First Design Patterns* |
| HLD | Alex Xu *System Design Interview* vol 1 & 2, *DDIA*, ByteByteGo |
| Practice | LeetCode (Top Interview 150 — Easy/Medium), HackerRank Java, peer mocks |
