# 05 — JUnit 5, Mockito & Spring Boot Testing

> Structure: **Part A** JUnit 5 concepts → **Part B** Mockito concepts (normal classes & methods) → **Part C** Mocking static & private → **Part D** Spring Boot testing concepts (application context, controller, service, JPA).
> Legend: 🟢 basic · 🟡 intermediate · 🔴 senior-depth

## ⭐ Start Here
**Time:** 1-month plan **3 h** · 2-month plan **5 h** (see [README](README.md)).

| Part | Learn | 1-month | 2-month |
|---|---|---|---|
| **A** | JUnit 5 concepts | 40 min | 50 min |
| **B** | Mockito concepts (normal classes & methods) | 60 min | 80 min |
| **C** | Mocking static & private | 20 min | 30 min |
| **D** | Spring Boot testing (controller, service, JPA) | 60 min | 100 min |
| | **Total** | **3 h** | **5 h** |

**Done when:** write a Mockito service test (success + failure + `verify never`) in <10 min; explain mock vs spy; mock a static call; explain why private methods aren't mocked; pick the right Spring test annotation for controller / service / repository.

**Dependencies:** Spring Boot → `spring-boot-starter-test` (bundles JUnit 5, Mockito, AssertJ, Spring Test, JsonPath, Awaitility). Plain Java → `junit-jupiter`, `mockito-junit-jupiter`, `assertj-core`.

---

# Part A — JUnit 5 Concepts

## A1. What is JUnit 5
- **JUnit 5 = Platform + Jupiter + Vintage.** Platform = launcher (IDE/Maven/Gradle integrate). Jupiter = new API (`org.junit.jupiter.api`). Vintage = backward compat for JUnit 3/4.
- Test classes and methods can be **package-private** (no `public` needed).
- A **new test-class instance is created per test method** (`PER_METHOD`, default) → instance fields are not shared between tests.

## A2. Test structure — AAA
- **Arrange → Act → Assert** (or Given-When-Then). One behavior per test.
- Name by behavior: `shouldReturnX_whenY()` or `@DisplayName("returns X when Y")`.
- **Test pyramid:** many fast unit tests → fewer integration/slice → very few E2E.
- **FIRST properties:** Fast, Independent, Repeatable, Self-validating, Timely.

## A3. Lifecycle annotations
| Annotation | Runs | Notes |
|---|---|---|
| `@BeforeAll` | once before all tests | must be `static` (unless `@TestInstance(PER_CLASS)`) |
| `@BeforeEach` | before each `@Test` | fresh state per test |
| `@Test` | the test method | must return `void`, not `private` |
| `@AfterEach` | after each test | cleanup per-test resources |
| `@AfterAll` | once after all tests | must be `static` |
Order with inheritance: superclass `@BeforeEach` → subclass `@BeforeEach` → test → subclass `@AfterEach` → superclass `@AfterEach`.

## A4. Assertions
| Assertion | Use |
|---|---|
| `assertEquals(expected, actual)` | equality (**expected first**, actual second) |
| `assertEquals(expected, actual, delta)` | doubles/floats — always use a delta |
| `assertTrue(condition)` / `assertFalse` | boolean checks |
| `assertNull(obj)` / `assertNotNull` | null checks |
| `assertThrows(Ex.class, () -> ...)` | returns the exception → assert on message/cause |
| `assertDoesNotThrow(() -> ...)` | no exception expected |
| `assertAll(() -> ..., () -> ...)` | run **all** assertions, report all failures at once |
| `assertTimeout(Duration, () -> ...)` | time limit |
| `assertSame` / `assertNotSame` | reference identity (==) |
| `fail("msg")` | force failure |
Messages can be lazy: `assertEquals(x, y, () -> "expensive " + compute())`.

## A5. AssertJ (fluent assertions — prefer over JUnit assertions)
```java
// Strings
assertThat("Hello World").startsWith("Hello").contains("lo W").hasSize(11);

// Numbers
assertThat(42).isBetween(40, 45).isPositive();
assertThat(new BigDecimal("10.00")).isEqualByComparingTo("10");            // ignores scale

// Collections
assertThat(list).hasSize(3).contains("a").containsExactly("a", "b", "c");
assertThat(employees).extracting(Employee::name).containsExactlyInAnyOrder("Asha", "Ravi");
assertThat(employees).filteredOn(e -> e.age() > 30).extracting(Employee::name).containsOnly("Asha");

// Exceptions
assertThatThrownBy(() -> service.place(null)).isInstanceOf(IllegalArgumentException.class).hasMessageContaining("null");

// Optional
assertThat(Optional.of("x")).isPresent().contains("x");
assertThat(Optional.empty()).isEmpty();

// Soft assertions — report ALL failures at once
SoftAssertions.assertSoftly(s -> {
    s.assertThat(result.name()).isEqualTo("Asha");
    s.assertThat(result.age()).isEqualTo(34);
});
```

