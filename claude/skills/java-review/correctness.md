# Java Correctness Review

Review changed code for Java-specific correctness pitfalls that produce wrong behavior at runtime. These are language-level traps that often pass unit tests because the test doesn't exercise the failing edge case.

---

## Checklist

### 1. `Optional` Misuse

**Anti-patterns to flag:**

```java
// ❌ get() without isPresent() — throws NoSuchElementException
return repository.findById(id).get();

// ❌ Optional as a field or constructor parameter — not designed for this
private Optional<String> name;  // use null or a required value instead

// ❌ isPresent() + get() instead of idiomatic API
if (opt.isPresent()) { return opt.get(); } else { return default; }
// fix:
return opt.orElse(default);

// ❌ Optional.of() with a value that can be null — throws NPE
Optional.of(possiblyNull);  // use Optional.ofNullable() instead
```

**Correct patterns:**
```java
repository.findById(id).orElseThrow(() -> new EntityNotFoundException(id));
repository.findById(id).ifPresent(this::process);
repository.findById(id).map(Entity::getName).orElse("unknown");
```

---

### 2. Stream Pipeline Missing Terminal Operation

A stream without a terminal operation (`collect`, `forEach`, `count`, `findFirst`, etc.) does nothing — it's lazy and never executes.

```java
// ❌ filter has no effect — no terminal operation
orders.stream().filter(o -> o.isActive());  // silently does nothing

// ✅
List<Order> active = orders.stream().filter(o -> o.isActive()).toList();
```

---

### 3. `equals()` / `hashCode()` in JPA Entities

JPA entities must not use auto-generated `equals`/`hashCode` based on all fields — Hibernate proxies make this unreliable.

**Rules:**
- For entities with a natural business key: implement `equals`/`hashCode` on that key only.
- For entities with a surrogate UUID: implement `equals`/`hashCode` on the ID, but handle the case where ID is null (before `persist`).
- Do **not** use Lombok `@Data` on JPA entities — generates `hashCode` from all fields including mutable ones and calls `equals` on lazy-loaded collections.
- Records cannot be JPA entities (no no-arg constructor, immutable fields).

```java
// ❌ Lombok @Data on entity — dangerous
@Data
@Entity
public class Order { ... }

// ✅ business-key equality
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Order other)) return false;
    return id != null && id.equals(other.getId());
}
@Override
public int hashCode() { return getClass().hashCode(); }
```

---

### 4. Null Safety

**Look for:**
- Calling methods on values returned from `Map.get()` without null check (returns null if key absent)
- Dereferencing method return values annotated with `@Nullable` without guard
- `String.equals()` called on a potentially null variable — use `"literal".equals(variable)` or `Objects.equals()`

```java
// ❌ NPE if key absent
String value = map.get(key).trim();

// ✅
String raw = map.get(key);
if (raw == null) throw new IllegalArgumentException("Missing key: " + key);
String value = raw.trim();

// or
String value = map.getOrDefault(key, "").trim();
```

---

### 5. Date/Time Handling

The project likely uses `java.time`. Common mistakes:

| Situation | Wrong type | Right type |
|---|---|---|
| Database timestamp (with TZ) | `Date`, `Calendar` | `Instant` or `OffsetDateTime` |
| Business date (no time) | `LocalDateTime` | `LocalDate` |
| User-facing datetime | `LocalDateTime` | `ZonedDateTime` |
| Duration / elapsed time | `long` millis | `Duration` |

**Watch for:**
- `LocalDateTime.now()` used for audit fields — stores no timezone, breaks when server TZ changes.
- `Date` or `Calendar` anywhere in new code — legacy API, should not appear in Java 21 code.
- Comparing `LocalDateTime` instances without consistent timezone context.

---

### 6. `ConcurrentModificationException` Risk

Look for modifications to a collection while iterating it:

```java
// ❌ throws ConcurrentModificationException
for (Order o : orders) {
    if (o.isCancelled()) orders.remove(o);
}

// ✅ use removeIf
orders.removeIf(Order::isCancelled);

// ✅ or collect to new list
orders.stream().filter(o -> !o.isCancelled()).toList();
```

---

### 7. Soft Delete Correctness

This project uses `active` flag + `deactivated_at`/`deactivated_by` fields. Verify:

- All queries filter `WHERE active = true` (or `active IS TRUE`) — missing this returns deactivated records.
- Deactivation sets both `deactivated_at = now()` and `deactivated_by = currentUser`, not just the flag.
- No hard deletes (`DELETE FROM ...`) in code that should soft-delete.
- Cascade operations don't accidentally reactivate soft-deleted children.

---

### 8. UUID / ID Type Consistency

IDs use `VARCHAR(40)` / string-typed UUIDs per project convention. Verify:

- New entities store ID as `String`, not `Long` or `UUID`.
- ID generation uses `UUID.randomUUID().toString()` or equivalent.
- No numeric sequence IDs introduced in new entities.
- Foreign key columns match the ID column type of the referenced table.
