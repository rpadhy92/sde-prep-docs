# 04 — Spring Boot & Spring Ecosystem

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **8 h** · 2-month plan **13 h** (see [README](README.md)).

**Learn in this order (build the mini-project as you go)**
1. **IoC/DI, bean lifecycle & scopes** (§1) — 1.5 h
2. **Boot: auto-configuration, property precedence, profiles** (§2) — 1 h
3. **REST layer: controllers, validation, `@RestControllerAdvice`** (§3) — 1.5 h
4. **JPA: relationships, lazy vs eager, N+1 fix** (§4) — 2 h
5. **`@Transactional`: proxy mechanics, propagation, rollback rules, pitfalls** (§5) — 1 h
6. **Security: JWT flow + filter chain; AOP idea** (§6–7) — 1 h

**Done when:** you can explain auto-config and the bean lifecycle, reproduce & fix an N+1, and name 3 `@Transactional` pitfalls (self-invocation, private methods, rollback on checked exceptions).
**Skip if short on time:** custom starters, `BeanFactoryPostProcessor`, OAuth2 details beyond Auth Code + client credentials, native images, second-level cache.

> Testing → [05_JUNIT_MOCKITO.md](05_JUNIT_MOCKITO.md) · Microservices → [08_MICROSERVICES.md](08_MICROSERVICES.md) · Kafka → [09_KAFKA.md](09_KAFKA.md) · DB → [06_DATABASE_SQL.md](06_DATABASE_SQL.md)
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Learn by building:** extend the mini-project from the README (REST → JPA → Security → Kafka). Every concept below should have a line of code you wrote.
2. **For each topic:** (a) know *what problem it solves*, (b) *how Spring implements it* (proxy, post-processor, filter), (c) *common pitfalls*.
3. **Debug-first mindset:** run with `--debug` (conditions report), enable `logging.level.org.springframework=DEBUG` on one startup, and read what happens.
4. **Time:** 8 h (1-month) / 13 h (2-month). First pass = the ⭐ box while building REST + JPA; second pass = §4–8 depth; third pass = §9 questions.

---

## 1. Spring Core
- [ ] IoC & DI; **constructor** vs setter vs field injection (why constructor: immutability, testability, fails fast)
- [ ] Bean lifecycle: instantiate → populate → `Aware` callbacks → `BeanPostProcessor.before` → `@PostConstruct`/`InitializingBean` → `postProcessAfter` (**proxies are created here**) → ready → `@PreDestroy`
- [ ] Scopes: singleton, prototype, request, session; singleton-injecting-prototype problem (`ObjectProvider`, `@Lookup`, scoped proxy)
- [ ] `ApplicationContext` vs `BeanFactory`; `refresh()` phases; component scanning
- [ ] Stereotypes: `@Component/@Service/@Repository/@Controller`, `@Configuration` (CGLIB full vs lite), `@Bean`
- [ ] Resolving ambiguity: `@Primary`, `@Qualifier`, by-name, `List<Interface>` injection
- [ ] Circular dependencies: why they occur, why constructor injection fails, fixes (redesign, `@Lazy`, setter)
- [ ] `@Value`, `@ConfigurationProperties` (type-safe, validated, relaxed binding), Profiles, `Environment`
- [ ] `BeanPostProcessor` vs `BeanFactoryPostProcessor`
- [ ] Events: `ApplicationEventPublisher`, `@EventListener`, `@TransactionalEventListener`, `@Async`