## A6. Parameterized tests
Run the same test with different inputs. Needs `@ParameterizedTest` instead of `@Test`.

| Source | Provides |
|---|---|
| `@ValueSource(ints = {1,2,3})` | single-argument literals (ints, strings, longs, doubles…) |
| `@NullSource`, `@EmptySource`, `@NullAndEmptySource` | null / empty string/collection |
| `@CsvSource({"a,1", "b,2"})` | multiple args per row (comma-separated) |
| `@CsvFileSource(resources = "/data.csv")` | from a CSV file |
| `@MethodSource("factoryMethodName")` | static method returning `Stream<Arguments>` — most flexible |
| `@EnumSource(Status.class)` | enum constants (with `names` / `mode = EXCLUDE` to filter) |
```java
@ParameterizedTest(name = "{0} + {1} = {2}")
@CsvSource({"1,2,3", "10,-5,5", "0,0,0"})
void adds(int a, int b, int expected) { assertEquals(expected, a + b); }

@ParameterizedTest @MethodSource("invalidAmounts")
void rejectsInvalid(BigDecimal amount) { assertThrows(IllegalArgumentException.class, () -> svc.place("c1", amount)); }
static Stream<Arguments> invalidAmounts() { return Stream.of(Arguments.of(BigDecimal.ZERO), Arguments.of(new BigDecimal("-1")), Arguments.of((BigDecimal) null)); }
```

## A7. Other useful annotations
| Annotation | Purpose |
|---|---|
| `@Nested` | inner class to group tests by context (e.g., "when amount is valid" / "when invalid") |
| `@Disabled("reason")` | skip a test (always give a reason) |
| `@Tag("slow")` / `@Tag("integration")` | categorize; filter with `-Dgroups=slow` |
| `@Timeout(value = 2, unit = SECONDS)` | fail if exceeded |
| `@RepeatedTest(5)` | repeat (detect flakiness) |
| `@TempDir Path dir` | injects a temp directory, cleaned automatically |
| `@TestInstance(PER_CLASS)` | share one instance across tests (then `@BeforeAll`/`@AfterAll` can be non-static) |
| `@TestMethodOrder(OrderAnnotation.class)` + `@Order(1)` | explicit ordering (avoid — tests should be independent) |
| `@EnabledOnOs(MAC)` / `@EnabledOnJre` / `@EnabledIf` | conditional execution |

## A8. Extensions (replaces JUnit 4 runners/rules)
- Register with `@ExtendWith(XExtension.class)`.
- Key extensions: **`MockitoExtension`** (Mockito), **`SpringExtension`** (Spring — included in `@SpringBootTest`, `@WebMvcTest`, etc.).
- Extension points: `BeforeEachCallback`, `AfterEachCallback`, `ParameterResolver`, `TestExecutionExceptionHandler`, etc.

## A9. JUnit 4 → 5 migration
| JUnit 4 | JUnit 5 |
|---|---|
| `@Before` / `@After` | `@BeforeEach` / `@AfterEach` |
| `@BeforeClass` / `@AfterClass` | `@BeforeAll` / `@AfterAll` |
| `@Ignore` | `@Disabled("reason")` |
| `@RunWith(SpringRunner.class)` | `@ExtendWith(SpringExtension.class)` (or just use `@SpringBootTest`) |
| `@RunWith(MockitoJUnitRunner.class)` | `@ExtendWith(MockitoExtension.class)` |
| `@Test(expected = X.class)` | `assertThrows(X.class, () -> ...)` |
| `@Test(timeout = 1000)` | `@Timeout(1)` |
| `@Rule` / `@ClassRule` | extensions / `@TempDir` |
| `@Category` | `@Tag` |
| `public` required | package-private OK |

## A10. Deterministic tests
- Inject `Clock` instead of `LocalDateTime.now()` / `Instant.now()`.
- Inject `Supplier<UUID>` or `IdGenerator` instead of `UUID.randomUUID()`.
- No `Thread.sleep` → use **Awaitility** (`await().atMost(2, SECONDS).untilAsserted(() -> ...)`).
- No shared mutable static state between tests.

