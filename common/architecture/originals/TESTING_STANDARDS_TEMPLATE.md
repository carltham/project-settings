# Testing Standards

**Project:** [PROJECT_NAME]  
**Version:** 1.0  
**Last Updated:** [DATE]  
**Next Review:** [DATE + 3 MONTHS]

---

## How to Use This Template

**Instructions for project teams:**

1. Replace all `[PLACEHOLDER]` values with your project-specific information
2. Adjust coverage targets based on your project's risk profile
3. Remove sections that don't apply to your tech stack
4. Define tier-specific testing strategies
5. Align with your team's sprint capacity
6. Have tech lead and QA review and approve

**Key sections to customize:**
- Coverage targets (Section 3)
- Test execution schedule (Section 4)
- Tier-specific strategies (Section 5)
- Data safety requirements (Section 6)

---

## Overview

Testing is not optional. Tests verify that the system meets its architectural contract and prevents regressions.

**Testing Philosophy:**
- Tests are executable documentation of expected behavior
- Tests validate architecture compliance (layering, isolation, security)
- Tests must be deterministic, independent, and repeatable
- Tests are part of the build, not an afterthought

**Test Coverage by Layer:**
- Tier 1 (UI): [X]% coverage target (UI logic, event handling)
- Tier 2 (UIControllers): [X]% coverage target (state, delegation)
- Tier 3 (API): [X]% coverage target (endpoints, validation)
- Tier 4 (Services): [X]% coverage target (business logic, rules)
- Tier 5 (Repositories): [X]% coverage target (data access, queries)

---

## Test Classification

### Unit Tests

**Purpose:** Verify individual components in isolation

**Scope:**
- Classes: Services, Repositories, Utilities, Validators
- Each test verifies ONE behavior
- Mocked dependencies (database, external services)
- Run: `mvn test`
- Speed: Fast (< 1 second each)

**Naming:** `[Class]Test.java` or `[Class]Tests.java`

**Coverage Target:** [X]% for Tier 4 (Services)

### Integration Tests

**Purpose:** Verify components work together

**Scope:**
- API endpoints + Services + Repositories (minus Database)
- Mock external dependencies (APIs, payment gateways)
- In-memory or test database
- Run: `mvn verify -P integration-tests`
- Speed: Medium (1-10 seconds each)

**Naming:** `[Class]IT.java` (e.g., `CashierControllerIT`, `UserRepositoryIT`)

**Coverage Target:** [X]% for Tier 3-5 (API, Services, Repositories)

### End-to-End Tests

**Purpose:** Verify user workflows work completely

**Scope:**
- UI → API → Services → Database
- Real or realistic environment
- Slow but high-confidence
- Run: In staging environment
- Examples: "User can purchase an item", "Admin can create user"

**When to use:** Critical user paths only (not every feature)

**Coverage Target:** [X] critical user paths

---

## Test Quality Standards

### Determinism (MANDATORY)

Tests MUST produce the same result every run.

**Requirements:**
- No wall-clock time dependency: Inject `Clock.fixed()` instead of `System.currentTimeMillis()`
- No random number dependency: Inject seeded random instead of `new Random()`
- No external service dependency: Mock [EXTERNAL_SERVICES] in tests
- No mutable global state: Clear state before/after tests
- No execution order dependency: Tests run independently and in any order

**Verification:**
```bash
mvn test
mvn test  # Run twice - results must be identical
```

**If test is flaky:**
1. Find the time/random/external dependency
2. Inject a test double (fake, stub, mock)
3. Verify with repeated runs

### Test Independence (MANDATORY)

Tests must NOT depend on other tests.

**Requirements:**
- Each test sets up its own data
- Tests don't share state
- Test A passes even if Test B fails
- Tests can run in any order
- Tests can run in parallel (if configured)

**Anti-patterns:**
- ❌ Test B depends on Test A creating data
- ❌ Tests share a static cache
- ❌ Tests modify shared database state

### Regression Testing Workflow (MANDATORY)

When fixing a bug, represent the fix with a test first.

**Workflow:**
1. Write test that reproduces the bug (fails)
2. Fix the bug in code (test passes)
3. Commit test + fix together
4. Never commit fix without a failing test first

