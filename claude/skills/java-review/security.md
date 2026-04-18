# Java Security Review — Spring Boot 3 / PostgreSQL

Review changed code for security vulnerabilities specific to the Spring Boot + JPA + PostgreSQL stack. Focus on injection, data exposure, authentication boundaries, and input validation.

---

## Checklist

### 1. SQL / JPQL Injection

**Native queries with string concatenation are the highest risk:**

```java
// ❌ SQL injection — user input concatenated into query string
@Query(value = "SELECT * FROM taobao.orders WHERE customer_name = '" + name + "'",
       nativeQuery = true)

// ✅ always use named parameters
@Query(value = "SELECT * FROM taobao.orders WHERE customer_name = :name",
       nativeQuery = true)
List<OrderEntity> findByName(@Param("name") String name);
```

**JPQL is safer but not immune:**
```java
// ❌ JPQL with concatenation — still injectable
entityManager.createQuery("FROM Order WHERE status = '" + status + "'");

// ✅ parameterized
entityManager.createQuery("FROM Order WHERE status = :status")
    .setParameter("status", status);
```

**Flag any use of `EntityManager.createNativeQuery()` with dynamic string construction.**

---

### 2. Sensitive Data in Logs or API Responses

Look for fields being logged or serialized that should not be:

- Passwords, tokens, API keys — must never appear in logs or JSON responses
- Personal data fields (phone, email, passport numbers) — check if logging is appropriate
- Internal IDs or stack traces exposed in REST error responses

```java
// ❌ logs sensitive field
log.info("Processing order for user: {}", user.toString()); // if toString() includes password

// ❌ exception detail exposed to caller
return ResponseEntity.badRequest().body(e.getMessage()); // may reveal schema/logic

// ✅ sanitized error response
return ResponseEntity.badRequest().body(new ErrorResponse("Invalid request"));
```

Ensure `@JsonIgnore` or response DTOs exclude sensitive fields. Domain entities should not be returned directly from controllers.

---

### 3. Missing Input Validation

New `@RestController` endpoints with `@RequestBody` parameters must have `jakarta.validation` constraints, and the controller method must carry `@Valid` or `@Validated`.

```java
// ❌ no validation — accepts any JSON payload
@PostMapping("/orders")
public ResponseEntity<OrderResponse> create(@RequestBody CreateOrderRequest request) { ... }

// ✅
@PostMapping("/orders")
public ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest request) { ... }

// And the request class:
public record CreateOrderRequest(
    @NotBlank String customerId,
    @NotNull @Positive BigDecimal amount,
    @NotEmpty List<@Valid OrderLineRequest> lines
) {}
```

Also check: path variables and query parameters annotated with `@NotBlank`, `@Min`/`@Max` where appropriate.

---

### 4. Mass Assignment via `@RequestBody`

Verify that request body DTOs are narrow — they must not include fields that the client should not be allowed to set (e.g., `id`, `createdAt`, `createdBy`, `active`).

```java
// ❌ entity used directly as request body — client can set id, active, createdBy
@PostMapping
public void create(@RequestBody OrderEntity entity) { ... }

// ✅ dedicated request record with only writable fields
public record CreateOrderRequest(String customerId, BigDecimal amount) {}
```

---

### 5. `no-security` Profile Bypass

The project has a `no-security` Spring profile that bypasses authentication. Verify:
- No new endpoint is protected only by the assumption that `no-security` is off in production.
- Security configuration does not have `permitAll()` accidentally broadened.
- Any new `SecurityFilterChain` bean restricts access correctly and doesn't inadvertently widen the default-deny posture.

---

### 6. Hardcoded Credentials or Secrets

Flag any literal strings that look like passwords, tokens, or connection strings in source code:

```java
// ❌
String apiKey = "sk-prod-abcdef123456";
DataSource ds = DriverManager.getConnection("jdbc:postgresql://...", "admin", "password123");
```

These must come from environment variables or Spring configuration properties (never from `application.properties` committed to source control with real values).

---

### 7. N+1 Query Problems

Look for patterns that trigger one query per entity when the data could be fetched in bulk:

```java
// ❌ N+1 — loads each order's items in a separate query
for (Order order : orders) {
    List<Item> items = order.getItems(); // triggers SELECT per order if lazy
    process(items);
}

// ✅ fetch join in the query
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.active = true")
List<Order> findAllWithItems();
```

Also flag: calling a repository method inside a loop, or using `FetchType.EAGER` broadly when not needed.

---

### 8. Audit Field Integrity

All writes must populate audit fields properly. Check that services creating or updating entities set:

| Operation | Fields to set |
|---|---|
| Create | `createdAt`, `createdBy`, `updatedAt`, `updatedBy`, `active = true` |
| Update | `updatedAt`, `updatedBy` |
| Soft delete | `active = false`, `deactivatedAt`, `deactivatedBy` |

These must not be accepted from the client — they must be set server-side from the security context (`SecurityContextHolder`) and `Instant.now()`.
