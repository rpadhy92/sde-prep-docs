# 10 — Low-Level Design (LLD) & Design Patterns

## ⭐ Fundamentals First (start here)
**Time:** 1-month plan **6 h** · 2-month plan **9 h** (see [README](README.md)).

**Learn in this order**
1. **SOLID** — one violation + fix example each (C1) — 1 h
2. **8 essential patterns with code skeletons:** Singleton, Factory, Builder, Strategy, Observer, State, Decorator, Chain of Responsibility (C2) — 2 h
3. **The 7-step interview template** (Part B) — 30 min, then *use it every time*
4. **Practice problems:** Parking Lot → Vending Machine (State) → Notification Service → (2-month) Rate Limiter, BookMyShow, Splitwise (C4) — 2.5 h (1-month: 3 problems; 2-month: 6)
5. Concurrency-in-LLD checklist (C3) — 30 min

**Done when:** you do a new LLD problem in 45 min using the template, with clean core code and an extensibility discussion.
**Skip if short on time:** Visitor/Memento/Mediator/Flyweight/Bridge, UML formalities, 🔴 problems beyond 1–2.

> Two parts: **(A) how to prepare** and **(B) how to approach it in the interview**, then the content (SOLID, patterns, problems, code skeletons).
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

---

# PART A — How to Prepare for LLD

## A1. What interviewers evaluate
| Signal | What they look for |
|---|---|
| Requirement handling | clarifies scope, identifies use-cases, states assumptions |
| Modeling | right entities, responsibilities, relationships (not a god class) |
| OOP & SOLID | abstractions, interfaces, extensibility without modification |
| Patterns | uses patterns **where justified**, can explain why |
| Code quality | compilable core flow, naming, small methods, immutability |
| Edge cases & concurrency | validation, errors, thread-safety where relevant |
| Communication | thinks aloud, evolves design, accepts feedback |

## A2. Preparation roadmap (6 h in the 1-month plan / 9 h in the 2-month plan — the table below is the *ideal order*; compress phases for the 1-month plan)
| Phase | Days | Activity |
|---|---|---|
| 1. Foundations | 1–3 | OOP principles, **SOLID** (write a violation + refactor for each), composition vs inheritance, UML class/sequence basics |
| 2. Patterns | 4–8 | 1 pattern/day: intent → structure → Java code → **where it's used in JDK/Spring** → when *not* to use. Priority list in §C |
| 3. Problem practice | 9–13 | 1 LLD problem/day (§D). Timebox 45 min: requirements → classes → code core flow → extensions. Then compare with a reference & note gaps |
| 4. Mock & polish | 14 | Timed mock with peer / record yourself; refine template |

**Practice discipline**
1. Use a **fixed template** (Part B) every time until automatic.
2. Code in a plain editor at least half the time (no autocomplete) — interviews often use shared editors.
3. After each problem, write: *What entities? Which patterns & why? What would change if we add X?*
4. Keep a **pattern → trigger phrase** cheat-sheet (e.g., "multiple interchangeable algorithms" → Strategy; "behavior changes with state" → State; "notify many on change" → Observer; "create objects by type without `if/else`" → Factory).
5. Write unit tests for 1–2 designs — testability proves the design.
6. Read good open-source code (Spring, Guava, JDK collections) to see patterns in real use.

## A3. Pitfalls to avoid
- Over-engineering (patterns everywhere) or under-engineering (one class does everything)
- Starting to code before clarifying requirements
- Anemic God-classes (`Manager`, `Util`), static everything, global state
- Ignoring concurrency when the domain is concurrent (booking, inventory, rate-limiter)
- No extensibility discussion; hard-coded `if/else` on types
- Skipping validations/exceptions; mutable leaked internals

---

# PART B — How to Approach LLD in the Interview (7-Step Template, ~45–60 min)

