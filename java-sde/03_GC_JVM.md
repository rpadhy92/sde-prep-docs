# 03 — JVM Memory & Garbage Collection

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **3 h** · 2-month plan **5 h** (see [README](README.md)).

**Learn in this order**
1. **Memory areas** (§1): heap (Young/Old), Metaspace, thread stacks — and which error each produces. (30 min)
2. **GC basics** (§2): reachability from GC roots, generational hypothesis, Minor vs Major vs Full GC, stop-the-world. (45 min)
3. **Collectors** (§3): Parallel vs **G1** (default) vs ZGC — one-line "when to use". (45 min)
4. **Troubleshooting** (§5): workflow + the OOM/latency scenario table; know `jstack`, `jmap`/heap dump + MAT, GC logs. (45–60 min)

**Done when:** you can draw the heap + object lifecycle, compare Parallel/G1/ZGC, and narrate an OOM or latency-spike investigation step by step.
**Skip if short on time:** card tables/write barriers, Shenandoah/CMS details, JIT/escape analysis, tuning flag minutiae (know `-Xms/-Xmx`, `MaxRAMPercentage`, heap-dump-on-OOM).

> Senior interviews probe: "Explain GC", "Which collector and why?", "Prod OOM / long pauses — how do you debug?". Goal: confident mental model + a concrete diagnostic workflow + real flags/tools.
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## How to Prepare (Method)
1. **Day 1–2:** memory model + GC fundamentals (§1–2). Draw the heap layout from memory.
2. **Day 3:** collectors (§3) — build a comparison table in your own words.
3. **Day 4:** run a tiny program with `-Xlog:gc*` and read the log; force an OOM and take a heap dump (§6 lab).
4. **Day 5:** troubleshooting scenarios (§5) — practice speaking the workflow.
5. **Day 6:** question bank (§7); log misses.
Time: 3 h (1-month: do the ⭐ box + §5; skip the lab if needed) / 5 h (2-month: include the §6 hands-on lab).

---

## 1. JVM Runtime Memory Areas
| Area | Holds | Shared? | Errors |
|---|---|---|---|
| **Heap** | all objects, arrays (Young + Old) | all threads | `OutOfMemoryError: Java heap space` |
| **Metaspace** (native memory) | class metadata, method info (replaced PermGen in Java 8) | all | `OutOfMemoryError: Metaspace` |
| **Thread stack** | frames: locals, operand stack, return addresses | per thread | `StackOverflowError`, `OOM: unable to create native thread` |
| **PC register** | current instruction | per thread | — |
| **Native method stack** | native calls (JNI) | per thread | — |
| **Code cache** | JIT-compiled code | all | JIT stops if full |
| **Direct/off-heap memory** | `ByteBuffer.allocateDirect`, Netty buffers | all | `OOM: Direct buffer memory` |

- [ ] Class loading: Bootstrap → Platform → Application; parent-delegation; `ClassNotFoundException` vs `NoClassDefFoundError`; class unloading & Metaspace leaks (redeploys)
- [ ] Object layout: header (mark word + class pointer), fields, padding; compressed oops (<32 GB heap)
- [ ] Escape analysis & scalar replacement (stack allocation illusion), TLABs (thread-local allocation buffers → fast allocation)
- [ ] String pool lives on heap (since Java 7)