### Example
```java
class JUnitBasicsTest {
    private final List<String> items = new ArrayList<>();

    @BeforeEach void setUp() { items.clear(); }

    @Test @DisplayName("adds an element")
    void addsElement() {
        items.add("a");
        assertEquals(1, items.size());
        assertThat(items).containsExactly("a");
    }

    @Test void throwsOnInvalidIndex() {
        var ex = assertThrows(IndexOutOfBoundsException.class, () -> items.get(5));
        assertThat(ex.getMessage()).contains("5");
    }

    @ParameterizedTest @NullAndEmptySource @ValueSource(strings = {" ", "\t"})
    void blankInputs(String s) { assertTrue(s == null || s.isBlank()); }
}
```

---

# Part B — Mockito Concepts (Normal Classes & Methods)

## B1. Why mock
- Isolate the class under test from its collaborators (DB, HTTP client, message queue) → fast, deterministic tests.
- Mock **collaborators** (interfaces/classes your code calls). Never mock the class under test. Never mock simple value objects/DTOs.

## B2. Test doubles
| Type | What it does | Example |
|---|---|---|
| **Dummy** | placeholder, never actually used | passed to satisfy a constructor but no methods called |
| **Stub** | returns canned answers | `when(repo.findById("x")).thenReturn(Optional.of(order))` |
| **Mock** | records calls → you verify interactions | `verify(events).publish(...)` |
| **Spy** | wraps a real object + partial stubbing | `spy(new ArrayList<>())` — real methods run unless stubbed |
| **Fake** | working lightweight implementation | in-memory repository |

## B3. Setup — annotations
| Annotation | What |
|---|---|
| `@ExtendWith(MockitoExtension.class)` | enables Mockito annotations in JUnit 5 (required) |
| `@Mock` | creates a mock of the declared type |
| `@InjectMocks` | creates the real object, injects `@Mock`s via constructor → setter → field |
| `@Spy` | creates a spy wrapping a real instance |
| `@Captor` | declares an `ArgumentCaptor<T>` |

**`@InjectMocks` injection order:** tries biggest constructor first → then setter methods → then fields. If a constructor arg type has no matching `@Mock`, it receives `null` (silent → leads to NPE).

## B4. Default answers (unstubbed methods return)
| Return type | Default |
|---|---|
| Object | `null` |
| int, long, double… | `0` |
| boolean | `false` |
| Collection / Map | empty (not null) |
| Optional | `Optional.empty()` |
| Stream | `Stream.empty()` |

## B5. Stubbing — `when` / `thenReturn`
| Need | Syntax |
|---|---|
| return a value | `when(mock.method()).thenReturn(value)` |
| throw an exception | `when(mock.method()).thenThrow(new RuntimeException("msg"))` |
| compute from arguments | `when(mock.method(any())).thenAnswer(inv -> inv.getArgument(0))` |
| consecutive results | `when(mock.method()).thenReturn(a).thenReturn(b)` or `.thenReturn(a, b)` |
| **void method** — throw | `doThrow(new Ex()).when(mock).voidMethod()` |
| **void method** — do nothing | `doNothing().when(mock).voidMethod()` (default; useful for spy) |
| **void method** — answer | `doAnswer(inv -> { ... return null; }).when(mock).voidMethod()` |
| **spy** — stub without calling real | `doReturn(value).when(spy).method()` (NOT `when(spy.method())`) |
| BDD style | `given(mock.method()).willReturn(value)` |

**Why `doReturn` for spies:** `when(spy.method())` calls the **real method** during stubbing setup (which may fail or have side effects). `doReturn(v).when(spy).method()` skips the real call.

## B6. Argument matchers
| Matcher | Matches |
|---|---|
| `any()` | anything (incl. null in Mockito 1.x; excl. null in 2+) |
| `any(Order.class)` | any Order (not null) |
| `anyString()`, `anyInt()`, `anyList()` | type-specific (not null) |
| `eq(value)` | equals match |
| `isNull()` / `notNull()` | null checks |
| `argThat(predicate)` | custom predicate: `argThat(o -> o.amount().signum() > 0)` |
| `contains("x")`, `startsWith("y")` | string matchers |

**⚠ All-or-none rule:** if **any** argument uses a matcher, **all** must. Wrap raw values in `eq()`.
```java
// ❌ when(mock.method("literal", any()))       → InvalidUseOfMatchersException
// ✅ when(mock.method(eq("literal"), any()))
```

