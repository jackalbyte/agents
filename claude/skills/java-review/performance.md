# Java Performance Review

Scan changed code for anti-patterns that compile cleanly but degrade throughput, increase heap pressure, or worsen GC behavior. Focus on **hot paths** — methods called per-request, per-event, or inside processing loops. A pattern in startup code or a rare utility is not a finding.

---

## Checklist

### 1. String Concatenation in Loops
Look for `+` used to build a `String` inside a `for`/`while` body.

**Why it matters:** `String` is immutable — each `+` allocates a full copy. O(n²) character copying as the accumulator grows.

**Fix:** Declare a `StringBuilder` before the loop; `append()` inside; `toString()` once after.

```java
// before
String result = "";
for (String line : lines) { result = result + line + "\n"; }

// after
StringBuilder sb = new StringBuilder();
for (String line : lines) { sb.append(line).append('\n'); }
String result = sb.toString();
```

Note: The compiler optimizes single-line `"a" + b + "c"` since JDK 9, but does **not** lift that optimization across loop iterations.

---

### 2. Stream Inside a Loop (Accidental O(n²))
Look for `.stream()` or `.parallelStream()` called on a collection **inside** a loop that also iterates that collection or another large one.

**Why it matters:** Each outer iteration pays the full streaming cost. 10k elements × 10k-element stream = 100M operations instead of 10k.

**Fix:** Single-pass `Map.merge()`, `Collectors.groupingBy()`, or pre-compute a lookup map.

```java
// before — O(n²)
for (Order o : orders) {
    long count = orders.stream().filter(x -> x.hour() == o.hour()).count();
    byHour.put(o.hour(), count);
}

// after — O(n)
for (Order o : orders) {
    byHour.merge(o.hour(), 1L, Long::sum);
}
```

---

### 3. `String.format()` in Hot Paths
Look for `String.format(...)` in methods that run per-request or inside loops.

**Why it matters:** Parses the format string on every call via regex through `java.util.Formatter`. Consistently the slowest string-building option in JMH benchmarks.

**Fix:** Use `+` concatenation (compiler-optimized) or `StringBuilder`. Reserve `String.format()` for numeric formatting or cold paths.

```java
// before
return String.format("Order %s for %s: $%.2f", id, customer, amount);

// after
return "Order " + id + " for " + customer + ": $" + String.format("%.2f", amount);
```

---

### 4. Autoboxing in Hot Paths
Look for `Integer`, `Long`, `Double`, `Boolean` used as local accumulators in loops, or as frequent `Map.get()`/`Map.put()` targets.

**Why it matters:** Each box/unbox round trip allocates a wrapper (~16 bytes per `Long`). A million-iteration loop = a million GC-eligible objects.

**Fix:** Use primitive types for local variables. Use `LongAdder` for concurrent counters.

```java
// before
Long sum = 0L;
for (Long value : values) { sum += value; }  // box every iteration

// after
long sum = 0L;
for (long value : values) { sum += value; }
```

---

### 5. Exceptions for Control Flow
Look for `try/catch` used to test routine conditions (invalid format, absent value) where the exception is expected to fire often.

**Why it matters:** `Throwable.fillInStackTrace()` walks the entire JVM call stack on every throw. Deep Spring call stacks make this very expensive. JMH shows exception-based branching can be 100×–500× slower than a conditional.

**Fix:** Pre-validate, keep catch as safety net for genuine edge cases.

```java
// before
try { return Integer.parseInt(value); }
catch (NumberFormatException e) { return defaultValue; }

// after
if (value == null || value.isBlank()) return defaultValue;
for (int i = 0; i < value.length(); i++) {
    char c = value.charAt(i);
    if (i == 0 && c == '-') continue;
    if (!Character.isDigit(c)) return defaultValue;
}
try { return Integer.parseInt(value); }
catch (NumberFormatException e) { return defaultValue; }
```

---

### 6. Too-Broad Synchronization
Look for `synchronized` on whole methods or `Collections.synchronizedMap()` in classes accessed by multiple threads.

**Why it matters:** One lock serializes all callers — becomes the bottleneck under concurrency.

**Fix:** `ConcurrentHashMap` for maps, `LongAdder` for counters, `ReentrantLock` to narrow the critical section.

```java
// before
private final Map<String, Long> counts = new HashMap<>();
public synchronized void increment(String key) { counts.merge(key, 1L, Long::sum); }

// after
private final ConcurrentHashMap<String, LongAdder> counts = new ConcurrentHashMap<>();
public void increment(String key) {
    counts.computeIfAbsent(key, k -> new LongAdder()).increment();
}
```

---

### 7. Recreating Reusable Objects
Look for `new ObjectMapper()`, `new Gson()`, `new XmlMapper()`, `DateTimeFormatter.ofPattern(...)` constructed inside methods.

**Why it matters:** These constructors do substantial work (module discovery, cache init, pattern compile). Designed to be built once and shared.

**Fix:** `private static final` constant. `ObjectMapper` is thread-safe after config. `DateTimeFormatter` is immutable.

```java
// before
public String serialize(Order order) throws Exception {
    return new ObjectMapper().writeValueAsString(order);
}

// after
private static final ObjectMapper MAPPER = new ObjectMapper();
public String serialize(Order order) throws Exception {
    return MAPPER.writeValueAsString(order);
}
```

---

### 8. Virtual Thread Pinning (JDK 21–23)
Look for `synchronized` methods/blocks that contain blocking I/O or JDBC calls.

**Why it matters:** On JDK 21–23, a virtual thread inside `synchronized` that blocks pins its carrier thread — the carrier cannot serve other virtual threads. Enough pinned carriers stall the whole application. JFR `jdk.VirtualThreadPinned` event reveals this.

**Fix on JDK 21–23:** Replace `synchronized` with `ReentrantLock`.

```java
// before — pins carrier on JDK 21-23
public synchronized String fetch(String key) throws IOException {
    return Files.readString(Path.of("/data/" + key));
}

// after
private final ReentrantLock lock = new ReentrantLock();
public String fetch(String key) throws IOException {
    lock.lock();
    try { return Files.readString(Path.of("/data/" + key)); }
    finally { lock.unlock(); }
}
```

**JDK 24+:** JEP 491 resolves most `synchronized` pinning. Only native calls can still pin. Skip this check if the project targets JDK 24+.
