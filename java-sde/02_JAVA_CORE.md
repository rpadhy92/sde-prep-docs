# 02 — Core Java Concepts

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **7 h** · 2-month plan **11 h** (see [README](README.md)).

**Learn in this order (the must-know core)**
1. **OOP + `equals`/`hashCode`, immutability, abstract class vs interface** (§1) — 1 h
2. **Collections internals — `HashMap` above all** (§3): hashing, buckets, load factor, resize, treeification, `ConcurrentHashMap` — 2 h
3. **Concurrency basics** (§4): thread states, `synchronized`, `volatile`, `ExecutorService`/`ThreadPoolExecutor` flow, `CompletableFuture`, deadlock — 2 h
4. **Java 8+** (§5): lambdas, streams, `Optional`, records — 1 h
5. **Exceptions** (§2) — 30 min · then quiz yourself with 15 questions from §7.

**Done when:** you can explain, without notes — HashMap internals, why override `hashCode` with `equals`, `volatile` vs `synchronized`, thread-pool behavior, `map` vs `flatMap`.
**Skip if short on time:** Phaser/StampedLock, NIO.2, annotations/reflection deep-dive, virtual-thread internals (know *what* and *why* only).

> JVM memory & Garbage Collection are in [03_GC_JVM.md](03_GC_JVM.md).
> Coding practice for collections lives in [01_CODING.md](01_CODING.md).
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Study per topic** with the checklist; write a **½-page note** in your own words.
2. **Answer format:** *Definition → internals → tiny code example → trade-off/pitfall → where you used it.*
3. **Prove it in code** — write a 20–40 line demo for each 🔴 topic (e.g., deadlock, memory leak with `ThreadLocal`, HashMap with bad `hashCode`).
4. **Self-quiz** with the question list at the end; speak answers aloud; log misses.
5. **Time:** 7 h (1-month) / 11 h (2-month). First pass = the ⭐ box; second pass = §1–6 checklists; third pass = §7 questions.

---

## 1. Language Fundamentals & OOP
- [ ] JDK vs JRE vs JVM; source → bytecode → JIT/interpretation
- [ ] Primitives vs wrappers; autoboxing pitfalls (`Integer` cache −128..127; `==` vs `equals`; NPE on unboxing null)
- [ ] `String`: immutability, string pool, `intern()`, `String` vs `StringBuilder` vs `StringBuffer`, `compact strings`
- [ ] 4 OOP pillars with real examples; **composition over inheritance**
- [ ] Overloading vs overriding; covariant returns; static vs dynamic binding; method hiding
- [ ] `static`, `final`, `finally`, `finalize` (deprecated); `this`/`super`; initialization order (static block → instance block → constructor, parent first)
- [ ] Abstract class vs interface (default/static/private methods); when to use which
- [ ] Access modifiers; inner / nested / local / anonymous classes; effectively-final capture
- [ ] **Immutability** — design recipe & why it helps thread-safety
- [ ] `equals()`/`hashCode()` contract; impact on `HashMap`/`HashSet`
- [ ] Pass-by-value (even for references)
- [ ] Cloning: shallow vs deep; copy constructors; why `Cloneable` is discouraged
- [ ] **Generics:** type erasure, bounded types, wildcards, **PECS** (`? extends` producer / `? super` consumer)
- [ ] Enums (fields, abstract methods, `EnumMap`/`EnumSet`, enum singleton)
- [ ] Annotations (retention/target, custom), reflection basics
- [ ] Serialization: `Serializable`, `serialVersionUID`, `transient`, security risks; prefer JSON

## 2. Exception Handling
- [ ] Checked vs unchecked; `Error` vs `Exception`
- [ ] try/catch/finally flow; `return` in `finally` gotcha
- [ ] try-with-resources, `AutoCloseable`, suppressed exceptions
- [ ] Custom exceptions; wrapping (cause chain); don't swallow; don't log-and-rethrow
- [ ] Overriding rules with exceptions; multi-catch

