---
description: Analyze code for performance bottlenecks and optimization opportunities
allowed-tools: Bash(git diff:*), Read, Grep, Glob
---

# Performance Review

Analyze recent changes for performance issues and optimization opportunities.

## 1. Database Performance Analysis

### Query Efficiency
- [ ] Check for N+1 query problems
- [ ] Verify proper use of indexes
- [ ] Review query complexity
- [ ] Check for missing JOINs optimization
- [ ] Evaluate pagination implementation
- [ ] Review transaction boundaries

### Connection Management
- [ ] Connection pool sizing
- [ ] Connection leak detection
- [ ] Proper connection closing
- [ ] Statement reuse and prepared statements

### ORM/Query Builder Issues
- [ ] Lazy vs eager loading strategy
- [ ] Unnecessary data fetching (SELECT *)
- [ ] Batch operations vs individual queries
- [ ] Proper use of caching at ORM level

## 2. Algorithm Complexity Analysis

### Time Complexity
- [ ] Identify O(n²) or worse algorithms
- [ ] Check for nested loops over large datasets
- [ ] Review recursive operations for stack depth
- [ ] Evaluate sorting algorithm choices
- [ ] Check for redundant computations

### Space Complexity
- [ ] Memory allocation patterns
- [ ] Large data structure usage
- [ ] Unnecessary data copying
- [ ] Memory leaks potential
- [ ] Buffer sizing

## 3. Concurrency & Parallelism

### Thread Safety
- [ ] Proper synchronization
- [ ] Race condition prevention
- [ ] Deadlock potential
- [ ] Lock contention analysis

### Concurrency Patterns (Go-specific)
- [ ] Goroutine leak detection
- [ ] Channel buffer sizing
- [ ] Select statement efficiency
- [ ] Context cancellation handling
- [ ] WaitGroup usage

### Thread Pools (Java-specific)
- [ ] Thread pool sizing
- [ ] Task queue management
- [ ] ExecutorService usage
- [ ] CompletableFuture chains

### Async/Await (Python-specific)
- [ ] Proper async/await usage
- [ ] Event loop blocking prevention
- [ ] Concurrent task management
- [ ] Async I/O optimization

## 4. I/O Operations

### File I/O
- [ ] Buffered I/O usage
- [ ] Stream processing for large files
- [ ] Proper resource cleanup
- [ ] Async file operations where appropriate

### Network I/O
- [ ] HTTP client timeout configuration
- [ ] Connection reuse (Keep-Alive)
- [ ] Compression usage
- [ ] Retry logic efficiency
- [ ] Streaming for large payloads

### API Calls
- [ ] Batch API calls where possible
- [ ] Parallel requests for independent calls
- [ ] Circuit breaker implementation
- [ ] Response caching strategy

## 5. Caching Strategy

### Cache Usage
- [ ] Appropriate cache keys
- [ ] Cache invalidation strategy
- [ ] Cache hit rate optimization
- [ ] Cache size limits
- [ ] TTL configuration

### Cache Levels
- [ ] Application-level caching (in-memory)
- [ ] Distributed cache usage (Redis, Memcached)
- [ ] HTTP caching headers
- [ ] Database query result caching

## 6. Serialization & Deserialization

- [ ] Efficient serialization format (JSON vs Protocol Buffers vs MessagePack)
- [ ] Lazy deserialization opportunities
- [ ] Schema evolution considerations
- [ ] Large object serialization

## 7. Resource Management

### Memory
- [ ] Object pooling opportunities
- [ ] String concatenation in loops
- [ ] Large collection handling
- [ ] Memory profiling recommendations

### CPU
- [ ] CPU-intensive operations
- [ ] Regex compilation caching
- [ ] Reflection usage
- [ ] Lazy initialization

### Network
- [ ] Bandwidth optimization
- [ ] Payload size reduction
- [ ] Compression usage
- [ ] CDN utilization

## 8. Logging & Monitoring

- [ ] Log level appropriate for production
- [ ] Expensive logging operations
- [ ] Structured logging usage
- [ ] Metrics collection overhead
- [ ] Trace sampling strategy

## 9. Framework-Specific Issues

### Spring Boot (Java)
- [ ] Component scan optimization
- [ ] Lazy bean initialization
- [ ] Actuator endpoint security
- [ ] JPA fetch strategies

### Standard Library (Go)
- [ ] Sync.Pool usage for object reuse
- [ ] Proper defer placement
- [ ] String builder usage
- [ ] Map pre-allocation

### Framework (Python)
- [ ] Generator vs list comprehensions
- [ ] Django QuerySet optimization
- [ ] FastAPI async endpoint usage
- [ ] Pickle vs JSON

## 10. Performance Testing Recommendations

Based on findings, recommend:
- Load testing scenarios
- Benchmarking specific functions
- Profiling CPU/memory
- Database query analysis tools
- Performance regression tests

---

## Output Format

Provide a structured report:

### Summary
- Overall performance assessment
- Critical issues count
- Optimization opportunities count

### Critical Performance Issues
List issues with HIGH impact on performance:
- Description
- Location (file:line)
- Impact assessment
- Recommended fix

### Optimization Opportunities
List MEDIUM/LOW priority improvements:
- Description
- Location (file:line)
- Expected gain
- Implementation effort

### Recommendations
- Immediate actions required
- Long-term optimization strategy
- Performance testing plan
- Monitoring and alerting setup

### Metrics to Track
- Response time percentiles (p50, p95, p99)
- Throughput (requests/second)
- Error rates
- Resource utilization (CPU, memory, I/O)
