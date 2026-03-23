---
description: Generate a comprehensive integration test following project best practices
---

You are tasked with writing an integration test for a Spring Boot application. Follow these guidelines carefully:

## Testing Philosophy

### Mocking Strategy
- **Use mocks ONLY for dependencies we cannot control:**
  - AMQP/RabbitMQ connections
  - HTTP API endpoints (use WireMock)
  - External services
- **DO NOT mock internal components** that can be tested directly
- **Use Testcontainers** for database testing
- **Prefer testing through the public API** (controllers, REST endpoints)

### Test Structure
- Use **@Nested classes** to organize test cases by category
- Use **@DisplayName** annotations for all test classes and methods
- Follow the Given-When-Then pattern in test methods
- Group tests logically:
  - Successful Operations
  - Business Rule Violations
  - Input Validation Errors
  - Data Format Validation
  - Error Handling and Edge Cases

## Required Annotations

```java
@SpringBootTest(
    classes = YourApplication.class,
    webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT
)
@AutoConfigureMockMvc
@ActiveProfiles({"test", "no-security"})
@WireMockTest(httpPort = 8181)
@TestPropertySource(properties = {
    "app.external-service.base-url=http://localhost:8181"
})
@DisplayName("Your Feature Integration Tests")
```

## Test Class Structure

1. **Constants Section**: Define API endpoints, error messages, and test data constants
2. **Dependencies**: Inject required beans (@Autowired, @PersistenceContext)
3. **Setup/Teardown**:
   - `@BeforeEach`: Reset WireMock, clear database, setup test data
   - `@AfterEach`: Clean up WireMock
4. **Nested Test Classes**: Organize by test category
5. **Helper Methods**: Private methods for creating test data and mocking

## Test Coverage Requirements

### Happy Path Tests
- Valid data with sufficient resources
- Boundary conditions (exact balance matches, edge values)
- Different valid scenarios

### Validation Tests
- Missing required fields
- Invalid field values
- Non-existing entities (IDs, references)
- Wrong data types
- Format validation (decimals, dates, etc.)

### Business Logic Tests
- Insufficient funds/resources
- Currency mismatches
- Date validations (too old, future dates)
- State conflicts
- Permission violations

### Error Handling Tests
- API errors (500, 404, etc.)
- Malformed responses
- Null inputs
- Network timeouts (if applicable)

## Code Patterns

### Test Method Template
```java
@Test
@Transactional
@DisplayName("Should [expected behavior] when [condition]")
void testMethodName() throws Exception {
    // Given - Setup test data and mocks
    setupMocksForExternalService(testData);

    Request request = Request.builder()
        .field1(value1)
        .field2(value2)
        .build();

    // When & Then - Execute and verify
    mockPost(API_ENDPOINT, request)
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.field1").value(expected1))
        .andExpect(jsonPath("$.field2").value(expected2));
}
```

### Parameterized Tests
```java
@ParameterizedTest
@ValueSource(strings = {"0", "-100", "-0.01"})
@DisplayName("Should reject operation with invalid values")
void testWithMultipleValues(String value) throws Exception {
    // Test implementation
}
```

### WireMock Setup
```java
private void setupExternalServiceMock(String id, double value) throws Exception {
    stubFor(get(urlPathEqualTo("/api/v2/resource"))
        .withQueryParam("id", equalTo(id))
        .willReturn(aResponse()
            .withStatus(200)
            .withHeader("Content-Type", MediaType.APPLICATION_JSON_VALUE)
            .withBody(objectMapper.writeValueAsString(responseObject))));
}
```

### Helper Methods
- Create reusable methods for test data setup
- Extract common mock configurations
- Use builder pattern for test objects
- Keep helper methods focused and single-purpose

## Verification Best Practices

1. **Verify positive fields explicitly**: Check all expected response fields
2. **Verify negative cases**: Use `.doesNotExist()` for fields that shouldn't be present
3. **Verify API interactions**: Use `verify()` to ensure correct API calls
4. **Assert exceptions**: Use `assertThatThrownBy()` with specific exception types and messages
5. **Check database state**: Query EntityManager when needed to verify persistence

## Example Nested Class Structure

```java
@Nested
@DisplayName("Successful Operations")
class SuccessfulOperations {
    // Happy path tests
}

@Nested
@DisplayName("Business Rule Violations")
class BusinessRuleViolations {
    // Business logic validation tests
}

@Nested
@DisplayName("Input Validation Errors")
class InputValidationErrors {
    // Field validation tests
}

@Nested
@DisplayName("Data Format Validation")
class DataFormatValidation {
    // Format and type validation tests
}

@Nested
@DisplayName("Error Handling and Edge Cases")
class ErrorHandlingAndEdgeCases {
    // Error scenarios and edge cases
}
```

## Important Notes

- **Always use @Transactional** for tests that modify database state
- **Reset WireMock** in @BeforeEach and @AfterEach
- **Use meaningful test data constants** instead of magic numbers
- **Write descriptive assertion messages** when using custom assertions
- **Test both success and failure paths** for every feature
- **Consider timezone handling** when testing with dates
- **Use ObjectMapper** for JSON serialization in mocks
- **Extract common setup** into helper methods
- **Keep tests independent** - each test should work in isolation

## Your Task

Based on the above guidelines, generate a comprehensive integration test for the feature the user describes. Ensure:
1. All test categories are covered
2. Edge cases are considered
3. Error messages match business requirements
4. WireMock is properly configured for external dependencies
5. Test data is realistic and meaningful
6. All assertions are specific and thorough

Ask the user for clarification if you need more details about:
- The feature being tested
- External dependencies that need mocking
- Business rules and validations
- Expected error messages
- Database schema details