## B7. Verification
| Syntax | Meaning |
|---|---|
| `verify(mock).method()` | called exactly once (default) |
| `verify(mock, times(3)).method()` | called exactly 3 times |
| `verify(mock, never()).method()` | never called |
| `verify(mock, atLeastOnce()).method()` | ≥ 1 |
| `verify(mock, atLeast(2)).method()` | ≥ 2 |
| `verify(mock, atMost(5)).method()` | ≤ 5 |
| `verifyNoInteractions(mock1, mock2)` | no method called at all on these mocks |
| `verifyNoMoreInteractions(mock)` | no unverified calls remain |
| `inOrder(mock1, mock2)` → `inOrder.verify(mock1).a()` → `inOrder.verify(mock2).b()` | calls happened in this order |

**Verify behavior that matters** (event published, email sent, data saved). Don't verify every method call — it makes tests brittle.

## B8. ArgumentCaptor
Captures an argument your code builds internally and passes to a collaborator. Lets you assert on it.
```java
@Captor ArgumentCaptor<Order> orderCaptor;

@Test void capturesSavedOrder() {
    // ... trigger service.place(...)
    verify(repo).save(orderCaptor.capture());
    assertThat(orderCaptor.getValue().status()).isEqualTo(Order.Status.PAID);
    assertThat(orderCaptor.getValue().customerId()).isEqualTo("c1");
}
// getAllValues() → for multiple invocations
```
Use `capture()` inside `verify()`, not inside `when()` stubbing.

## B9. Strict stubs (default with `MockitoExtension`)
| Exception | Cause | Fix |
|---|---|---|
| `UnnecessaryStubbingException` | you stubbed a method the test never called | remove the stub or use `lenient().when(...)` |
| `PotentialStubbingProblem` | stubbed with args `A` but code called with `B` | fix the arg matcher or value |
Opt out per class: `@MockitoSettings(strictness = Strictness.LENIENT)`.

## B10. Spies
- A spy wraps a **real object**. Real methods run unless you override with `doReturn`.
- Use rarely — usually a sign the class has too many responsibilities.
```java
List<String> spy = spy(new ArrayList<>());
spy.add("a");                                    // real method runs
doReturn(100).when(spy).size();                  // override size()
assertEquals(100, spy.size());
assertEquals("a", spy.get(0));                   // still real
```

## B11. Common errors
| Error | Cause / fix |
|---|---|
| `NullPointerException` on SUT | `@InjectMocks` didn't inject (missing `@Mock` or no `MockitoExtension`) |
| `InvalidUseOfMatchersException` | mixed raw values and matchers → wrap literals in `eq()` |
| `UnnecessaryStubbingException` | stub not used → remove or `lenient()` |
| `PotentialStubbingProblem` | stubbed args ≠ called args |
| `WrongTypeOfReturnValue` | `thenReturn` type doesn't match return type; or `when(spy.f())` → use `doReturn` |
| `UnfinishedStubbingException` | nested `when()` inside `thenReturn()`; or forgot `.thenReturn()`  |
| `NotAMockException` | `verify()` called on a real object, not a mock |

## B12. Good habits
- One behavior per test.
- Mock collaborators, not the SUT or value objects.
- Assert **outcomes** (return value, saved state) first; verify interactions only for side effects that are the behavior itself.
- Don't verify every call (brittle).
- Don't `reset()` mocks between tests (create fresh mocks per test instead — `MockitoExtension` does this).