## 2. GC Fundamentals
- [ ] **Why GC:** automatic reclamation of unreachable objects.
- [ ] **Reachability:** GC roots (thread stacks/locals, static fields, JNI refs, active threads) → traverse → **mark** live → reclaim rest. Cycles are collected (not ref-counting).
- [ ] **Generational hypothesis:** most objects die young → split heap into **Young** (Eden + S0 + S1) and **Old/Tenured**.
- [ ] **Object lifecycle:** allocate in Eden → Minor GC copies survivors to S0/S1 (age++) → after `MaxTenuringThreshold` (or survivor overflow) promoted to Old → Old fills → Major/Mixed/Full GC.
- [ ] **Minor GC** (young only; short STW) vs **Major/Old GC** vs **Full GC** (whole heap + Metaspace; longest STW).
- [ ] **Stop-the-world (STW)** pauses; safepoints; **latency vs throughput** vs **footprint** trade-off.
- [ ] Basic algorithms: **Mark-Sweep** (fragmentation), **Mark-Compact** (move live objects, pointer update), **Copying** (young gen, fast, needs space).
- [ ] **Card table / remembered sets / write barriers:** let young GC avoid scanning the entire old gen (old→young references).
- [ ] **Concurrent vs parallel vs incremental** collection (parallel = many GC threads while app paused; concurrent = GC threads run alongside app).
- [ ] **Reference types:** Strong, Soft (cache; cleared before OOM), Weak (`WeakHashMap`, cleared next GC), Phantom (cleanup via `Cleaner`; replaces `finalize`).
- [ ] **Finalization** deprecated: unpredictable, slows GC → use try-with-resources/`Cleaner`.
- [ ] **Premature promotion / promotion failure / concurrent mode failure / humongous allocation** (G1) — know the names.

## 3. Collectors (Comparison)
| Collector | Flag | Type | Best for | Notes |
|---|---|---|---|---|
| Serial | `-XX:+UseSerialGC` | single-thread, STW | tiny heaps, containers with 1 CPU | simple |
| Parallel (Throughput) | `-XX:+UseParallelGC` | multi-thread STW | batch jobs, max throughput | default in Java 8 |
| **G1** | `-XX:+UseG1GC` | region-based, mostly concurrent, mixed collections | general server apps, 4–64 GB heaps | **default since Java 9**; pause target `-XX:MaxGCPauseMillis=200` |
| **ZGC** | `-XX:+UseZGC` (+ `-XX:+ZGenerational` in 21) | concurrent, colored pointers/load barriers | very low latency (sub-ms pauses), large heaps (to TBs) | production since 15; generational in 21 |
| **Shenandoah** | `-XX:+UseShenandoahGC` | concurrent compaction | low pause (OpenJDK/Red Hat builds) | similar goals to ZGC |
| CMS | removed in Java 14 | — | legacy | know historically (fragmentation, concurrent mode failure) |

### G1 in a nutshell (most likely asked)
- Heap divided into ~2048 equal **regions** (1–32 MB); each region is Eden/Survivor/Old/**Humongous** (objects ≥ 50% region size).
- **Young collections** (STW, parallel copying) → **concurrent marking** cycle when old occupancy hits `InitiatingHeapOccupancyPercent` (default 45%) → **mixed collections** (young + the *most garbage-rich* old regions — "Garbage First").
- Aims to meet `MaxGCPauseMillis` by choosing how many regions to collect.
- **Failure mode: Full GC** (to-space exhausted / evacuation failure / humongous churn) → huge pause → fix by more headroom, lowering IHOP, avoiding humongous objects, increasing heap, or ZGC.

### Choosing a collector (answer pattern)
- Batch/ETL, throughput first → **Parallel**.
- Typical microservice, balanced → **G1** (default; tune little).
- Strict p99/p999 latency, large heap → **ZGC**/Shenandoah.
- Tiny container (≤2 CPU, ≤512 MB) → **Serial** or G1 with small heap.

## 4. Sizing & Tuning
### Common flags
```
-Xms2g -Xmx2g                    # set equal in prod → avoid resize pauses
-XX:+UseG1GC -XX:MaxGCPauseMillis=200
-XX:MaxRAMPercentage=75.0        # container-aware sizing (instead of fixed Xmx)
-Xss512k                         # thread stack size
-XX:MaxMetaspaceSize=256m
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/dumps
-XX:+ExitOnOutOfMemoryError      # let Kubernetes restart the pod
-Xlog:gc*:file=gc.log:time,uptime,level,tags:filecount=5,filesize=20m    # Java 9+ unified logging
```
(Java 8: `-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log`.)

