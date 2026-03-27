---
description: Review architectural decisions, design patterns, and code structure
allowed-tools: Bash(git diff:*), Read, Grep, Glob
---

# Architecture Review

Review architectural decisions, design patterns, and overall code structure for maintainability and scalability.

## 1. SOLID Principles Assessment

### Single Responsibility Principle (SRP)
- [ ] Each class/module has one reason to change
- [ ] Clear separation of concerns
- [ ] No "God objects" or "Swiss Army knife" classes
- [ ] Functions/methods have single, well-defined purpose

### Open/Closed Principle (OCP)
- [ ] Open for extension, closed for modification
- [ ] Extensibility through interfaces/abstractions
- [ ] Plugin architecture where appropriate
- [ ] Minimal conditional complexity

### Liskov Substitution Principle (LSP)
- [ ] Subtypes can replace base types
- [ ] Proper inheritance hierarchies
- [ ] No broken abstractions
- [ ] Interface segregation

### Interface Segregation Principle (ISP)
- [ ] Clients not forced to depend on unused methods
- [ ] Focused, cohesive interfaces
- [ ] Proper abstraction levels

### Dependency Inversion Principle (DIP)
- [ ] Depend on abstractions, not concretions
- [ ] Dependency injection usage
- [ ] Inversion of Control (IoC)
- [ ] Testability through interfaces

## 2. Design Patterns Analysis

### Appropriate Pattern Usage
- [ ] Patterns solve actual problems (not over-engineering)
- [ ] Correct pattern implementation
- [ ] Pattern documentation in code

### Common Patterns to Review
- **Creational**: Factory, Builder, Singleton (avoid excessive use)
- **Structural**: Adapter, Decorator, Facade, Proxy
- **Behavioral**: Strategy, Observer, Command, Template Method
- **Concurrency**: Producer-Consumer, Thread Pool, Future/Promise

### Anti-patterns Detection
- [ ] God Object
- [ ] Spaghetti Code
- [ ] Lava Flow (dead code)
- [ ] Golden Hammer (overusing one solution)
- [ ] Premature Optimization
- [ ] Tight Coupling
- [ ] Circular Dependencies

## 3. Layered Architecture Review

### Layer Separation
- [ ] Clear boundaries between layers
- [ ] Proper dependency direction (top-down)
- [ ] No layer bypassing
- [ ] Appropriate layer responsibilities

### Common Layers
- **Presentation**: Controllers, HTTP handlers, API endpoints
- **Business Logic**: Services, domain models, use cases
- **Data Access**: Repositories, DAOs, database queries
- **Infrastructure**: External integrations, utilities

### Layer Communication
- [ ] DTOs for data transfer between layers
- [ ] Proper exception handling at layer boundaries
- [ ] No leaky abstractions
- [ ] Clean data flow

## 4. Domain-Driven Design (DDD) Principles

### Domain Modeling
- [ ] Rich domain models vs anemic models
- [ ] Bounded contexts properly defined
- [ ] Ubiquitous language usage
- [ ] Aggregates and entities properly identified

### Repositories & Services
- [ ] Repository pattern for data access
- [ ] Domain services for business logic
- [ ] Application services for orchestration
- [ ] Clear service boundaries

## 5. Microservices Architecture (if applicable)

### Service Design
- [ ] Services aligned with business capabilities
- [ ] Appropriate service granularity
- [ ] Independent deployability
- [ ] Database per service pattern

### Communication Patterns
- [ ] Synchronous vs asynchronous communication
- [ ] API gateway usage
- [ ] Service discovery
- [ ] Circuit breakers and resilience patterns

### Data Management
- [ ] Eventual consistency strategy
- [ ] Saga pattern for distributed transactions
- [ ] Event sourcing where appropriate
- [ ] CQRS pattern usage

## 6. Code Organization & Structure

### Package/Module Structure
- [ ] Logical grouping of related code
- [ ] Clear naming conventions
- [ ] Appropriate visibility modifiers (public/private)
- [ ] Minimal cross-package dependencies

