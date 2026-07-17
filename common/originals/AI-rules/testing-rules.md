# Common Testing Rules

## Test Naming

- Integration tests: Suffix with `IT` exactly (e.g., `CashierControllerIT`, `UserServiceIT`)
- Unit tests: Must NOT use `IT` suffix (e.g., `CategoryValidatorTest`, `PasswordEncoderTest`)
- Never use `ITTest` or `TestIT`

**Why?** Clear naming allows test frameworks to separate unit and integration test runs with different configurations and coverage reporting.

**Language-Specific Setup:** See language-specific rules (Java, C++, TypeScript/Node, Angular) in `../development-rules/language-specific/` for framework/language configuration details.

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