**Why:** Test demonstrates the bug existed and fix works.

### Prefer Observable Business Outcomes

Assert what the user experiences, not internal implementation.

**Correct:**
```java
order.getStatus() == PAID              // User sees order is paid
payment.isProcessed() == true          // Observable outcome
email.wasNotSent() == false            // Externally observable
```

**Wrong:**
```java
paymentCounter.increment() == 1        // Implementation detail
cache.size() > 0                       // Internal state
database.queries.count == 5            // Query optimization detail
```

### Never Weaken Tests for Green Builds (MANDATORY)

Failing tests indicate real problems. Fix the code, not the test.

**Never do this:**
- ❌ `@Disabled` (skip the test)
- ❌ Broad mocks: `when(anything()).thenReturn(anything())`
- ❌ Vague assertions: `assertThat(result).isNotNull()`
- ❌ Delete or delete assertions when they fail

**If test fails:**
1. Understand why
2. Fix the code
3. Verify test passes

---

## Security & Isolation Testing

### Cross-Tenant Denial Tests (MANDATORY)

Every data-access feature MUST include a test verifying cross-tenant access is DENIED.

**Requirements:**
- User from Tenant A cannot read/write data from Tenant B
- User cannot access data outside their scope (department, role, account)
- System rejects cross-tenant access before database query

**Test Example:**
```java
@Test
public void userFromTenantA_cannotAccess_TenantB_data() {
    User userA = createUser(TENANT_A);
    CompanyData dataFromTenantB = createData(TENANT_B);
    
    assertThatThrownBy(() -> service.read(dataFromTenantB, userA))
        .isInstanceOf(AccessDeniedException.class);
}
```

**Coverage:**
- Test each data-access method (read, write, delete, list)
- Test cross-tenant scenarios
- Test cross-scope scenarios (if applicable)

### Secrets Not in Output (MANDATORY)

Assert that secrets, tokens, and confidential data do NOT appear anywhere.

**Requirements:**
- Secrets must not appear in: logs, error messages, API responses, audit events
- Sensitive fields redacted in: snapshots, test output, debugging
- Test data uses fake/test credentials, never production credentials

**Test Example:**
```java
@Test
public void apiResponse_doesNotExpose_apiKey() {
    String apiKey = "test-key-12345";
    ApiResponse response = callApi(apiKey);
    
    assertThat(response.getBody()).doesNotContain(apiKey);
    assertThat(response.getHeaders()).doesNotContain(apiKey);
    assertThat(logs).doesNotContain(apiKey);
}
```

**Coverage:**
- Test each endpoint with secrets
- Test error responses (secrets must not leak)
- Test audit logs (secrets redacted)

---

## Data Safety (MANDATORY)

### Test Data Requirements

- ❌ Never use production credentials in tests
- ❌ Never use personal data (real names, emails, phone numbers)
- ❌ Never use live accounts or real services
- ❌ Never use real files or documents

### Test Environment Isolation

- Tests run against: [TEST_DATABASE/IN_MEMORY_DATABASE]
- External service calls mocked: [LIST_EXTERNAL_SERVICES]
- Test data cleared: [FREQUENCY - e.g., before each test]

---

## Tier-Specific Testing Strategy

### Tier 1 & 2: UI & UIControllers

**What to test:**
- State management (data set/get correctly)
- Delegation to API (correct method called with correct data)
- Error handling (exceptions displayed properly)
- Event handling (UI updates on event)

**What NOT to test:**
- Rendering details (pixel positions, colors)
- Swing/AWT framework behavior
- Database queries (mocked)

**Test Type:** Unit tests (fast)

**Coverage Target:** [X]%

### Tier 3: REST API

**What to test:**
- Endpoints respond with correct status codes
- Request validation (bad input rejected)
- Authorization (only authorized users can access)
- Response format matches specification

**What NOT to test:**
- Business logic (tested in Services)
- Database queries (tested in Repositories)
- Rendering/serialization details

**Test Type:** Integration tests (API + framework)

**Coverage Target:** [X]%

### Tier 4: Business Services

**What to test:**
- Business logic correctness
- Validation rules enforced
- Edge cases handled
- Exceptions thrown correctly

