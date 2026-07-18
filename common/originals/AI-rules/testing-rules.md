# Common Testing Rules

## Core Testing Principles

Testing follows seven core principles that guide design, implementation, and execution:

1. **Iterative TDD** — Development driven by end-to-end (IT) tests, pausing at layer boundaries to create focused layer tests (LT)
2. **Layer-Scoped** — Each test targets a single responsibility boundary; LT tests one layer, IT tests full flows
3. **Deterministic** — Tests repeat with identical results; no sleeps, randomness, or timing dependencies
4. **Isolated** — Each test runs independently with clean state; no cross-test dependencies or shared side effects
5. **Security-Focused** — Tests validate security boundaries, especially cross-tenant isolation and credential protection
6. **Critical-Issue-Only Units** — Unit tests (*Test) written ONLY for critical issues (threading, DB transactions, file I/O)
7. **North Star IT** — End-to-end Playwright/Selenium IT tests are the ultimate validation that features work

These principles ensure tests are fast, reliable, maintainable, and catch real bugs.

---

## Test Naming (MANDATORY)

**Test Suffix Conventions:**
- **Unit tests:** Suffix with `Test` (e.g., `CategoryValidatorTest`, `PasswordEncoderTest`)
- **Layer tests:** Suffix with `LT` (e.g., `UserServiceLT`, `OrderRepositoryLT`)
- **Integration tests:** Suffix with `IT` (e.g., `CashierControllerIT`, `UserServiceIT`)

**Never use:** `ITTest`, `TestIT`, `LTTest`, `TestLT` (double suffixes)

**Why?** Clear naming allows test frameworks to separate unit, layer, and integration test runs with different configurations and coverage reporting. This enables running different test types independently with appropriate isolation levels.

**Language-Specific Setup:** See language-specific rules (Java, C++, TypeScript/Node, Angular) in `../development-rules/language-specific/` for framework/language configuration details.

### Test Profile Configuration (MANDATORY)

Tests MUST be separated by profiles in build configuration:

**Maven Profiles (Java):**
- `default` profile → Runs **only** `*Test.java` files (fast unit tests)
- `-P layer` profile → Runs **only** `*LT.java` files (layer tests with isolated dependencies)
- `-P integration` profile → Runs **only** `*IT.java` files (full integration tests)
- `-P all-tests` profile → Runs all three (unit + layer + integration)

**Build Commands:**
```bash
mvn test                           # Runs unit tests only (fastest, default)
mvn test -P layer                  # Runs layer tests only
mvn test -P integration            # Runs integration tests only
mvn test -P all-tests              # Runs all test types
```

**Why?** 
- Unit tests run fast (~seconds) with mocks → suitable for pre-commit checks
- Layer tests run medium speed (~minutes) with test containers → suitable for component validation
- Integration tests run slow (~minutes+) with real services → suitable for CI/CD pipeline
- Developers run unit tests locally; CI runs all tests before merge

See Java/language-specific templates for profile configuration details.

### Coverage Reports (MANDATORY)

Each test run MUST generate a separate coverage report:

**Report Generation:**
- `mvn test` → Generates `target/site/jacoco/index.html` (unit test coverage)
- `mvn test -P layer` → Generates `target/site/jacoco-layer/index.html` (layer test coverage)
- `mvn test -P integration` → Generates `target/site/jacoco-integration/index.html` (integration test coverage)
- `mvn test -P all-tests` → Generates combined report (all test coverage)

**Coverage Report Contents:**
- Line coverage (% of lines executed)
- Branch coverage (% of decision branches taken)
- Method coverage (% of methods called)
- Class coverage (% of classes executed)
- Breakdown by package and class

**Why?** Separate reports show which code is covered by which test type:
- Unit test coverage → Fast feedback on business logic correctness
- Layer test coverage → Validates internal layer contracts
- Integration test coverage → Verifies end-to-end workflows work

Review coverage reports regularly to identify gaps and improve test quality. Set minimum coverage thresholds per test type.

See Java/language-specific templates for JaCoCo configuration.

### Iterative TDD Workflow (MANDATORY)

Development is driven by end-to-end (IT) tests that iterate through layers:

**Workflow:**
1. Write end-to-end IT test (full feature flow)
2. Run → fails at a layer boundary (e.g., Service → Repository)
3. **Pause the IT** (mark as suspended/skipped, don't proceed)
4. **Create LT for that layer** (test only the paused layer with mocked adjacent layers)
5. Implement code to make LT pass
6. Resume IT → continues, fails at next boundary
7. Repeat until IT passes end-to-end

**Key Constraint:** LT scope = exactly what the paused IT needs at that layer, nothing more. Prevents over-engineering and scope creep.

**Example:**
```
Paused IT: "User creates order and receives confirmation"
↓ Fails: OrderService cannot create order
↓ Create LT: OrderServiceLT tests createOrder() with mocked OrderRepository
↓ LT passes, OrderService implemented → Resume IT
↓ IT fails: OrderRepository cannot persist to database
↓ Create LT: OrderRepositoryLT tests SQL query execution
↓ LT passes → Resume IT
↓ IT passes end-to-end ✅
```

### Layer-Scoped Testing (MANDATORY)

Each test targets a single responsibility boundary and does not span unnecessary layers.

**Rules:**
- **LT tests one layer** — Mock or stub all adjacent layers
- **IT tests flow across layers** — Only start IT when all layers are connected
- **No cross-layer leakage** — A Service LT does not test Repository internals

**Benefits:**
- Failures are localized (easy to debug)
- Tests run fast (fewer dependencies)
- Layer boundaries are explicit
- Refactoring one layer doesn't break unrelated tests

**Testing Pyramid:**
```
       ↑ End-to-End (IT) — Slow, comprehensive, no mocks
       │ Layer Tests (LT) — Medium speed, mocked adjacent layers
       │ Unit Tests (Test) — Fast, but ONLY for critical issues
       ↓ (rare)
```

### Critical-Issue-Only Unit Tests

Unit tests (*Test) are written ONLY for critical issues that require isolated testing.

**When to write unit tests:**
- ✅ Threading/concurrency bugs discovered
- ✅ Database transaction isolation issues
- ✅ File I/O edge cases or race conditions
- ❌ Default unit tests for normal code paths

**Benefits:**
- Faster test suite (fewer tests overall)
- Focus on integration (where real bugs hide)
- Unit tests stay high-signal (critical issues only)
- No coverage target pressure

**Example:**
```java
// ❌ Don't write this by default
TEST(OrderValidator, ValidateEmail) { ... }
TEST(OrderValidator, ValidateAmount) { ... }

// ✅ Write this only if concurrency bug found
TEST(OrderValidator, ThreadSafeValidation) {
    // Test only the critical race condition
    Thread t1 = new Thread(() -> validator.validate(order1));
    Thread t2 = new Thread(() -> validator.validate(order2));
    t1.start(); t2.start();
    t1.join(); t2.join();
    // Verify no race condition, no data corruption
}
```

---

## Test Data Safety

- Never use production credentials in tests
- Never use personal data (real names, emails, phone numbers)
- Never use live accounts or real services
- Never use real documents or files in test fixtures

**Why?** Protects privacy, prevents accidental data corruption, allows tests to run safely in CI/CD.

---

## Test Quality Standards

### Determinism, Independence, Repeatability (MANDATORY)

Tests MUST produce the same result every run, regardless of:
- Execution order
- Wall-clock timing (current time/date)
- Public networks
- Mutable external data

**Freeze or inject dependencies:**
- Time: Use `Clock.fixed()` instead of `System.currentTimeMillis()`
- Random: Inject seeded random generator instead of `new Random()`
- External services: Mock provider clients
- Exchange rates, configuration: Inject test values

**Example - Correct**:
```java
@Test
public void shouldCalculatePrice() {
    Clock clock = Clock.fixed(Instant.parse("2026-07-12T10:00:00Z"), UTC);
    PriceCalculator calc = new PriceCalculator(clock, mockRandom);
    
    BigDecimal price = calc.calculate(items);
    assertThat(price).isEqualTo(BigDecimal.valueOf(99.99));  // Deterministic
}
```

**Example - Wrong (Flaky)**:
```java
@Test
public void shouldCalculatePrice() {
    BigDecimal price = calc.calculate(items);
    assertThat(price).isCloseTo(expectedPrice, within(1));  // Result varies by time
}
```

### Observe Business Outcomes, Not Implementation Details

Assert what the user sees and cares about, not internal implementation.

**Correct**:
```java
@Test
public void shouldProcessPayment() {
    service.processPayment(order);
    assertThat(order.getStatus()).isEqualTo(PAID);  // Observable outcome
}
```

**Wrong**:
```java
@Test
public void shouldProcessPayment() {
    service.processPayment(order);
    assertThat(paymentCounter.getCount()).isEqualTo(1);  // Implementation detail
}
```

### Regression Fix: Test First, Then Fix

A regression fix MUST be represented by a test that fails for the reported defect.

1. Write test that demonstrates the bug (fails)
2. Fix the bug (test passes)
3. Commit both together

Never create test after fix (loses the failure evidence).

### Never Weaken Tests to Get Green Builds

Failing tests indicate real problems. Fix the bugs, not the tests.

**Never do this**:
```java
@Test
@Disabled  // ❌ Don't skip!
public void shouldValidateEmail() {
    // ...
}
```

```java
@Test
public void shouldValidateEmail() {
    assertThat(result).isNotNull();  // ❌ Too vague, tests nothing
}
```

```java
when(validator.validate(any())).thenReturn(true);  // ❌ Broad mock, defeats test
```

### End-to-End Validation — The North Star (MANDATORY)

Integration tests (IT) are the ultimate validation that features work end-to-end.

**Rules:**
- **Every feature has an IT** — Playwright/Selenium IT validates the full user flow
- **IT runs on real system** — No mocking, no stubs; all layers connected
- **IT validates user perspective** — Tests what the user sees and does, not internal implementation
- **LT supports IT** — Layer tests fill gaps when IT fails at boundaries, but IT is the goal
- **Pass IT = feature done** — When IT passes, the feature works end-to-end

**Example:**
```typescript
// ✅ North Star IT: User completes order end-to-end
test('user creates order and receives confirmation', () => {
  const client = new WebClient();
  client.login('user@example.com', 'password');
  
  client.navigate('/products');
  client.addToCart('Widget', 1);
  client.checkout();
  
  const confirmation = client.getCurrentPage();
  expect(confirmation).toContain('Order #12345 confirmed');  // What user sees
  expect(confirmation).toContain('ship to: 123 Main St');
});
```

When this IT passes, the feature works end-to-end. Supporting LTs verify individual layers (auth, cart, checkout, shipping) but the IT is the north star.

---

## Isolation & Security Gates

### Cross-Tenant Denial Tests (MANDATORY)

Every data-access feature MUST include a test that verifies cross-tenant access is DENIED.

**Example**:
```java
@Test
public void userFromTenantA_cannotAccess_TenantB_data() {
    User userA = createUserInTenant(TENANT_A);
    CompanyData dataFromTenantB = getDataFromTenant(TENANT_B);
    
    assertThatThrownBy(() -> service.read(dataFromTenantB, userA))
        .isInstanceOf(AccessDeniedException.class);
}
```

Also test cross-scope access (user within tenant, but different department/role).

### Secrets Not in Output (MANDATORY)

Assert that secrets, tokens, and confidential data do NOT appear in:
- Logs
- Audit events
- Error messages
- API responses
- Snapshots

**Example**:
```java
@Test
public void apiResponse_doesNotExpose_secrets() {
    String apiKey = "super-secret-key-12345";
    ApiResponse response = callApiWithSecret(apiKey);
    
    assertThat(response.getBody()).doesNotContain(apiKey);
    assertThat(response.getBody()).doesNotContain("secret");
    assertThat(response.getHeaders()).doesNotContainValue(apiKey);
}
```

---

## Test Uploads (Project-Specific)

- Test uploads placed in directory defined in project's local `AI-rules/testing-rules.md`
- Tests that process uploaded files must fetch recursively from that directory
- Never hardcode individual file paths in test code
- Clear test-upload directory before each test or test suite

---

## Test Logging (Project-Specific)

Create new log file in `logs/` at test suite start:
- Filename format: `Testing-YYMMDD-HH:MM CEST.log.md`
- Example: `Testing-260618-15:50 CEST.log.md`
- List test suite name and one-line description for each test

---

**Last Updated:** 2026-07-17