### Tuning principles
1. **Measure first** (GC logs + metrics); never tune blindly.
2. Fix **allocation rate/ object churn** in code before changing flags (reuse buffers, avoid boxing, avoid huge temp collections, stream large results).
3. Set realistic goals: target throughput (% time not in GC, aim >95–98%) and p99 pause.
4. Change **one flag at a time**; load-test.
5. Containers: JVM is container-aware (Java 10+/8u191+). Leave room for Metaspace, threads, direct buffers, code cache → heap ≈ 50–75% of container memory, otherwise **OOMKilled** by the kernel (no Java OOM!).

## 5. Troubleshooting Playbook

### Workflow (say this in interviews)
1. **Symptom:** latency spikes? high CPU? OOM? container restarts?
2. **Metrics:** heap used after GC (the "live set" trend), GC pause time/frequency, allocation rate (Micrometer `jvm.gc.*`, Grafana).
3. **GC logs:** pause types (Young / Mixed / Full), reasons, before→after sizes.
4. **Thread dump (`jstack`/`jcmd Thread.print`)**: are threads blocked/GC threads busy?
5. **Heap dump (`jmap -dump` / `jcmd GC.heap_dump`)** → analyze in **Eclipse MAT / VisualVM**: dominator tree, leak suspects, GC-root paths.
6. **Profile (JFR / async-profiler):** allocation hot spots.
7. **Fix → verify** under load → add alert.

### Scenarios
| Scenario | Likely causes | Actions |
|---|---|---|
| Heap usage **keeps rising after each Full GC** (saw-tooth trending up) → eventual `OOM: Java heap space` | Memory **leak**: static maps/caches without eviction, listeners not deregistered, `ThreadLocal` in pools, unclosed resources, session objects, unbounded queues | Heap dump → MAT leak suspects → fix root-path; use bounded caches (Caffeine), `WeakReference`, `remove()` ThreadLocal |
| **Frequent long Full GCs**, heap nearly full but no leak | Heap too small / high live set; premature promotion; humongous allocations (G1) | Increase heap; tune young size; fix large object allocations; consider ZGC |
| **Latency spikes (p99)** aligned with GC | Long STW (Full/mixed GC), big heap with Parallel | Move to G1/ZGC; lower pause target; reduce allocation; check swapping |
| `OOM: GC overhead limit exceeded` | >98% time in GC reclaiming <2% | Same as leak/undersized heap |
| `OOM: Metaspace` | class loader leak (hot redeploys), dynamic proxies/CGLIB/bytecode gen in a loop | Cap `MaxMetaspaceSize`, find loader leak, avoid regenerating classes |
| `OOM: unable to create native thread` | too many threads / small `ulimit`, big `-Xss` | Bound thread pools; use virtual threads; raise ulimit; reduce `-Xss` |
| `OOM: Direct buffer memory` | Netty/NIO buffers unreleased | Release `ByteBuf`; cap `-XX:MaxDirectMemorySize` |
| Pod **OOMKilled** with heap well below `-Xmx` | Non-heap memory (Metaspace, threads, direct) + `Xmx` ≈ container limit | Lower `MaxRAMPercentage`; check native memory (`-XX:NativeMemoryTracking=summary`) |
| High CPU in GC threads | high allocation rate, undersized young gen | Reduce allocation; right-size heap; check GC threads vs CPUs |
| `StackOverflowError` | unbounded recursion; deep object graphs; huge call chains | Fix recursion; bounded depth; (last resort) larger `-Xss` |

### Common leak patterns (name 3+)
Static collections (`static Map` caches), un-removed listeners/callbacks, `ThreadLocal` not removed with thread pools, unclosed streams/connections/`ResultSet`, inner-class references (non-static inner holds outer), `ClassLoader` leaks, unbounded queues/`ConcurrentHashMap` keyed by request data, `String.intern` misuse, `ArrayList.subList` retaining a huge parent (older JDKs).