### Demo domain & full service test
```java
record Order(String id, String customerId, BigDecimal amount, Order.Status status) {
    enum Status { NEW, PAID, FAILED }
}
record PaymentResult(boolean success, String txId) {}
class PaymentException extends RuntimeException { PaymentException(String m) { super(m); } }

interface OrderRepository { Order save(Order o); Optional<Order> findById(String id); }
interface PaymentClient    { PaymentResult charge(String customerId, BigDecimal amount); }
interface EventPublisher   { void publish(String topic, Object event); }
class IdGenerator          { static String next() { return UUID.randomUUID().toString(); } }

class OrderService {
    private final OrderRepository repo; private final PaymentClient payment; private final EventPublisher events;
    OrderService(OrderRepository repo, PaymentClient payment, EventPublisher events) {
        this.repo = repo; this.payment = payment; this.events = events;
    }
    Order place(String customerId, BigDecimal amount) {
        if (amount == null || amount.signum() <= 0) throw new IllegalArgumentException("amount must be positive");
        PaymentResult r = payment.charge(customerId, amount);
        Order saved = repo.save(new Order(IdGenerator.next(), customerId, amount,
                                          r.success() ? Order.Status.PAID : Order.Status.FAILED));
        if (r.success()) events.publish("orders.paid", saved);
        return saved;
    }
}
```
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock OrderRepository repo;
    @Mock PaymentClient payment;
    @Mock EventPublisher events;
    @InjectMocks OrderService service;
    @Captor ArgumentCaptor<Order> orderCaptor;

    @Test void paymentSucceeds_savesPaidOrder_andPublishesEvent() {
        given(payment.charge("c1", BigDecimal.TEN)).willReturn(new PaymentResult(true, "tx1"));
        given(repo.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));

        Order result = service.place("c1", BigDecimal.TEN);

        verify(repo).save(orderCaptor.capture());
        assertThat(orderCaptor.getValue().status()).isEqualTo(Order.Status.PAID);
        verify(events).publish(eq("orders.paid"), same(result));
    }

    @Test void paymentFails_doesNotPublish() {
        given(payment.charge(any(), any())).willReturn(new PaymentResult(false, null));
        given(repo.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));
        assertEquals(Order.Status.FAILED, service.place("c1", BigDecimal.ONE).status());
        verify(events, never()).publish(any(), any());
    }

    @Test void invalidAmount_touchesNothing() {
        assertThrows(IllegalArgumentException.class, () -> service.place("c1", BigDecimal.ZERO));
        verifyNoInteractions(payment, repo, events);
    }

    @Test void downstreamException_propagates() {
        given(payment.charge(any(), any())).willThrow(new PaymentException("timeout"));
        assertThrows(PaymentException.class, () -> service.place("c1", BigDecimal.ONE));
        verifyNoInteractions(repo, events);
    }

    @Test void consecutiveStubbing() {
        given(payment.charge(any(), any())).willThrow(new PaymentException("timeout"))
                                           .willReturn(new PaymentResult(true, "tx2"));
        given(repo.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));
        assertThrows(PaymentException.class, () -> service.place("c1", BigDecimal.ONE));
        assertEquals(Order.Status.PAID, service.place("c1", BigDecimal.ONE).status());
    }

    @Test void voidMethodStubbing() {
        given(payment.charge(any(), any())).willReturn(new PaymentResult(true, "tx"));
        given(repo.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));
        doThrow(new IllegalStateException("broker down")).when(events).publish(anyString(), any());
        assertThrows(IllegalStateException.class, () -> service.place("c1", BigDecimal.ONE));
    }
}
```

---

# Part C — Mocking Static & Private

## C1. Static methods — `mockStatic`
- Mockito **5**: works out of the box (inline mock maker is the default). Mockito 3.4–4: needs `mockito-inline`.
- Always use **try-with-resources** — the mock is **thread-local**, active only inside the block.
- After the block closes, the real behavior is restored.
```java
class StaticMockTest {
    @Test void mockStatic_scopedToBlock() {
        try (MockedStatic<IdGenerator> ids = mockStatic(IdGenerator.class)) {
            ids.when(IdGenerator::next).thenReturn("fixed-id");
            assertEquals("fixed-id", IdGenerator.next());
            ids.verify(IdGenerator::next);                                  // verify the static call
        }
        assertNotEquals("fixed-id", IdGenerator.next());                    // real behavior restored
    }
}
```
**`mockConstruction`** — intercept `new` calls:
```java
@Test void interceptsConstructor() {
    try (MockedConstruction<Mailer> mc = mockConstruction(Mailer.class,
            (mock, ctx) -> when(mock.send(any())).thenReturn("mocked"))) {
        assertEquals("mocked", new Mailer().send("a@b.com"));
        assertEquals(1, mc.constructed().size());
    }
}
```

**Better design (prefer this):** wrap the static behind something injectable.
| Instead of | Inject |
|---|---|
| `LocalDate.now()` / `Instant.now()` | `Clock` → `LocalDate.now(clock)` |
| `UUID.randomUUID()` | `Supplier<UUID>` or `IdGenerator` interface |
| `Files.readString(path)` | `FileReader` wrapper or inject the content |

Then test with a stub/mock — no `mockStatic` needed.

**Caveats:** don't `mockStatic` JDK internals (`Thread`, `System`) — flaky. `MockedStatic` is thread-local, so other threads don't see it.

## C2. Private methods — you don't mock them
Mockito **cannot stub or verify private methods** — by design. Private methods are implementation details.

| # | Approach | When to use |
|---|---|---|
| 1 | **Test through the public method** that calls it | ✅ default — assert the observable result |
| 2 | **Extract into a collaborator** (new class/interface) and mock that | private logic is complex, slow, or has side effects |
| 3 | Widen to **package-private** (`@VisibleForTesting`) + **spy** + `doReturn` | legacy code you can't restructure |
| 4 | **Reflection** (`Method.setAccessible(true)`, `ReflectionTestUtils.invokeMethod`) | last resort — *invoke* (not stub) a private method |
| 5 | **PowerMock** | legacy only — avoid on Java 17+ |

```java
// Option 1 — test via the public API (preferred)
class DiscountService {
    BigDecimal applyDiscount(BigDecimal price, String code) { return price.subtract(discountFor(code)); }
    private BigDecimal discountFor(String code) { return "SAVE10".equals(code) ? BigDecimal.TEN : BigDecimal.ZERO; }
}
class DiscountServiceTest {
    DiscountService svc = new DiscountService();
    @Test void saveCodeGivesTenOff()     { assertEquals(new BigDecimal("90"), svc.applyDiscount(new BigDecimal("100"), "SAVE10")); }
    @Test void unknownCodeNoDiscount()   { assertEquals(new BigDecimal("100"), svc.applyDiscount(new BigDecimal("100"), "X")); }
}