## 2. Spring Boot Fundamentals
- [ ] What Boot adds: starters, auto-configuration, embedded server, actuator, opinionated defaults
- [ ] `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
- [ ] **Auto-config mechanics:** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (older: `spring.factories`), `@Conditional*` (`OnClass`, `OnMissingBean`, `OnProperty`), ordering; exclude/debug with `--debug`
- [ ] Custom starter: `autoconfigure` module + `starter` module; `@ConfigurationProperties`; `@ConditionalOnMissingBean` lets users override
- [ ] Config: `application.yml`, **property precedence** (cmd-line > env vars > profile files > default files), profile-specific files, externalised config, secrets (Vault/K8s secrets; never in git)
- [ ] Embedded servers (Tomcat default; Jetty/Undertow); thread/connection tuning (`server.tomcat.threads.max`)
- [ ] `SpringApplication.run()` flow: env prep → context create → refresh → runners (`CommandLineRunner`) → ready; startup optimization (lazy init, AOT)
- [ ] Packaging: fat JAR, layered JAR, Docker (buildpacks/Jib), native image (GraalVM)
- [ ] **Boot 3 specifics:** Java 17 baseline, `jakarta.*`, Micrometer observability, `ProblemDetail`, virtual threads (`spring.threads.virtual.enabled=true`), `RestClient`

## 3. Spring MVC / REST
- [ ] Request flow: `DispatcherServlet` → `HandlerMapping` → `HandlerAdapter` → controller → `HttpMessageConverter` (Jackson) → response
- [ ] Annotations: `@RestController`, `@RequestMapping` family, `@PathVariable`, `@RequestParam`, `@RequestBody`, `@RequestHeader`, `@ResponseStatus`, `ResponseEntity`
- [ ] **Validation:** `@Valid`/`@Validated`, Bean Validation constraints, custom constraint validator, `MethodArgumentNotValidException`
- [ ] **Global error handling:** `@RestControllerAdvice` + `@ExceptionHandler`, `ProblemDetail` (RFC 7807), consistent error schema, no stack traces leaked
- [ ] **Filter vs Interceptor vs AOP:** Filter = servlet-level (before DispatcherServlet), Interceptor = MVC-level (pre/post handler), AOP = method-level on beans
- [ ] REST design: resource naming, HTTP verbs & idempotency (PUT/DELETE idempotent, POST not), status codes (201/204/400/401/403/404/409/422/429/5xx), versioning, pagination, filtering, DTO vs entity, HATEOAS (overview)
- [ ] Jackson: `@JsonIgnore`, `@JsonProperty`, `@JsonInclude`, custom (de)serializers, date handling
- [ ] File upload/download, streaming (`StreamingResponseBody`), CORS, async controllers (`Callable`, `DeferredResult`, `CompletableFuture`)
- [ ] HTTP clients: `RestTemplate` (maintenance mode) vs `WebClient` (reactive) vs `RestClient` (3.2+) vs OpenFeign — **timeouts, retries, connection pools**
- [ ] OpenAPI/Swagger (springdoc)

## 4. Spring Data JPA / Hibernate
- [ ] JPA (spec) vs Hibernate (impl) vs Spring Data JPA (repository abstraction)
- [ ] Mapping: `@Entity`, `@Id`, `@GeneratedValue` (IDENTITY vs SEQUENCE — batching implications), `@Column`, `@Embedded`, `@Enumerated(STRING)`, `@Version`
- [ ] Relationships: `@OneToMany/@ManyToOne/@ManyToMany/@OneToOne`, `mappedBy` (inverse side), owning side, cascade types, `orphanRemoval`, helper methods to sync both sides, avoid `@ManyToMany` for non-trivial links
- [ ] **Lazy vs Eager**; `LazyInitializationException` (fixes: fetch join, `@EntityGraph`, DTO projection; **not** open-in-view)
- [ ] **N+1 problem:** detect (SQL logging / Hibernate statistics) & fix (`JOIN FETCH`, `@EntityGraph`, `@BatchSize`, DTO projections)
- [ ] Persistence context: entity states (transient/managed/detached/removed), dirty checking, flush modes, first-level cache; second-level cache & query cache (when stale)
- [ ] Repositories: `CrudRepository`/`JpaRepository`/`PagingAndSortingRepository`; derived queries; `@Query` JPQL/native; interface/class **projections**; `Specification`/Criteria; `@Modifying`
- [ ] Pagination/sorting (`Pageable`, `Slice` vs `Page` — count query cost), keyset pagination
- [ ] Locking: optimistic (`@Version`, `OptimisticLockException`) vs pessimistic (`@Lock(PESSIMISTIC_WRITE)`)
- [ ] Batching: `hibernate.jdbc.batch_size`, `saveAll` pitfalls with IDENTITY, bulk update JPQL bypasses persistence context
- [ ] Migrations: Flyway/Liquibase; `ddl-auto` (`validate` in prod, never `update`)
- [ ] Connection pool (HikariCP): `maximumPoolSize` sizing, `connectionTimeout`, leak detection; open-in-view (`spring.jpa.open-in-view=false`)
- [ ] `equals/hashCode` on entities (business key / id handling), `toString` loops with bidirectional associations

## 5. Transactions
- [ ] `@Transactional` = AOP proxy → begin/commit/rollback around the method
- [ ] **Propagation:** REQUIRED (default), REQUIRES_NEW, NESTED, SUPPORTS, MANDATORY, NOT_SUPPORTED, NEVER
- [ ] **Isolation:** READ_UNCOMMITTED → SERIALIZABLE; anomalies: dirty read, non-repeatable read, phantom, lost update; DB defaults (PostgreSQL READ COMMITTED, MySQL InnoDB REPEATABLE READ)
- [ ] Rollback rules: runtime exceptions & errors by default; checked need `rollbackFor`
- [ ] **Pitfalls:** self-invocation (bypasses proxy), `private`/`final` methods, wrong layer, long transactions holding connections, swallowing exceptions, `readOnly=true` semantics, transaction + async thread boundaries, calling remote services inside a tx
- [ ] `@TransactionalEventListener(AFTER_COMMIT)` for side-effects (publish Kafka after commit → better: **Outbox**, see [08](08_MICROSERVICES.md))
- [ ] Distributed transactions: avoid 2PC → Saga/Outbox

## 6. Spring AOP
- [ ] Concepts: aspect, advice, pointcut, join point, weaving; **JDK proxy (interface) vs CGLIB (subclass)**; Boot defaults to CGLIB
- [ ] Advice: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around`
- [ ] Use cases: logging, timing, auditing, caching, retry, security
- [ ] Limitations: only Spring-managed beans, only public method calls through proxy (self-invocation issue)