### Tools
| Tool | Use |
|---|---|
| `jps`, `jcmd` | list JVMs; run diagnostics (`jcmd <pid> GC.heap_info`, `Thread.print`, `GC.class_histogram`) |
| `jstat -gcutil <pid> 1000` | live generation occupancy & GC counts/time |
| `jstack` | thread dump / deadlock detection |
| `jmap -histo` / `-dump` | object histogram / heap dump |
| **JFR** (Java Flight Recorder) + JMC | low-overhead production profiling (allocations, GC, locks) |
| **Eclipse MAT** | heap dump analysis (dominator tree, leak suspects) |
| VisualVM / JConsole | live monitoring |
| GCViewer / GCeasy | visualize GC logs |
| async-profiler | CPU / allocation flame graphs |
| Micrometer + Prometheus/Grafana | `jvm_memory_used_bytes`, `jvm_gc_pause_seconds`, alerts |

## 6. Hands-On Lab (do it — it gives you stories)
1. Write a program that adds objects to a `static List` in a loop. Run with `-Xmx64m -Xlog:gc* -XX:+HeapDumpOnOutOfMemoryError`. Observe the log → OOM → open `.hprof` in MAT → find the leaking list.
2. Run a REST endpoint that allocates lots of short-lived objects; compare G1 vs Parallel vs ZGC pause times with `-Xlog:gc`.
3. Allocate objects ≥ half region size under G1 → observe **humongous** allocations in the log.
4. Create a `ThreadLocal` leak in a fixed thread pool; confirm via heap dump.
5. Practice reading a log line:
```
[2.345s][info][gc] GC(12) Pause Young (Normal) (G1 Evacuation Pause) 512M->128M(1024M) 8.421ms
```
→ Young pause, heap 512 MB → 128 MB, committed 1024 MB, pause 8.4 ms.

## 7. Question Bank
1. 🟢 What is GC? How does it decide an object is garbage? (reachability from GC roots)
2. 🟢 Heap structure (Young/Eden/Survivor/Old); PermGen vs Metaspace?
3. 🟢 Minor vs Major vs Full GC; what is stop-the-world?
4. 🟡 Walk through an object's life from allocation to promotion/collection.
5. 🟡 Explain Mark-Sweep-Compact vs Copying. Why is copying used for young gen?
6. 🟡 Serial vs Parallel vs G1 vs ZGC — which and why?
7. 🟡 **How does G1 work?** regions, mixed collections, pause target, humongous objects.
8. 🟡 `System.gc()` — what does it do and why avoid it?
9. 🟡 Strong/soft/weak/phantom references — use-cases?
10. 🟡 How can memory leak in a GC language? Give 3 examples.
11. 🟡 Which JVM flags do you set in production and why (`Xms=Xmx`, `MaxRAMPercentage`, heap dump on OOM)?
12. 🔴 Production service has p99 latency spikes every few minutes — how do you investigate?
13. 🔴 Heap dump analysis: what do you look at in MAT (dominator tree, retained size, path to GC roots)?
14. 🔴 Pod keeps getting OOMKilled but heap looks fine — why?
15. 🔴 What are card tables / write barriers and why needed?
16. 🔴 What causes premature promotion; how do you tune the young generation size/survivor ratio?
17. 🔴 ZGC vs G1: trade-offs (throughput overhead vs pause), when you'd migrate?
18. 🔴 How do you reduce GC pressure *in code*? (object reuse/pooling carefully, primitives over boxed, avoid string concat in loops, pre-size collections, streams on huge data, avoid large temporary lists)
19. 🔴 `OutOfMemoryError: GC overhead limit exceeded` — what does it mean?
20. 🔴 Does `finalize()` run reliably? What replaces it?

## 8. Self-Check
- [ ] Draw heap + object-lifecycle diagram and explain in 2 min
- [ ] Compare Parallel/G1/ZGC in a table from memory
- [ ] Read a G1 log line and a `jstat -gcutil` output
- [ ] Describe full OOM investigation workflow with exact tools/commands
- [ ] Have one **real story** (yours or the lab) of a GC/memory issue: symptom → tool → root cause → fix