**What NOT to test:**
- Database queries (mocked repository)
- API layer (tested separately)
- UI behavior

**Test Type:** Unit tests (fast, mocked dependencies)

**Coverage Target:** [X]%

### Tier 5: Repositories

**What to test:**
- CRUD operations work
- Queries return correct data
- Filtering/sorting works
- Pagination works
- Tenant scoping enforced in queries

**What NOT to test:**
- Business logic (belongs in Services)
- API response format (belongs in API tier)

**Test Type:** Integration tests (repository + database)

**Coverage Target:** [X]%

---

## Test Execution & CI/CD

### Local Development

**Run unit tests:**
```bash
mvn test
```

**Run unit + integration tests:**
```bash
mvn verify -P integration-tests
```

**Run specific test:**
```bash
mvn test -Dtest=CategoryServiceTest
```

### Continuous Integration Pipeline

**Pre-commit (optional):**
- `mvn test` — Unit tests only (fast feedback)

**Pull Request:**
- `mvn verify -P integration-tests` — All tests
- Code coverage must be ≥ [X]%
- All tests must pass

**Before Merge to Main:**
- All tests passing
- Coverage targets met
- Security tests passing (cross-tenant, secrets)

**Production Deployment:**
- End-to-end tests run in staging
- Critical user paths verified
- Rollback plan in place

---

## Code Coverage

### Coverage Targets by Tier

| Tier | Type | Target |
|------|------|--------|
| Tier 1 (UI) | UI logic only | [X]% |
| Tier 2 (UIControllers) | State, delegation | [X]% |
| Tier 3 (API) | Endpoints, validation | [X]% |
| Tier 4 (Services) | Business logic | [X]% |
| Tier 5 (Repositories) | Data access | [X]% |

### Coverage Reporting

**View reports after testing:**
- Unit test coverage: `target/site/jacoco/index.html`
- Integration test coverage: `target/site/jacoco-it/index.html`

**Coverage gates:**
- Minimum [X]% for all code
- Minimum [X]% for Services (Tier 4)
- Minimum [X]% for Repositories (Tier 5)

---

## Common Testing Mistakes

**❌ Testing implementation details instead of behavior**
- Right: "Order status changed to PAID"
- Wrong: "Database column updated_at is recent"

**❌ Shared state between tests**
- Each test sets up its own data
- Previous test results don't affect current test

**❌ Mocking too much**
- Right: Mock external services (APIs, payments)
- Wrong: Mock your own services (defeats the test)

**❌ Ignoring flaky tests**
- Flaky tests indicate non-deterministic code
- Fix the root cause (time, random, external dependency)
- Don't just re-run and hope

**❌ Testing private methods directly**
- Test public behavior only
- If private method is complex, extract to separate class

---

## Test Tools & Framework

**Recommended:**
- **Unit testing:** [JUNIT/TESTNG] + [MOCKITO/EASYMOCK]
- **Integration testing:** [SPRINGTEST] + embedded database
- **Coverage:** [JACOCO]
- **Build:** Maven with Surefire + Failsafe
- **Assertions:** [ASSERTJ/HAMCREST]

---

## Related Documents

- [ARCHITECTURE_PRINCIPLES_TEMPLATE.md](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md) — Layering model
- [java-testing-maven-setup.md](../../common-AI-rules/java-testing-maven-setup.md) — Maven configuration
- [testing-rules.md](../../common-AI-rules/testing-rules.md) — Testing rules (determinism, isolation)

---

## Customization Checklist

- [ ] Define coverage targets for each tier
- [ ] Choose unit test framework (JUnit, TestNG)
- [ ] Choose mocking framework (Mockito, EasyMock)
- [ ] Configure Maven Surefire + Failsafe
- [ ] Set up JaCoCo coverage reporting
- [ ] Define critical user paths for E2E tests
- [ ] Set up test database (in-memory or Docker)
- [ ] Configure CI/CD pipeline
- [ ] Define test naming conventions
- [ ] Train team on testing standards
- [ ] Review this document quarterly

---

**Last Updated:** 2026-07-12  
**Maintained by:** QA & Development Team  
**Status:** Production-ready template