### Naming Conventions
- [ ] Consistent naming across codebase
- [ ] Descriptive and meaningful names
- [ ] Follows language/framework conventions
- [ ] No misleading names

### File Organization
- [ ] Reasonable file sizes
- [ ] Related code co-located
- [ ] Test files properly organized
- [ ] Configuration files organized

## 7. Coupling & Cohesion

### Coupling Analysis
- [ ] Loose coupling between modules
- [ ] Minimal inter-module dependencies
- [ ] Interface-based communication
- [ ] No circular dependencies

### Cohesion Assessment
- [ ] High cohesion within modules
- [ ] Related functionality grouped together
- [ ] Clear module boundaries
- [ ] Single purpose per module

## 8. Scalability Considerations

### Horizontal Scalability
- [ ] Stateless application design
- [ ] Session management strategy
- [ ] Load balancing readiness
- [ ] Distributed caching

### Vertical Scalability
- [ ] Efficient resource utilization
- [ ] Threading/concurrency model
- [ ] Memory management
- [ ] CPU optimization

### Data Scalability
- [ ] Database sharding strategy
- [ ] Read replicas usage
- [ ] Caching strategy
- [ ] Query optimization

## 9. Maintainability & Technical Debt

### Code Readability
- [ ] Self-documenting code
- [ ] Appropriate comments (why, not what)
- [ ] Consistent formatting
- [ ] Reasonable complexity

### Testability
- [ ] Unit test coverage
- [ ] Mock-friendly design
- [ ] Test doubles support
- [ ] Integration test strategy

### Technical Debt
- [ ] Identify technical debt areas
- [ ] Assess debt severity
- [ ] Refactoring opportunities
- [ ] Deprecation of legacy code

### Documentation
- [ ] Architecture decision records (ADRs)
- [ ] API documentation
- [ ] Code-level documentation
- [ ] Deployment documentation

## 10. Error Handling & Resilience

### Exception Handling
- [ ] Proper exception hierarchy
- [ ] Catch specific exceptions
- [ ] Meaningful error messages
- [ ] No swallowed exceptions

### Resilience Patterns
- [ ] Retry logic with backoff
- [ ] Circuit breakers
- [ ] Timeouts configured
- [ ] Graceful degradation
- [ ] Bulkheads for isolation

### Observability
- [ ] Structured logging
- [ ] Metrics collection
- [ ] Distributed tracing
- [ ] Health checks

## 11. Security Architecture

### Authentication & Authorization
- [ ] Proper authentication mechanism
- [ ] Role-based access control
- [ ] Principle of least privilege
- [ ] Secure credential storage

### Data Protection
- [ ] Encryption at rest and in transit
- [ ] Input validation
- [ ] Output encoding
- [ ] SQL injection prevention

### API Security
- [ ] Rate limiting
- [ ] CORS configuration
- [ ] API authentication
- [ ] Request validation

## 12. Performance Architecture

### Caching Strategy
- [ ] Multi-level caching
- [ ] Cache invalidation
- [ ] CDN usage
- [ ] Database query caching

### Asynchronous Processing
- [ ] Background jobs for long-running tasks
- [ ] Message queues
- [ ] Event-driven architecture
- [ ] Proper async/await usage

---

## Output Format

Provide a comprehensive report:

### Executive Summary
- Overall architecture health score (1-10)
- Major concerns
- Strengths

### Critical Issues
List architectural problems requiring immediate attention:
- Issue description
- Location/component affected
- Impact on system
- Recommended solution
- Refactoring effort

### Design Pattern Issues
- Misused patterns
- Missing patterns
- Anti-patterns found

### Technical Debt Assessment
- High-priority debt items
- Medium-priority improvements
- Long-term refactoring recommendations

### Scalability Assessment
- Current scalability limitations
- Bottlenecks
- Scaling recommendations

### Best Practices Violations
- SOLID principle violations
- Coupling/cohesion issues
- Code organization problems

### Recommendations
- Immediate actions
- Short-term improvements (next sprint)
- Long-term architecture evolution
- Refactoring strategy

### Positive Observations
- Well-implemented patterns
- Strong architectural decisions
- Good practices to continue
