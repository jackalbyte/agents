# Java Architecture Review — Spring Boot 3 / Hexagonal

Review changed code for violations of the hexagonal (clean) architecture and Spring-specific structural misuse. The project uses adapters → ports → domain layering rooted at `ru.chinatoday.taobao`.

---

## Checklist

### 1. Hexagonal Layer Violations

**Domain layer must not import:**
- Spring annotations (`@Service`, `@Component`, `@Repository`, `@Autowired`, `@Transactional`)
- JPA/Hibernate types (`@Entity`, `@Column`, `EntityManager`, `JpaRepository`)
- Adapter-layer classes (controllers, REST DTOs, persistence entities)
- Any `jakarta.*` persistence types

**Why it matters:** Domain logic coupled to infrastructure becomes untestable in isolation and breaks when infrastructure changes.

**What to look for:** Imports in `domain/` packages that reference `org.springframework`, `jakarta.persistence`, or adapter package names.

```java
// bad — domain service importing Spring
package ru.chinatoday.taobao.domain.service;
import org.springframework.stereotype.Service;     // ❌
import jakarta.persistence.EntityManager;          // ❌

// good — domain service is pure Java
package ru.chinatoday.taobao.domain.service;
import ru.chinatoday.taobao.domain.port.OrderRepository; // ✅ port interface only
```

---

### 2. Missing Port Interface (Direct Concrete Dependency)

Look for domain services or use cases that depend directly on adapter implementations instead of port interfaces.

**Why it matters:** Bypasses the hexagonal boundary — the domain is now tightly coupled to one infrastructure implementation.

```java
// bad — domain depends on JPA adapter directly
private final OrderJpaRepository orderJpaRepository;  // ❌ concrete adapter

// good — domain depends on port
private final OrderRepository orderRepository;         // ✅ port interface
```

---

### 3. Wrong Spring Stereotype for the Layer

| Layer | Correct stereotype |
|---|---|
| Domain use case / service | `@Service` (or no annotation if wired manually) |
| Persistence adapter | `@Repository` |
| REST/messaging adapter | `@Component` or `@RestController` |
| Port interface | No annotation — it's a plain Java interface |

**Why it matters:** `@Repository` triggers Spring's persistence exception translation — using it outside a persistence adapter gives confusing behavior. `@Component` on a domain service signals the developer doesn't understand the layer.

---

### 4. `@Transactional` Placement

- `@Transactional` belongs on **application/use-case services**, not on domain entities or port interfaces.
- It must be on **public** methods — Spring's AOP proxy cannot intercept package-private or private calls.
- Self-invocation within the same bean bypasses the proxy and silently drops the transaction.

```java
// bad — @Transactional on a port interface method (has no effect)
public interface OrderRepository {
    @Transactional  // ❌ ignored on interface
    void save(Order order);
}

// bad — self-invocation bypasses proxy
@Service
public class OrderService {
    public void process() { this.save(); } // ❌ save() runs without transaction
    @Transactional
    public void save() { ... }
}
```

---

### 5. Domain Entity vs. Persistence Entity Confusion

Look for domain classes (in `domain/`) that carry `@Entity`, `@Table`, `@Id` annotations, or that extend `AbstractPersistable`.

**Why it matters:** JPA entities are infrastructure artifacts. Domain entities are pure business objects. Mixing them creates bidirectional coupling.

**Fix:** Keep two separate classes — a JPA entity in the persistence adapter, a domain model in the domain layer — and map between them in the adapter.

---

### 6. Fat Constructor / Missing Factory Method for Domain Objects

Look for domain objects built entirely via setters or with public constructors that allow invalid state.

**Why it matters:** Domain objects should enforce their own invariants. If any caller can set fields freely, invariants aren't guaranteed.

```java
// bad — nothing prevents an incomplete Order
Order order = new Order();
order.setId(id);
// caller forgets order.setCustomer(...)

// good — factory method enforces required fields
Order order = Order.create(id, customer, items); // throws if invalid
```

---

### 7. Record vs. Class Choice

- Use `record` for **value objects** (DTOs, query results, simple domain values) — immutable, equality by value.
- Use `class` for **entities with identity** — equality by ID, mutable lifecycle.
- Do not use `record` for JPA entities — JPA requires a no-arg constructor and mutable fields.

---

### 8. SOLID Violations

**Single Responsibility:** Flag classes with more than one reason to change — e.g., a service that both validates input, executes business logic, and formats the response.

**Dependency Inversion:** Flag `new ConcreteClass()` inside services for dependencies that should be injected. Constructor injection is preferred over field injection (`@Autowired` on a field).

```java
// bad — field injection hides dependencies, makes testing harder
@Autowired
private OrderRepository repository;  // ⚠️

// good — constructor injection, dependencies explicit
private final OrderRepository repository;
public OrderService(OrderRepository repository) {
    this.repository = repository;
}
```