// Option 2 — extract + mock
interface DiscountPolicy { BigDecimal discountFor(String code); }
class PricingService {
    private final DiscountPolicy policy;
    PricingService(DiscountPolicy policy) { this.policy = policy; }
    BigDecimal price(BigDecimal base, String code) { return base.subtract(policy.discountFor(code)); }
}
@ExtendWith(MockitoExtension.class)
class PricingServiceTest {
    @Mock DiscountPolicy policy;
    @Test void usesPolicy() {
        given(policy.discountFor("VIP")).willReturn(new BigDecimal("25"));
        assertEquals(new BigDecimal("75"), new PricingService(policy).price(new BigDecimal("100"), "VIP"));
    }
}

// Option 3 — package-private + spy (legacy)
class LegacyPricing {
    BigDecimal total(BigDecimal base) { return base.add(tax(base)); }
    BigDecimal tax(BigDecimal base) { return base.multiply(new BigDecimal("0.18")); } // was private; widened
}
class LegacyPricingTest {
    @Test void spyOverridesTax() {
        LegacyPricing spy = spy(new LegacyPricing());
        doReturn(new BigDecimal("5")).when(spy).tax(any());
        assertEquals(new BigDecimal("105"), spy.total(new BigDecimal("100")));
    }
}

// Option 4 — reflection (last resort, for invoking not stubbing)
@Test void reflection_lastResort() throws Exception {
    Method m = DiscountService.class.getDeclaredMethod("discountFor", String.class);
    m.setAccessible(true);
    assertEquals(BigDecimal.TEN, m.invoke(new DiscountService(), "SAVE10"));
}
```

**Private fields:** prefer **constructor injection**. For legacy `@Value`/field-injected beans: `ReflectionTestUtils.setField(obj, "fieldName", value)`.

**Also remember:** `final` classes/methods are mockable in Mockito 5. `equals`/`hashCode` aren't reliably stubbable.

---

# Part D — Spring Boot Testing Concepts

## D1. The key idea — how much ApplicationContext to load
Spring's TestContext framework builds and **caches** an `ApplicationContext`. **Load the smallest context that proves what you need.**

| What to test | Annotation | What's in the context | Speed |
|---|---|---|---|
| Service / domain logic | **none** — plain JUnit + Mockito | no Spring context | ⚡⚡⚡ |
| Controller (mapping, validation, JSON, errors, security) | **`@WebMvcTest(XController.class)`** | web slice only | ⚡⚡ |
| Repository / JPA (queries, mappings) | **`@DataJpaTest`** | JPA slice | ⚡⚡ |
| JSON serialization | `@JsonTest` | Jackson only | ⚡⚡ |
| REST client | `@RestClientTest` | client + mock server | ⚡⚡ |
| Full wiring / transactions / AOP | **`@SpringBootTest`** | **full** context | ⚡ |

**Rule:** many plain unit tests → some slices → *few* `@SpringBootTest`.

## D2. `@Mock` vs `@MockitoBean`
| Annotation | What | Spring context? |
|---|---|---|
| `@Mock` | plain Mockito mock — **not** a Spring bean | no |
| `@MockitoBean` | replaces a bean **in the Spring context** with a Mockito mock | yes |
| `@MockitoSpyBean` | wraps the real bean with a spy in the context | yes |

Use `@Mock` for plain unit tests. Use `@MockitoBean` inside `@WebMvcTest` / `@SpringBootTest` where Spring needs to wire the mock as a bean.

## D3. Controller testing — `@WebMvcTest` + `MockMvc`

**What's loaded:** `@RestController`, `@ControllerAdvice`, Jackson, validation, filters, `MockMvc`.
**Not loaded:** `@Service`, `@Repository`, `@Component` → must be mocked with `@MockitoBean`.

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {
    @Autowired MockMvc mvc;
    @Autowired ObjectMapper mapper;
    @MockitoBean OrderService service;

    @Test void getOrder_returns200() throws Exception {
        given(service.get("o1")).willReturn(new Order("o1", "c1", BigDecimal.TEN, Order.Status.PAID));
        mvc.perform(get("/orders/{id}", "o1").accept(APPLICATION_JSON))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.id").value("o1"))
           .andExpect(jsonPath("$.status").value("PAID"));
    }

    @Test void createOrder_invalidBody_returns400() throws Exception {
        mvc.perform(post("/orders").contentType(APPLICATION_JSON).content("{\"amount\": -5}"))
           .andExpect(status().isBadRequest());
        verifyNoInteractions(service);                           // validation stopped before service
    }

    @Test void getOrder_notFound_returns404() throws Exception {
        given(service.get("x")).willThrow(new NoSuchElementException("order x"));
        mvc.perform(get("/orders/x")).andExpect(status().isNotFound());
    }
}
```