| Step | Time | What to do / say |
|---|---|---|
| **1. Clarify requirements** | 5–8 min | Ask about actors, core use-cases, scale, constraints, out-of-scope. **Write functional requirements as bullets** (e.g., "park/unpark a vehicle; multiple vehicle types; fee on exit; N floors"). State assumptions. |
| **2. Identify entities & relationships** | 5 min | Nouns → classes (`ParkingLot`, `Floor`, `Spot`, `Vehicle`, `Ticket`); verbs → methods. Decide *has-a* / *is-a*, cardinalities. |
| **3. Define interfaces & key APIs** | 5 min | Public contracts first: `ParkingLot.park(Vehicle) : Ticket`, `PricingStrategy.calculate(Ticket) : Money`. Show enums/status. |
| **4. Class diagram (rough)** | 5 min | Boxes/arrows: classes, interfaces, important fields, relations. Keep it small & readable. |
| **5. Apply patterns / SOLID — only where justified** | 5 min | "Fee rules vary → **Strategy**"; "vehicle creation by type → **Factory**"; "spot lifecycle → **State**"; explain *why* and what it buys (OCP). |
| **6. Code core flow** | 15–20 min | Implement the **main use-case end-to-end** (compilable-looking Java): validation, errors, immutability where useful. Don't code every class — prioritize. |
| **7. Edge cases, concurrency, extensions, tests** | 5–8 min | Race conditions (two cars for last spot → lock/atomic), invalid inputs, failures; "How would you add EV charging / new payment method?"; what you'd unit test. |

### Say-aloud phrases
- "Let me confirm the requirements first…" · "I'll start with the simplest design that satisfies these and then extend."
- "I'm using Strategy here so adding a new pricing rule doesn't modify existing code (Open/Closed)."
- "This method needs to be thread-safe because…; I'll use a `ReentrantLock` per spot/atomic state transition."
- "Trade-off: X is simpler, Y is more extensible; given the requirements I'd choose X."

### If you're stuck
Restate requirements → list the main nouns → pick the central use-case → write its sequence of calls → derive classes from that.

---

# PART C — Content

## C1. OOP & SOLID Refresher
- [ ] Encapsulation, abstraction, inheritance, polymorphism; composition over inheritance; immutability; interfaces for roles
- [ ] **S**ingle Responsibility — one reason to change (split `Invoice` from `InvoicePrinter`)
- [ ] **O**pen/Closed — extend via new classes/strategies, not editing `if/else` chains
- [ ] **L**iskov Substitution — subtypes must honor the base contract (`Square extends Rectangle` anti-example)
- [ ] **I**nterface Segregation — small role interfaces (`Printer`, `Scanner` vs fat `Machine`)
- [ ] **D**ependency Inversion — depend on abstractions; constructor injection; enables testing
- [ ] DRY, KISS, YAGNI, Law of Demeter, high cohesion/low coupling, "tell, don't ask"
- [ ] UML basics: class (inheritance ▷, implementation ⇢, association →, aggregation ◇, composition ◆), sequence diagrams
- [ ] Layering: controller → service → repository; domain model; hexagonal/clean architecture overview

## C2. Design Patterns (★ = must know)
| Category | Patterns |
|---|---|
| **Creational** | ★ Singleton, ★ Factory Method, ★ Abstract Factory, ★ Builder, Prototype |
| **Structural** | ★ Adapter, ★ Decorator, ★ Proxy, ★ Facade, Composite, Bridge, Flyweight |
| **Behavioral** | ★ Strategy, ★ Observer, ★ Template Method, ★ Command, ★ Chain of Responsibility, ★ State, Iterator, Mediator, Memento, Visitor |