## 3. Collections Framework
- [ ] Hierarchy: `Collection` → `List/Set/Queue/Deque`; `Map` separate
- [ ] `ArrayList` (growth 1.5×) vs `LinkedList` vs `Vector`; `CopyOnWriteArrayList`
- [ ] **`HashMap` internals:** `hash()` spreading, table index `(n-1) & hash`, load factor 0.75, resize (doubling), collision chaining, **treeification (≥8 nodes & table ≥64)**, null key bucket 0
- [ ] `HashMap` vs `Hashtable` vs `ConcurrentHashMap` (Java 8: CAS + synchronized per bin; no null keys/values)
- [ ] `LinkedHashMap` (insertion/access order → LRU), `TreeMap` (red-black, O(log n)), `WeakHashMap`, `IdentityHashMap`
- [ ] `HashSet`/`LinkedHashSet`/`TreeSet` (backed by maps)
- [ ] `PriorityQueue` (binary heap), `ArrayDeque`
- [ ] Fail-fast vs fail-safe iterators; `ConcurrentModificationException`; `removeIf`
- [ ] `Comparable` vs `Comparator`; `Collections`/`Arrays` utilities
- [ ] Immutable collections: `List.of` vs `Collections.unmodifiableList` (view vs copy)
- [ ] Big-O table of all common operations (memorize)

| Structure | get | add | remove | contains |
|---|---|---|---|---|
| ArrayList | O(1) | O(1) amortized (end) | O(n) | O(n) |
| LinkedList | O(n) | O(1) at ends | O(1) at ends / O(n) middle | O(n) |
| HashMap/HashSet | O(1) avg | O(1) avg | O(1) avg | O(1) avg |
| TreeMap/TreeSet | O(log n) | O(log n) | O(log n) | O(log n) |
| PriorityQueue | peek O(1) | O(log n) | poll O(log n) / remove(obj) O(n) | O(n) |

## 4. Multithreading & Concurrency (Senior Core)
- [ ] Thread lifecycle & states; `Thread` vs `Runnable` vs `Callable`
- [ ] `synchronized` (method/block/static), monitors, reentrancy
- [ ] **`volatile`:** visibility + ordering (not atomicity); Java Memory Model; **happens-before**
- [ ] `wait/notify/notifyAll` (always in a loop); `sleep` vs `wait`; `join`; interrupts
- [ ] Race condition, deadlock, livelock, starvation — causes/prevention (lock ordering, tryLock with timeout)
- [ ] `ReentrantLock`, `ReadWriteLock`, `StampedLock`, `Condition`
- [ ] Atomics (`AtomicInteger`, `LongAdder`), CAS & ABA
- [ ] `ThreadLocal` & leak risk with thread pools (`remove()`)
- [ ] **Executor framework:** `ThreadPoolExecutor` (core, max, queue, keep-alive, rejection policy), flow: core → queue → max → reject; why `Executors.newFixedThreadPool` unbounded queue is risky
- [ ] `Future`, **`CompletableFuture`** (`supplyAsync`, `thenApply/thenCompose/thenCombine`, `allOf/anyOf`, `exceptionally/handle`, custom executor, timeouts)
- [ ] Synchronizers: `CountDownLatch`, `CyclicBarrier`, `Semaphore`, `Phaser`
- [ ] Concurrent collections: `ConcurrentHashMap`, `BlockingQueue` family, `CopyOnWrite*`, `ConcurrentLinkedQueue`
- [ ] Fork/Join, parallel streams — when *not* to use
- [ ] **Virtual threads (Java 21)** & structured concurrency: what/why/when; pinning with `synchronized`
- [ ] Safe design: immutability, thread confinement, stateless services, lock-free where simple

## 5. Java 8 → 21 Features
- [ ] **Java 8:** lambdas, functional interfaces (`Function`, `Predicate`, `Consumer`, `Supplier`, `BiFunction`, `UnaryOperator`), method refs, **Streams**, `Optional`, default methods, `java.time`, `CompletableFuture`
- [ ] Streams: lazy pipeline; `map/flatMap/filter/reduce/collect`; `Collectors` (`groupingBy`, `partitioningBy`, `toMap`, `joining`); primitive streams; parallel pitfalls
- [ ] `Optional` best practices (return type only; `orElseGet` vs `orElse`; avoid `get()`)
- [ ] **9–11:** modules (overview), `List.of/Map.of`, `var`, `String` APIs (`isBlank`, `strip`, `lines`, `repeat`), HTTP Client
- [ ] **14–17:** **records**, **sealed classes**, pattern matching `instanceof`, text blocks, switch expressions
- [ ] **21:** virtual threads, sequenced collections, record patterns, pattern matching for `switch`
- [ ] LTS: 8, 11, 17, 21 — why many companies run 8/11, migration pain points (`javax` → `jakarta`, removed APIs)