**What to test here:** status codes (200/201/400/404/409/500), response JSON, request validation, error handling (`@ControllerAdvice`), headers, security (401/403).

**Security on classpath?** Use `@WithMockUser(roles="USER")`, `.with(csrf())`, `.with(jwt())` (needs `spring-security-test`).

**`MockMvc` essentials:**
- Request: `get/post/put/patch/delete`, `.contentType()`, `.content()`, `.param()`, `.header()`, `.with(csrf())`, `.with(user("u"))`
- Assert: `status().isOk()`, `jsonPath("$.field").value(v)`, `content().json(...)`, `header().string(...)`, `.andDo(print())` for debugging

## D4. Service testing — unit test vs integration test

**Default: plain JUnit + Mockito** (Part B). No Spring context = fastest.

Use **`@SpringBootTest`** only when the test depends on Spring behavior:
- `@Transactional` rollback/propagation
- AOP proxies (`@Cacheable`, `@Async`, `@Retryable`)
- Real wiring between service + repository
- Bean validation on method arguments

```java
@SpringBootTest
class OrderServiceIT {
    @Autowired OrderService service;
    @Autowired OrderJpaRepository repo;
    @MockitoBean PaymentClient payment;
    @MockitoBean EventPublisher events;

    @Test void publishFailsAfterSave_transactionRollsBack() {
        given(payment.charge(any(), any())).willReturn(new PaymentResult(true, "tx1"));
        doThrow(new IllegalStateException("broker down")).when(events).publish(any(), any());

        assertThrows(IllegalStateException.class, () -> service.place("c1", BigDecimal.TEN));
        assertThat(repo.findByCustomerId("c1")).isEmpty();       // rollback proven
    }
}
```

**`@SpringBootTest` options:**
- `webEnvironment = MOCK` (default, no server) — use with `@AutoConfigureMockMvc`
- `webEnvironment = RANDOM_PORT` — real server → `TestRestTemplate`/`WebTestClient` (⚠ server runs on another thread → test-method `@Transactional` won't rollback server-side work)
- `classes = {...}` — limit config
- `properties = {"key=value"}` — override properties

## D5. JPA / Repository testing — `@DataJpaTest`

**What's loaded:** entities, Spring Data repositories, `DataSource`, `TestEntityManager`, Flyway/Liquibase.
Each test runs in a **transaction that is rolled back**.
By default, **replaces your DB with embedded H2**.

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)                  // use real DB, not H2
@Testcontainers
class OrderRepositoryTest {
    @Container @ServiceConnection
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired OrderJpaRepository repo;
    @Autowired TestEntityManager em;