### Pattern quick-reference
| Pattern | Trigger phrase | JDK/Spring example |
|---|---|---|
| Singleton | exactly one instance (config, registry) | Spring beans (container-managed), `Runtime.getRuntime()` |
| Factory Method | create objects by type/config w/o `if/else` | `Calendar.getInstance`, `BeanFactory`, `List.of` |
| Abstract Factory | families of related objects | `DocumentBuilderFactory`, cloud-provider SDK factories |
| Builder | many optional params, readable construction | `StringBuilder`, Lombok `@Builder`, `UriComponentsBuilder` |
| Adapter | make incompatible interface fit | `Arrays.asList`, Spring `HandlerAdapter` |
| Decorator | add behavior dynamically, stack wrappers | `BufferedReader`, `Collections.unmodifiableList` |
| Proxy | control access / lazy / cross-cutting | Spring AOP, `@Transactional`, JPA lazy loading |
| Facade | simple front for complex subsystem | service-layer façade, `JdbcTemplate` |
| Composite | tree of uniform objects | file system, UI components |
| Strategy | interchangeable algorithms | `Comparator`, `PasswordEncoder`, pricing/discount rules |
| Observer | one-to-many notification | Spring `ApplicationEvent`, listeners, reactive streams |
| Template Method | skeleton algorithm, steps overridden | `JdbcTemplate`, `AbstractList`, `HttpServlet` |
| Command | encapsulate request (undo/queue) | `Runnable`, job queues, undo/redo |
| Chain of Responsibility | pass request along handlers | Servlet `Filter` chain, Spring Security filters, logging levels |
| State | behavior changes with state | order/vending/ATM state machines |
| Iterator | traverse without exposing structure | `Iterator`, `Iterable` |

### Code skeletons (commonly asked)
**Thread-safe Singleton (Bill Pugh) / Enum**
```java
public final class Config {
    private Config() {}
    private static class Holder { static final Config INSTANCE = new Config(); }
    public static Config getInstance() { return Holder.INSTANCE; }
}
public enum AppRegistry { INSTANCE; /* serialization- & reflection-safe */ }
```
**Strategy (pricing)**
```java
interface PricingStrategy { Money calculate(Duration parked, VehicleType type); }
class HourlyPricing implements PricingStrategy { /* ... */ }
class FlatRatePricing implements PricingStrategy { /* ... */ }
class ParkingLot { private final PricingStrategy pricing;
    ParkingLot(PricingStrategy pricing) { this.pricing = pricing; } }
```
**State (vending machine)**
```java
interface State { void insertCoin(VendingMachine m); void selectItem(VendingMachine m, String code); void dispense(VendingMachine m); }
class IdleState implements State { public void insertCoin(VendingMachine m) { m.setState(new HasMoneyState()); } /* others → error */ }
```
**Observer**
```java
interface OrderListener { void onStatusChanged(Order o); }
class Order { private final List<OrderListener> listeners = new CopyOnWriteArrayList<>();
    void setStatus(Status s) { this.status = s; listeners.forEach(l -> l.onStatusChanged(this)); } }
```
**Chain of Responsibility (logger/validator)**
```java
abstract class Handler { protected Handler next; Handler then(Handler n) { next = n; return n; }
    abstract void handle(Request r); protected void pass(Request r) { if (next != null) next.handle(r); } }
```
**Builder**
```java
public final class User {
    private final String name; private final String email; private final int age;
    private User(Builder b) { name = b.name; email = b.email; age = b.age; }
    public static Builder builder() { return new Builder(); }
    public static final class Builder { private String name, email; private int age;
        public Builder name(String n) { this.name = n; return this; }
        public Builder email(String e) { this.email = e; return this; }
        public Builder age(int a) { this.age = a; return this; }
        public User build() { Objects.requireNonNull(name); return new User(this); } }
}
```

## C3. Concurrency in LLD (often the differentiator)
- Make state transitions **atomic**: `AtomicReference`/`compareAndSet`, `synchronized`/`ReentrantLock` per entity (fine-grained), `ConcurrentHashMap.compute`.
- Avoid check-then-act races (`if (spot.isFree()) spot.occupy()` → use `tryOccupy()` that is atomic).
- Prefer **immutable** value objects; confine mutable state inside one class.
- Lock ordering to avoid deadlock; timeouts with `tryLock`.
- Idempotency for retried operations (booking, payment).