## 6. Misc
- [ ] I/O vs NIO vs NIO.2 (`Files`, `Path`, channels); buffered streams
- [ ] Reflection & dynamic proxies (basis of Spring AOP)
- [ ] Logging: SLF4J + Logback, levels, MDC, don't log secrets/PII
- [ ] Maven: lifecycle, scopes, dependency mediation, BOMs; Gradle basics
- [ ] Testing → [05_JUNIT_MOCKITO.md](05_JUNIT_MOCKITO.md)

---

## 7. Question Bank (self-quiz — answer aloud)

### OOP / Language
1. 🟢 Abstract class vs interface (post-Java 8)? Why were default methods added?
2. 🟢 Why is `String` immutable? How do you make a custom immutable class? Is it *truly* immutable (reflection)?
3. 🟢 `==` vs `equals`; `Integer` cache surprise (`127 == 127` true, `128 == 128` false)?
4. 🟡 Why override `hashCode` with `equals`? What breaks otherwise?
5. 🟡 Overloading vs overriding with `null` argument / varargs ambiguity?
6. 🟡 Shallow vs deep copy — how to deep copy?
7. 🟡 What is type erasure? Why can't you `new T()` or `List<int>`?
8. 🟡 Explain PECS with `Collections.copy`.
9. 🔴 Why prefer composition over inheritance? Example of fragile base class.
10. 🔴 Records/sealed classes: when to use, when not to?

### Exceptions
11. 🟢 Checked vs unchecked — which for business validation and why?
12. 🟡 What if both `try` and `finally` throw/return? Output of a tricky snippet.
13. 🟡 try-with-resources: close order & suppressed exceptions?

### Collections
14. 🟢 `ArrayList` vs `LinkedList` — when does LinkedList actually win?
15. 🟡 **How does `HashMap` work internally?** Same hash for two keys? What changed in Java 8?
16. 🟡 `HashMap` vs `ConcurrentHashMap` vs `Hashtable` vs `Collections.synchronizedMap`?
17. 🟡 Fail-fast vs fail-safe iterators; how to remove during iteration safely?
18. 🟡 `TreeMap` vs `HashMap` vs `LinkedHashMap` — choose for sorted/insertion-order/lookup-heavy workloads.
19. 🔴 What happens when `HashMap` resizes while multiple threads `put`?
20. 🔴 How would you implement an LRU cache? Make it thread-safe?

### Concurrency
21. 🟡 `volatile` vs `synchronized` vs `AtomicInteger`?
22. 🟡 Explain happens-before. Why is double-checked locking broken without `volatile`?
23. 🟡 `sleep` vs `wait`; why `wait()` must be in a `while` loop?
24. 🟡 How does `ThreadPoolExecutor` pick between new thread / queue / reject?
25. 🟡 `execute` vs `submit` — what happens to exceptions?
26. 🟡 `CompletableFuture` `thenApply` vs `thenCompose` vs `thenCombine`; exception handling; which pool runs the stage?
27. 🔴 Walk through diagnosing a production deadlock (`jstack`, thread states, lock owners).
28. 🔴 Design a thread pool for I/O-bound vs CPU-bound tasks — sizing formula? (CPU: N+1; I/O: N × (1 + wait/compute))
29. 🔴 `ThreadLocal` leak scenario in a servlet container?
30. 🔴 Virtual threads vs platform threads; what workloads benefit; what's pinning?

### Java 8+
31. 🟡 `map` vs `flatMap`; why can't a stream be reused; `peek` misuse?
32. 🟡 `Optional.orElse` vs `orElseGet`; why not use `Optional` as a field/param?
33. 🟡 When is `parallelStream()` harmful?
34. 🟡 Functional interfaces you used; write a custom one.
35. 🔴 How are lambdas compiled (invokedynamic) vs anonymous classes?

### Design/Practical
36. 🟡 How do you design a class to be safely used by multiple threads?
37. 🟡 How do you handle `InterruptedException` correctly?
38. 🔴 How would you find & fix a memory leak? (see [03_GC_JVM.md](03_GC_JVM.md))
39. 🔴 Java 8 → 17/21 migration: what breaks, what do you gain?
40. 🔴 Your thread pool queue grows unbounded in prod — causes & fixes (back-pressure, bounded queue + `CallerRunsPolicy`, rate limiting)?

---

## 8. Self-Check
- [ ] Draw HashMap internals (buckets, nodes, tree bin, resize) in 2 min
- [ ] Write deadlock + fix, producer–consumer, thread-safe singleton from memory
- [ ] Explain `ThreadPoolExecutor` flow & sizing with numbers
- [ ] Rewrite a 15-line imperative loop as streams *and* explain when not to
- [ ] Answer ≥32/40 questions above confidently
