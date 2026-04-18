# Java Testing Review — JUnit 5 / Mockito / Testcontainers

Review test code for quality issues that allow tests to pass while providing false confidence. Focus on test placement, assertion quality, naming, and coverage gaps.

---

## Checklist

### 1. Test Placement

| Test type | Correct location | Framework |
|---|---|---|
| Unit tests (business logic) | `src/test/java` | JUnit 5 + Mockito |
| Integration tests (DB, HTTP) | `src/integrationTest/java` | Testcontainers + Spring Boot Test |

**Flag:**
- Business logic tests in `integrationTest` — they load full Spring context unnecessarily, slowing CI.
- Database interaction tests in `src/test` mocking the repository — these pass but don't verify real query behavior.

```java
// ❌ unit test mocking the DB — doesn't verify query correctness
@Test
void shouldFindActiveOrders() {
    when(repository.findByActive(true)).thenReturn(List.of(order));
    // this test only verifies wiring, not the actual query
}

// ✅ integration test with real PostgreSQL (Testcontainers)
@Test
void shouldFindActiveOrders() {
    repository.save(activeOrder);
    repository.save(deactivatedOrder);
    List<Order> result = repository.findByActive(true);
    assertThat(result).containsExactly(activeOrder);
}
```

---

### 2. Tests That Only Verify Calls (Not Behavior)

Tests that only assert `verify(mock).method(...)` without checking what was returned or what state changed provide zero behavioral coverage.

```java
// ❌ verifies the method was called — not that it did the right thing
@Test
void shouldSaveOrder() {
    service.create(request);
    verify(repository).save(any());  // passes even if save() stored wrong data
}

// ✅ assert the actual outcome
@Test
void shouldSaveOrderWithCorrectFields() {
    OrderResponse result = service.create(request);
    assertThat(result.status()).isEqualTo(OrderStatus.PENDING);
    assertThat(result.customerId()).isEqualTo(request.customerId());
}
```

---

### 3. Test Method Naming

Names must clearly describe: what scenario is being tested, and what the expected outcome is. Use the pattern `should[ExpectedBehavior]When[Condition]` or `[method]_[scenario]_[expectedResult]`.

```java
// ❌ uninformative
@Test void test1() { ... }
@Test void createOrder() { ... }

// ✅ self-documenting
@Test void shouldRejectOrderWhenCustomerIsDeactivated() { ... }
@Test void shouldReturnEmptyListWhenNoActiveOrdersExist() { ... }
```

---

### 4. Missing Test Cases

For any changed method, verify tests cover:

- **Happy path** — the intended normal flow
- **Empty/null input** — empty collections, null parameters where applicable
- **Boundary values** — minimum/maximum amounts, edge-of-page pagination
- **Error conditions** — entity not found, validation failure, duplicate key
- **Soft delete filter** — queries must exclude deactivated records; test with both active and deactivated data present

Flag if only the happy path is tested for code that has explicit error branches.

---

### 5. Brittle Tests

**Hardcoded IDs:**
```java
// ❌ fails if test data changes order
assertThat(result.id()).isEqualTo("550e8400-e29b-41d4-a716-446655440000");

// ✅ verify ID is non-null and valid UUID format, not a specific value
assertThat(result.id()).isNotNull();
```

**Order-dependent tests:** Each test must be independent. `@TestMethodOrder` should not be required for correctness. Tests that rely on state left by a previous test are flaky by design.

**Execution time assertions:** Avoid `Thread.sleep()` or asserting that something completes within N milliseconds — timing-based tests are inherently flaky in CI.

---

### 6. `@MockBean` vs `@Mock`

- `@Mock` (Mockito) — for plain unit tests, no Spring context loaded.
- `@MockBean` (Spring Boot Test) — replaces a bean in the Spring context; only use in `@SpringBootTest` or slice tests. Using `@MockBean` in a unit test unnecessarily loads the application context.

```java
// ❌ @MockBean in a unit test — loads Spring context for no reason
@SpringBootTest
class OrderServiceTest {
    @MockBean OrderRepository repository;
    ...
}

// ✅ pure unit test
class OrderServiceTest {
    @Mock OrderRepository repository;
    @InjectMocks OrderService service;
    ...
}
```

---

### 7. Testcontainers Setup

Integration tests using PostgreSQL must use Testcontainers, not an in-memory H2 database. H2 has different SQL dialect and type handling — tests that pass on H2 can fail against real PostgreSQL.

Verify:
- `@Testcontainers` + `@Container PostgreSQLContainer` or a shared static container is present.
- The datasource URL is wired to the container, not to a hardcoded localhost address.
- Tests clean up their data or use `@Transactional` (rolled back after each test) to avoid state leakage between tests.

---

### 8. Javadoc on Test Classes (Russian)

Per project convention, public test classes must have Javadoc in Russian describing what feature or component is being tested.

```java
// ❌ missing Javadoc
public class OrderServiceTest { ... }

// ✅
/**
 * Тесты для {@link OrderService}: создание, отмена и поиск заказов.
 */
public class OrderServiceTest { ... }
```