## C4. LLD Practice Problems
| Level | Problem | Showcases |
|---|---|---|
| 🟡 | Parking Lot | Strategy (pricing), Factory (vehicle/spot), concurrency on spot allocation |
| 🟡 | Library Management | entities, reservations, Observer (due-date notifications) |
| 🟡 | Vending Machine | **State** |
| 🟡 | ATM | State, Chain of Responsibility (cash dispenser), transactions |
| 🟡 | Logger Framework | Singleton, Chain of Responsibility, Strategy (appenders) |
| 🟡 | Tic-Tac-Toe / Chess (core) | clean OOP, extensibility (N×N, players) |
| 🟡 | Notification Service (Email/SMS/Push) | Strategy, Factory, Observer, retry |
| 🟡 | Shopping Cart + Discounts/Coupons | Strategy/Decorator, pricing rules |
| 🔴 | Elevator System | State, Strategy (scheduling), concurrency |
| 🔴 | Rate Limiter | Strategy (token bucket / sliding window), thread-safety |
| 🔴 | LRU / LFU Cache | data structures, generics, thread-safety |
| 🔴 | BookMyShow / Ticket Booking | seat locking, concurrency, TTL holds |
| 🔴 | Splitwise (expense sharing) | Strategy (split types), debt simplification |
| 🔴 | Food Delivery / Ride Matching | Strategy (matching), Observer, State |
| 🔴 | In-memory Pub-Sub / Task Scheduler / Cron | Observer, concurrency, `PriorityQueue` + threads |
| 🔴 | Online Payment System / Wallet | idempotency, ledger, State |
| 🔴 | Key-Value store with TTL / transactions | structure design, expiry (lazy + background) |

**Worked mini-example — Parking Lot (outline to memorize)**
- Requirements: multiple floors; spot types (compact/large/EV); vehicle types; issue ticket on entry; fee on exit; display availability.
- Entities: `ParkingLot` (singleton/service), `Floor`, `ParkingSpot` (type, state), `Vehicle` (type), `Ticket` (id, spot, entryTime), `Payment`.
- Interfaces: `SpotAllocationStrategy` (nearest/first-fit), `PricingStrategy`, `PaymentProcessor`.
- Core flow: `park(vehicle)` → find spot via strategy → atomically occupy → create ticket. `unpark(ticket)` → compute fee → pay → free spot.
- Concurrency: `ParkingSpot.tryOccupy()` using `AtomicBoolean`/lock; allocation per spot-type queue.
- Extensions: EV charging, reservations, multi-entry gates, dynamic pricing, per-hour caps.

## C5. Question Bank
1. 🟢 Explain SOLID with an example from your project.
2. 🟢 Singleton: ways to implement; how can it be broken (reflection, serialization, clone, multiple classloaders)? how to prevent?
3. 🟡 Factory vs Abstract Factory vs Builder?
4. 🟡 Strategy vs State vs Template Method — differences?
5. 🟡 Decorator vs Proxy vs Adapter vs Facade?
6. 🟡 Composition vs inheritance; give a case where inheritance hurt you.
7. 🟡 How would you add a new payment method without modifying existing code (OCP)?
8. 🟡 Where do patterns appear in Spring (list 5)?
9. 🔴 Design a flexible pricing/discount engine (rules, stacking, priority, audit).
10. 🔴 How do you make an LLD thread-safe *and* testable (DI, immutable, clock injection)?
11. 🔴 Anemic domain model vs rich domain model?
12. 🔴 Critique this class (given 80 lines of messy code) — identify violations & refactor.
13. 🔴 Design an extensible notification system with retries, templates, user preferences, rate limits.

## C6. Self-Check
- [ ] Recite the 7-step template; apply to a new problem in 45 min
- [ ] Code 8 pattern skeletons from memory (Singleton, Factory, Builder, Strategy, Observer, State, Decorator, Chain)
- [ ] Explain each SOLID principle with violation + fix
- [ ] 6+ problems done end-to-end with code and a short "what I'd extend" note