## 7. Spring Security
- [ ] AuthN vs AuthZ; `SecurityFilterChain` bean (no more `WebSecurityConfigurerAdapter`); filter order
- [ ] `UserDetailsService`, `PasswordEncoder` (BCrypt/Argon2), `AuthenticationManager`/`Provider`, `SecurityContextHolder`
- [ ] Session vs token; **JWT**: header.payload.signature, validation (signature, `exp`, `iss`, `aud`), refresh tokens, revocation strategies, storage (HttpOnly cookie vs header)
- [ ] OAuth2/OIDC: Authorization Code (+PKCE), Client Credentials (service-to-service), Resource Server, scopes/roles
- [ ] Method security: `@PreAuthorize`, `@Secured`, SpEL; roles vs authorities
- [ ] CSRF (needed for cookie sessions, disabled for stateless tokens), CORS, security headers, common attacks (SQLi, XSS, IDOR), input validation, secrets, rate limiting

## 8. Production Concerns (Boot)
- [ ] Actuator endpoints (`health`, `metrics`, `prometheus`, `info`, `env` — secure them), health groups for liveness/readiness
- [ ] Micrometer metrics + custom metrics; logging (SLF4J/Logback JSON, MDC correlation IDs); tracing (Micrometer Tracing/OpenTelemetry)
- [ ] Caching: `@Cacheable/@CacheEvict/@CachePut`, Caffeine (local) vs Redis (distributed), TTL, stampede
- [ ] Scheduling: `@Scheduled` (+ ShedLock for multi-instance), `@Async` with custom `TaskExecutor`
- [ ] Graceful shutdown (`server.shutdown=graceful`), K8s probes, resource limits, JVM container flags
- [ ] Messaging → [09_KAFKA.md](09_KAFKA.md); resilience → [08_MICROSERVICES.md](08_MICROSERVICES.md)

---

## 9. Questions by Topic (3a)

### Core / IoC
1. 🟢 What is IoC/DI? Why constructor injection over field injection?
2. 🟡 Full bean lifecycle. Where do `@PostConstruct` and `BeanPostProcessor` fit; where are AOP proxies created?
3. 🟡 Is a singleton bean thread-safe? How to inject a prototype into a singleton?
4. 🟡 `@Component` vs `@Bean`; `@Configuration` proxying (`proxyBeanMethods`)?
5. 🟡 Two beans of same type — how does Spring choose? (`@Primary`, `@Qualifier`, parameter name)
6. 🔴 How are circular dependencies resolved (3-level cache); why constructor injection can't?
7. 🔴 `BeanFactory` vs `ApplicationContext`; what happens in `refresh()`?