    @Test void findByCustomer_returnsOnlyThatCustomersOrders() {
        em.persist(new OrderEntity("c1", BigDecimal.TEN));
        em.persist(new OrderEntity("c2", BigDecimal.ONE));
        em.flush(); em.clear();                                     // force SQL + clear 1st-level cache

        assertThat(repo.findByCustomerId("c1")).hasSize(1);
    }
}
```

**What to test:** custom `@Query`, derived queries, constraints (unique/not-null), relationships/cascade, lazy loading & N+1 (via Hibernate `Statistics` query count), pagination, `@Version` conflicts.

⚠ Rollback can hide flush/constraint errors → always call `em.flush()`.
Use `@Sql("/seed.sql")` for data fixtures.

**H2 vs Testcontainers:** H2 differs from Postgres (JSONB, window functions, locking, dialect) → use **Testcontainers** when SQL is DB-specific. `@ServiceConnection` auto-configures the DataSource. Docker must be available.

## D6. Replacing / adding beans in the context
| Need | Use |
|---|---|
| Replace a bean with a mock | `@MockitoBean` |
| Wrap a real bean with a spy | `@MockitoSpyBean` |
| Add/override beans (fixed `Clock`, fake) | `@TestConfiguration` + `@Bean @Primary` |
| Test-only properties | `@ActiveProfiles("test")` + `application-test.yml` |
| Runtime values (container port) | `@DynamicPropertySource` or `@ServiceConnection` |

## D7. Context caching — why suites get slow
Spring **caches** an `ApplicationContext` per unique configuration. Same config → reused. Different → **new context load** (seconds each).

**Things that create a new context:**
- Different set of `@MockitoBean`
- Different `@TestPropertySource` / `properties=`
- Different `@ActiveProfiles` / `@Import`
- `@DirtiesContext`

**Fixes:** use slices + plain Mockito, keep few `@SpringBootTest` configs, share mocks in one abstract base class, share Testcontainers, avoid `@DirtiesContext`.

## D8. Common Spring test errors
| Symptom | Cause / fix |
|---|---|
| `NoSuchBeanDefinitionException` in `@WebMvcTest` | slice doesn't load `@Service` → `@MockitoBean` it |
| `Unable to find a @SpringBootConfiguration` | test package not under main app package → fix package or `classes=` |
| 401/403 instead of 200 | security active → `@WithMockUser`, `csrf()`, `@Import(SecurityConfig.class)` |
| `LazyInitializationException` | no session → `@Transactional` on test or fetch join |
| Passes on H2, fails on Postgres | dialect differences → Testcontainers |
| `@Autowired` is null | not a Spring test (missing annotation) |
| Suite is slow (minutes) | too many context loads → check cache stats, fewer unique `@MockitoBean` configs |

---

## Practice Tasks
- [ ] **A:** 5 JUnit tests incl. `assertThrows` + one `@ParameterizedTest`
- [ ] **B:** `OrderService` unit tests — success, failure, invalid input, downstream exception, `ArgumentCaptor`, `verify(never())`
- [ ] **C:** mock a static call → refactor to injection and delete `mockStatic`; test a private method through the public API
- [ ] **D:** `@WebMvcTest` (200/400/404) · `@DataJpaTest` (custom query) · one `@SpringBootTest` rollback test
- [ ] *(2-month)* Testcontainers DB test; Kafka consumer test with Awaitility

## Question Bank
1. 🟢 Unit vs integration test; the test pyramid?
2. 🟢 `@BeforeEach` vs `@BeforeAll`?
3. 🟢 Mock vs Spy vs Stub vs Fake?
4. 🟢 `@Mock` vs `@InjectMocks` vs `@MockitoBean`?
5. 🟡 How do you test a method that throws / returns void / has side effects?
6. 🟡 `ArgumentCaptor` — when? Matcher "all or none" rule?
7. 🟡 What are Mockito strict stubs? `UnnecessaryStubbingException`?
8. 🟡 `doReturn().when(spy)` vs `when(spy.f())` — why?
9. 🟡 **How do you mock a static method?** What's the better design?
10. 🟡 **How do you test a private method?** (don't mock it — 4 alternatives)
11. 🟡 `@WebMvcTest` vs `@SpringBootTest` — what's loaded? Why is `@Service` missing in `@WebMvcTest`?
12. 🟡 When do you need `@SpringBootTest` for a service test vs plain Mockito?
13. 🟡 `@DataJpaTest` defaults — when to switch to Testcontainers?
14. 🟡 How do you test validation and error handling in a controller?
15. 🟡 How do you test code using `LocalDateTime.now()` / `UUID.randomUUID()`?
16. 🔴 What is the Spring context cache key? What causes extra context loads?
17. 🔴 `MockMvc` vs `TestRestTemplate` — threading and `@Transactional` implications?
18. 🔴 How do you test `@Transactional` rollback or N+1 regression?
19. 🔴 How do you test a Kafka consumer (retry/DLT/idempotency)?
20. 🔴 Test suite takes 25 min — how to speed it up?
21. 🔴 100% coverage — good or bad? What else measures test quality?

## Self-Check
- [ ] Write a Mockito service test (success + failure + `verify never`) in <10 min
- [ ] Show `mockStatic` with try-with-resources; explain the refactor that removes the need
- [ ] Explain why Mockito can't mock private methods and list 3 alternatives
- [ ] Pick the right annotation for controller / service / repository and state what each loads
- [ ] Name 3 things that break the Spring context cache