### Boot / Auto-config
1. 🟢 What does `@SpringBootApplication` do?
2. 🟡 How does auto-configuration work? How to exclude/debug it?
3. 🟡 Property precedence; per-environment config & secrets?
4. 🟡 `@ConfigurationProperties` vs `@Value`?
5. 🔴 Build a custom starter; how does `@ConditionalOnMissingBean` allow override?
6. 🔴 Walk through startup from `SpringApplication.run()` to "ready".
7. 🔴 Startup is 40 s — how do you speed it up?

### MVC / REST
1. 🟢 `@Controller` vs `@RestController`; `@PathVariable` vs `@RequestParam`?
2. 🟡 `DispatcherServlet` request flow?
3. 🟡 Global exception handling with consistent error response?
4. 🟡 Filter vs Interceptor vs AOP — when each?
5. 🟡 Design pagination, versioning, idempotency (idempotency keys for POST)? PUT vs PATCH vs POST?
6. 🟡 Request validation & custom validators?
7. 🔴 Stream a 1 GB file download without OOM; handle large uploads?
8. 🔴 `RestTemplate` vs `WebClient` — timeouts/retries/pooling; avoid cascading failures?
9. 🔴 Tomcat thread pool exhausted — symptoms, causes (blocking downstream), fixes?

### JPA / Hibernate
1. 🟢 `JpaRepository` vs `CrudRepository`; derived queries?
2. 🟡 Lazy vs Eager; `LazyInitializationException`?
3. 🟡 **N+1 problem — detect & fix?**
4. 🟡 `mappedBy`, cascade types, `orphanRemoval`?
5. 🟡 Entity vs DTO — why not return entities from APIs?
6. 🔴 Persistence context, dirty checking, flush modes?
7. 🔴 First- vs second-level cache; stale data scenarios?
8. 🔴 Optimistic vs pessimistic locking; prevent lost updates?
9. 🔴 Efficient bulk inserts (batching, IDENTITY problem, `JdbcTemplate`/COPY alternatives)?
10. 🔴 Why `open-in-view` is considered an anti-pattern?

### Transactions
1. 🟢 What does `@Transactional` do; which layer?
2. 🟡 Why doesn't it work on self-invocation/private methods?
3. 🟡 Default rollback rules; change them?
4. 🟡 `REQUIRED` vs `REQUIRES_NEW` example (audit log survives rollback)?
5. 🔴 Isolation levels & anomalies; DB default?
6. 🔴 Consistency across two microservices? (Saga/Outbox vs 2PC)
7. 🔴 Tx + Kafka publish — how to avoid dual-write inconsistency?

### AOP
1. 🟢 What is AOP; real uses?
2. 🟡 JDK proxy vs CGLIB; why `final` class/method issues?
3. 🟡 Write `@Around` timing aspect.

### Security
1. 🟢 AuthN vs AuthZ?
2. 🟡 Explain the Spring Security filter chain.
3. 🟡 JWT: structure, validation, revocation, storage?
4. 🟡 Stateless vs session auth; when CSRF is required?
5. 🔴 OAuth2 Auth Code + PKCE flow; Resource Server config?
6. 🔴 Securing service-to-service calls (client credentials, mTLS)?

### Ops
1. 🟡 What do you expose via Actuator in prod & how secure it?
2. 🔴 App is slow in prod — step-by-step investigation (metrics → threads → GC → DB → downstream → pool).
3. 🔴 Pod OOMKilled / restarts — what do you check?

---

## 10. Hands-On Checklist
- [ ] REST CRUD with DTOs, validation, `@RestControllerAdvice`, Swagger
- [ ] Profiles (`dev`/`prod`) + `@ConfigurationProperties`
- [ ] JPA entities with 1-N & N-N; reproduce & fix an N+1; Flyway migration
- [ ] `@Transactional` demos: rollback on checked vs unchecked, self-invocation bug, `REQUIRES_NEW`
- [ ] JWT security with roles; `@PreAuthorize`
- [ ] Custom `@Around` aspect; custom starter (optional)
- [ ] Actuator + Micrometer + correlation ID filter
- [ ] Caching with Caffeine/Redis; `@Scheduled` + ShedLock

## 11. Self-Check
- [ ] Explain auto-config, bean lifecycle, proxy/transaction behavior without notes
- [ ] Reproduce and fix N+1 live
- [ ] Draw request flow (Filter → DispatcherServlet → Interceptor → Controller → Service(tx proxy) → Repository)
- [ ] ≥80% of questions above answerable in 2 min each
