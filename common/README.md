# Project Settings & Standards

Centralized repository for **development standards**, **architecture guidelines**, and **organizational documentation** used across all projects.

**Purpose:** Ensure consistency, quality, and best practices through documented, reusable standards and templates.

---

## Quick Navigation

- **Need code-level standards?** → See [Common Development Rules](#common-development-rules)
- **Need architecture templates?** → See [Common Architecture Templates](#common/architecture-templates)
- **Setting up a new project?** → See [Global Rules & Local Extensions](#global-rules--local-extensions)
- **Extending rules for your project?** → See [Creating Local Rules for Your Project](#creating-local-rules-for-your-project)
- **AI Assistants:** Read [Global Rules & Local Extensions](#global-rules--local-extensions) first
- **Getting started?** → See [How to Use This Repository](#how-to-use-this-repository)

---

## Repository Structure

```
project-settings/
├── README.md                          (This file - project overview)
├── common/AI-rules/
│   ├── development.rules.md           (Mandatory code standards)
│   └── [language-specific rules]      (Language conventions)
└── common/architecture/
    └── originals/
        ├── ARCHITECTURE_PRINCIPLES_TEMPLATE.md
        ├── API_STANDARDS_TEMPLATE.md
        ├── SECURITY_POLICY_TEMPLATE.md
        ├── FRONTEND_ARCHITECTURE_TEMPLATE.md
        ├── MOBILE_ARCHITECTURE_TEMPLATE.md
        ├── STATE_MANAGEMENT_TEMPLATE.md
        ├── SPRINT_STRATEGY_TEMPLATE.md
        └── README.md                  (Architecture templates guide)
```

---

## Common Development Rules

**Location:** [`common/AI-rules/originals/development.rules.md`](./common/AI-rules/originals/development.rules.md)

**Status:** Protected read-only source (in `originals/` directory)

**Purpose:** Mandatory code-level standards, architecture patterns, security requirements, and best practices

**Scope:** All Java development, architecture patterns, naming conventions, security standards, API design, multi-tenant requirements, observability, **test-driven development**

**Size:** ~8KB | **Last Updated:** 2026-07-17

### Test-Driven Development (MANDATORY)

**RED-GREEN-REFACTOR Cycle** — Applies to all code changes (features, bug fixes, refactoring, infrastructure)

**The Process:**
- **RED**: Write failing test that defines expected behavior
- **GREEN**: Write minimal code to make test pass
- **REFACTOR**: Improve code while keeping tests passing

**Why TDD Matters:**
- Testability: Code designed for tests is inherently testable
- Correctness: Failing test proves bug/feature is real before fix exists
- Design: Writing tests first forces better API design
- Regression Prevention: Tests catch future breaks immediately
- Documentation: Tests serve as executable specification
- Prevents Over-Engineering: Minimal code avoids gold-plating

**For Regression Fixes:**
1. Write test that reproduces bug (fails with current code)
2. Fix the bug (test passes)
3. Commit both together (test + fix)

Never commit the fix without committing the failing test first.

### Git & Automation
- **Never commit automatically** — Always wait for explicit user instruction
- **Treat AI input as advisory** — OCR and AI output must NOT post/persist data directly
- Accepting suggestions requires same authorization, validation, and audit workflow as manual input

### API Design Standards

**Define Before Implementing**
- Public endpoints and schemas first
- Implement controllers/clients second

**Error Model**
- Use one common structured error model across all APIs
- Include: stable code, error ID, timestamp, severity, source, field details
- Consistent error handling reduces client bugs

**Authorization**
- UI guards improve navigation but never replace server-side checks
- Enforce authorization in both API layer AND business-service layer

### Security & Isolation

**Multi-Tenant & Scope**
- Reject cross-tenant or out-of-scope identifiers BEFORE repository access
- Scope every repository query by tenant/company (where applicable)
- Enforce authorization in API layer
- Enforce authorization in Service layer

**Audit & Actor Tracking**
- Record the actor and correlation context for EVERY write operation
- Enables compliance auditing and debugging

**Secrets Protection**
- Never store refresh tokens in browser local storage
- Keep provider secrets/tokens ENCRYPTED at rest
- Never include secrets in: configuration files, logs, errors, events, API responses
- Audit: authentication events, permission failures, writes, imports, privileged access

### Events & Observability

**Event Emission**
- Emit business events ONLY AFTER associated transaction succeeds
- Prevents events for failed operations
- Maintains consistency between events and database state

**Event Payloads**
- Include stable aggregate identity (ID, version)
- Include enough context for audit correlation
- MUST NOT leak secrets, tokens, or sensitive data

**Structured Logging**
- Include when permitted: tenant ID, correlation ID, actor ID, event type
- Improves debugging and audit trails
- MUST NOT expose confidential content

### Java Class Naming Standards (MANDATORY)

**Rule 1: PascalCase (UpperCamelCase)**
- All class names PascalCase, no underscores
- ✅ `CategoryManagementPanel`, `DatabaseConnection`, `PasswordChangeHandler`
- ❌ `categoryPanel`, `category_panel`, `Category_Panel`

**Rule 2: Swing/AWT Component Type Suffixes**
- All Swing/AWT classes end with component type: `Frame`, `Dialog`, `Panel`, `Renderer`, `Model`, `Listener`
- ✅ `GSPosApplicationFrame extends JFrame`, `UserRegistrationDialog extends JDialog`
- ❌ `Application extends JFrame`, `UserRegistration extends JDialog`

**Rule 3: No J-Prefix Abbreviations**
- J-prefix only for JDK classes (JFrame, JPanel)
- Custom classes never use J-prefix
- ✅ `ApplicationConfigurationFrame`, `CategoryManagementPanel`
- ❌ `JFrmConfig`, `JPanelCategory` (as custom classes)

**Rule 4: Descriptive Component Names**
- Class names must clearly describe purpose AND type
- Avoid vague single-word names
- ✅ `MainApplicationFrame`, `CategoryManagementPanel`, `PasswordChangeHandler`
- ❌ `Main`, `Panel`, `Handler`, `View`, `Processor`

**Rule 5: No Generic Abbreviations**
- Use full descriptive terms, avoid vague abbreviations
- ✅ `DataUtility`, `ValidationHelper`, `DatabaseConnection`, `ApplicationConfiguration`
- ❌ `DataUtil`, `Helper`, `Utils`, `Mgr`, `Cfg`, `Conn`

**Rule 6: No Underscore Separators**
- CamelCase for all class names (public and inner)
- ✅ `FrameBase`, `ConfigurationViewPanel`, `SelectorRenderer`
- ❌ `Frame_Base`, `View_Configuration`, `Renderer_Selector`

**Rule 7: Inner Classes Follow Same Rules**
- Private static inner classes use same standards
- ✅ `private static class NullFormat extends Formats`
- ❌ `private static class FormatsNULL extends Formats`

**Rule 8: Correct Spelling**
- All class names must use correct spelling
- Common misspellings: Supplyer→Supplier, Reciept→Receipt, Verifiy→Verify
- ✅ `SupplierManagementPanel`, `ReceiptDocument`, `VerificationHandler`
- ❌ `SupplyerPanel`, `RecieptPrinter`, `VerifiyHandler`

**Rule 9: File Name Must Match Public Class**
- File name exactly matches public class (case-sensitive)
- Java Language Specification requirement
- ✅ File `GSPosApplicationFrame.java` contains `public class GSPosApplicationFrame`
- ❌ File `MainFrame.java` containing `public class GSPosApplicationFrame`
- Impact: Improves IDE support, code discoverability, developer experience

### Architecture & Layering Standards (MANDATORY)

**Three-Tier Architecture Pattern** — Strict separation of concerns
```
Tier 1 (UI):         Swing/AWT components (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation, state management)
Tier 3 (API):        REST @RestController (HTTP endpoints, validation)
Tier 4 (Business):   @Service classes (business logic, rules)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   MySQL, PostgreSQL, etc.
```

**Enforcement Rules:**
- ✅ Swing classes: UI concerns ONLY (layout, rendering, events)
- ✅ UIControllers: State management, delegate via HttpClient
- ✅ Services: Business logic and validation
- ✅ Repositories: Data persistence and queries
- ❌ NO database connections in UI layer
- ❌ NO service instantiation in UI layer
- ❌ NO Spring context access from Swing components

**No AppView in Swing Classes** (Mandatory)

Swing/AWT classes must NOT import `gs.Application.AppView`

❌ Violations:
```java
private AppView m_App;              // Never!
m_App.getRepository();               // Never!
m_App.getService();                  // Never!
// Direct repository access from UI  // Never!
```

✅ Correct Pattern:
```java
// Framework-agnostic UIController
public class CategoryEditorUIController implements EditorUIController {
    private HttpClient httpClient;  // Only HttpClient
    
    public CategoryEditorUIController(HttpClient httpClient) {
        this.httpClient = httpClient;
    }
    
    public void save(CategoryDto dto) {
        httpClient.post("/api/categories", dto, CategoryDto.class);
    }
}

// Swing class delegates to controller
public class CategoryEditorPanel extends JPanel {
    private CategoryEditorUIController controller;  // Injected
    
    private void onSave() {
        controller.save(getFormData());  // Delegates only
    }
}
```

**Framework-Agnostic UIControllers** (Mandatory)

UIControllers must have ZERO Swing/AWT imports and ZERO direct database access

- ✅ Imports: Only DTOs, interfaces, standard Java
- ✅ Communication: HttpClient for REST API calls
- ✅ State: In-memory fields, listeners for UI updates
- ❌ NO database code, NO Swing dependencies, NO direct service instantiation
- **Benefit:** Testable without UI framework or database

**Dependency Injection Over Direct Instantiation** (Mandatory)

Always use constructor-based dependency injection

❌ Never:
```java
new CategoryService();          // In Swing code
new RepositoryImpl();            // Anywhere
new DatabaseConnection();       // Outside @Configuration
```

✅ Correct:
```java
@Service
public class CategoryService {
    private final ICategoryRepository repository;  // Injected
    
    @Autowired
    public CategoryService(ICategoryRepository repository) {
        this.repository = repository;
    }
}

public class CategoryEditorUIController {
    private final HttpClient httpClient;  // Constructor injection
    
    public CategoryEditorUIController(HttpClient httpClient) {
        this.httpClient = httpClient;
    }
}
```

**Database Logic Isolation** (Mandatory)

ALL database operations (SQL, RecordSet, JDBC) in Repository layer ONLY

✅ Correct Structure:
```java
// Repository layer (database code here only)
@Repository
public class CategoryRepository implements ICategoryRepository {
    public List<CategoryDto> findAll() {
        // Database query here - this is the only place
    }
}

// Service layer (delegates to repository)
@Service
public class CategoryService {
    private final ICategoryRepository repository;
    
    public List<CategoryDto> getCategories() {
        return repository.findAll();  // Delegates
    }
}
```

❌ Violations:
```java
executeQuery();                 // In Swing panels - Never!
new PreparedStatement();        // In UI code - Never!
// ResultSet processing in dialogs - Never!
JDBC code outside Repository    // Never!
```

**Configuration Classes for Infrastructure** (Mandatory)

Database connections, Spring context, and bootstrap logic in @Configuration classes, NOT in UI

✅ Correct Pattern:
```java
@Configuration
public class DatabaseConfiguration {
    @Bean
    public DatabaseConnection databaseConnection() {
        // Database setup here - only place!
        return new DatabaseConnection(...);
    }
}

// UI layer - completely clean
public class PointOfSalesApplicationFrame extends JFrame {
    // Only UI concerns here
    // No database, no Spring context, no AppView
}
```

❌ Never:
```java
// In JFrame - Never!
// Database initialization in JFrame
// Spring ApplicationContext in Swing
// DatabaseConnection creation in panels
```

### Versioning & Change Discipline

**Semantic Versioning** for modules and contracts:
- **PATCH:** Clarification or compatible correction (no new runtime responsibility, schema migration, endpoint, event, or report contract change)
- **MINOR:** Backward-compatible behavior or contract addition (forward schema extension)
- **MAJOR:** Breaking compatibility, changed module responsibilities, substantial new solution surface

### Audience
- All engineers (developers follow these standards)
- Architects (enforce these standards)
- Code reviewers (validate compliance)
- QA teams (test against these patterns)

---

## Commands & Execution Rules

**Location:** [`common/AI-rules/originals/commands.rules.md`](./common/AI-rules/originals/commands.rules.md)

**Status:** Protected read-only source (in `originals/` directory)

**Purpose:** Operational safety, command logging, source protection, and credential handling for all automation and AI-assisted development

**Scope:** AI tool execution, command safety, logging practices, secrets protection, build/generated output management

**Size:** ~2KB | **Last Updated:** 2026-07-17

### Command Logging & History

**Before Starting Any Job**
- Log the user's command in `logs/commands.log.md` (local project)
- Begin work only AFTER logging

**History Management**
- Store command history in `logs/commands.log.md`
- Keep only latest 40 entries, newest first
- Renumber entries on each addition: newest = 40, oldest dropped at 1

**Entry Format**
```
N. YY-MM-DD HH:MM CEST (prefix): ~ command
```

- Always use Swedish timezone (CEST/CET)
- Preserve user's command text EXACTLY as written
- Use `time unavailable` only if system time cannot be obtained

**AI Prefixes**
- `(cc)` — Claude Code
- `(cx)` — Codex
- `(gc)` — Gemini Code

### Source Protection (MANDATORY)

**Treat `originals/` as Read-Only**

The `originals/` directory contains source material that must be protected

❌ Never:
```bash
edit originals/file.md           # Never edit
mv originals/file.md other.md    # Never move
rm originals/                    # Never delete
```

✅ Instead:
```bash
cp originals/file.md myproject/file.md  # Copy to destination
# Edit the copy, leave original untouched
```

**Enforcement Rules:**
- NEVER edit, rename, move, overwrite, or delete files in `originals/`
- NEVER include `originals/` in cleanup, generated-output, formatting, or migration commands
- Read supplied source material in place only
- Do NOT copy contents into production code/tests/fixtures unless explicitly required
- Before recursive delete: inspect resolved targets and prove NO target is `originals/` or ancestor

### Command Safety (MANDATORY)

**Repository Status Inspection**
- Inspect repository status BEFORE modifying files
- Preserve unrelated user changes
- Do NOT stage or commit unrelated work

**Scoped & Non-Destructive**
- Prefer scoped, non-interactive commands with explicit paths
- ✅ `git add src/MyFile.java` (specific file)
- ❌ `git add .` (too broad)

**Destructive Operations**
- Do NOT use destructive git commands unless explicitly requested
- Do NOT force push or rewrite shared history
- Do NOT use broad filesystem deletion
- Always ask for approval before irreversible operations

**What Requires Approval?**
- Installing software
- Accessing external systems
- Changing infrastructure
- Performing irreversible operations

### Secrets & Credentials (MANDATORY)

**Never Expose in Output**
- Do NOT expose credentials, tokens, personal data, provider payloads in:
  - Command output
  - Logs
  - Fixtures
  - Commits

**Keep Out of Shell History**
- Avoid putting secrets in command arguments
- Shell history can be accessed later
- Process listings can expose arguments

### Generated & Build Output

**Where to Generate**
- Generate output ONLY in module's configured build directory
- OR in documented temporary directory
- Use scratchpad for temporary/intermediate files

**Never Commit:**
- Build output (compiled classes, JAR files)
- Dependency caches (node_modules, Maven cache)
- Runtime secrets (API keys, passwords)
- Uploaded documents
- Database volumes
- Local identity-provider state

### Completion & Verification

**Evidence Required**
- Run smallest relevant verification first
- Follow with module or release gates when practical
- Report commands that failed or were not run
- Never claim a check passed without successful output

**Leave Working Tree Clear**
- Identify all files changed
- Identify all checks performed
- Flag any known risks or follow-up needed
- Document unexpected behavior

### Audience
- AI assistants (Claude Code, Codex, Gemini)
- DevOps and automation engineers
- Developers using automated tools
- CI/CD systems and scripts

---

## Testing Rules

**Location:** [`common/AI-rules/originals/testing-rules.md`](./common/AI-rules/originals/testing-rules.md)

**Status:** Protected read-only source (in `originals/` directory)

**Purpose:** Testing standards, quality gates, and Java-specific test configuration

**Scope:** Test organization, Java naming, Maven configuration, test quality, data safety, security isolation

**Size:** ~2KB | **Last Updated:** 2026-07-17

### Test File Organization

**Test Uploads**
- Test uploads placed in project's configured test-upload directory
- Tests that process uploaded files must fetch recursively from that directory
- Never hardcode individual file paths in test code
- Clear test-upload directory before each test or test suite

**Test Logging**
- Create new log file in `logs/` at test suite start
- Filename format: `Testing-YYMMDD-HH:MM CEST.log.md`
  - Example: `Testing-260618-15:50 CEST.log.md`
- List test suite name and one-line description for each test

### Java Test Naming (MANDATORY)

**Integration Tests**
- Suffix with `IT` exactly (e.g., `CashierControllerIT`)
- ✅ `CategoryRepositoryIT`, `UserServiceIT`, `PaymentControllerIT`
- ❌ Never use `ITTest` or `TestIT`

**Unit Tests**
- Must NOT use `IT` suffix
- ✅ `CategoryValidatorTest`, `PasswordEncoderTest`
- ❌ Never `CategoryValidatorIT`, `PasswordEncoderITTest`

**Why the distinction?** Allows Maven to run unit and integration tests separately with different plugins and coverage reports

### Maven Configuration (MANDATORY)

**Unit Tests with Coverage**
```xml
<!-- In default build (NOT in profile) -->
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>prepare-agent</goal>
        <goal>report</goal>
      </goals>
    </execution>
  </executions>
</plugin>

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration>
    <excludes>
      <exclude>**/*IT.java</exclude>  <!-- Exclude integration tests -->
    </excludes>
    <argLine>@{argLine}</argLine>  <!-- Late-binding for JaCoCo -->
  </configuration>
</plugin>
```

**Key Rules:**
- `mvn test` must NOT run integration tests (`*IT.java`)
- `mvn test` MUST produce JaCoCo code coverage report
- Add JaCoCo `prepare-agent` + `report` to default build (NOT in profile)
- Exclude `*IT.java` explicitly in Surefire plugin
- Use `@{argLine}` (late-binding) so JaCoCo agent is picked up automatically

**Integration Tests in Profile**
```xml
<profile>
  <id>integration-tests</id>
  <build>
    <plugins>
      <!-- JaCoCo for integration tests -->
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>prepare-agent-integration</goal>
              <goal>report-integration</goal>
            </goals>
            <configuration>
              <destFile>${project.build.directory}/jacoco-it.exec</destFile>
            </configuration>
          </execution>
        </executions>
      </plugin>
      
      <!-- Failsafe for integration tests -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
        <configuration>
          <includes>
            <include>**/*IT.java</include>
          </includes>
          <argLine>@{argLine}</argLine>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>integration-test</goal>
              <goal>verify</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</profile>
```

**Run Commands:**
```bash
mvn test                              # Unit tests + coverage
mvn test -P integration-tests         # All tests + separate coverage
mvn verify -P integration-tests       # Integration tests only
```

### Test Data Safety (MANDATORY)

**Never Use Production Data in Tests**
- ❌ Production credentials
- ❌ Personal data (real names, emails, phone numbers)
- ❌ Live accounts or real services
- ❌ Real documents or files

**Why?** Protects privacy, prevents accidental data corruption, allows tests to run in CI/CD safely

### Test Quality Standards (MANDATORY)

**Determinism, Independence, Repeatability**
- Tests MUST produce same result every run
- Tests must NOT depend on:
  - Execution order
  - Wall-clock timing (current time)
  - Public networks
  - Mutable external data
- Freeze/inject time, UUID generation, provider clients, exchange rates

✅ Correct:
```java
@Test
public void shouldCalculatePrice() {
    // Inject time instead of using current time
    Clock clock = Clock.fixed(Instant.parse("2026-07-12T10:00:00Z"), UTC);
    
    // Inject random instead of using system random
    PriceCalculator calc = new PriceCalculator(clock, mockRandom);
    
    BigDecimal price = calc.calculate(items);
    assertThat(price).isEqualTo(BigDecimal.valueOf(99.99));  // Deterministic
}
```

❌ Wrong:
```java
@Test
public void shouldCalculatePrice() {
    BigDecimal price = calc.calculate(items);
    // Result depends on current time, random seed, external exchange rates
    assertThat(price).isCloseTo(expectedPrice, within(1));  // Flaky!
}
```

**Business Outcomes Over Implementation Details**
- Assert observable business results, not internal state
- ✅ Assert: "Payment was processed and order status changed to PAID"
- ❌ Assert: "Internal counter incremented by 1"

**Regression Test First**
- Regression fix MUST be represented by test that fails for reported defect
- Test demonstrates the bug, then fix makes it pass
- Prevents regression from being re-introduced

**Never Weaken Tests for Green Build**
- Do NOT delete, skip, or broadly mock a failing test to get green build
- Failing tests indicate real problems
- Fix the bug, not the test
- ❌ `@Disabled`, `.skip()`, `when().thenReturn(anything)` as quick fix

### Isolation & Security Gates (MANDATORY)

**Cross-Tenant Denial Tests**
- Every data-access feature MUST include cross-tenant denial test
- Verify user from Tenant A cannot access data from Tenant B
- Test user scope restrictions (account, department, etc.)

✅ Correct:
```java
@Test
public void userFromTenantA_cannotAccessTenantB_data() {
    User userA = createUserInTenant(TENANT_A);
    
    CompanyData dataFromTenantB = getDataFromTenant(TENANT_B);
    
    // Should deny access
    assertThatThrownBy(() -> service.read(dataFromTenantB, userA))
        .isInstanceOf(AccessDeniedException.class);
}
```

**Secrets Not in Output**
- Assert secrets, tokens, confidential data do NOT appear in:
  - Logs
  - Audit events
  - Error messages
  - Snapshots
  - API responses

✅ Correct:
```java
@Test
public void apiResponse_doesNotExpose_secrets() {
    String apiKey = "super-secret-key-12345";
    
    ApiResponse response = callApiWithSecret(apiKey);
    
    assertThat(response.getBody()).doesNotContain(apiKey);
    assertThat(response.getBody()).doesNotContain("secret");
}
```

### Audience
- QA engineers (write and maintain tests)
- Developers (write unit and integration tests)
- Build engineers (configure Maven)
- Code reviewers (validate test quality and coverage)
- Security teams (validate isolation and secrets protection)

---

## Common Architecture Templates

**Location:** [`common/architecture/originals/`](./common/architecture/originals/)

**Purpose:** Reusable, customizable architecture documentation templates for system-wide patterns and organizational structure

**Status:** Production-ready templates (require customization for each project)

**Total Size:** ~209KB across 9 templates | **Last Updated:** 2026-07-12

### Template Files

#### Core Architecture Documents

| Template | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[ARCHITECTURE_PRINCIPLES_TEMPLATE.md](./common/architecture/originals/ARCHITECTURE_PRINCIPLES_TEMPLATE.md)** | Design philosophy, core principles, technology decisions, evolution roadmap | Everyone | ~23KB |
| **[API_STANDARDS_TEMPLATE.md](./common/architecture/originals/API_STANDARDS_TEMPLATE.md)** | REST API design, versioning, response formats, error handling | Backend & Frontend engineers | ~13KB |
| **[SECURITY_POLICY_TEMPLATE.md](./common/architecture/originals/SECURITY_POLICY_TEMPLATE.md)** | Security requirements, incident response, compliance, data protection | Security, DevOps, All engineers | ~16KB |

**ARCHITECTURE_PRINCIPLES_TEMPLATE.md**
- 8+ core architectural principles
- Architectural constraints and trade-offs
- Non-functional requirements (performance, reliability, scalability)
- Technology stack decisions with justification
- Code quality standards and testing strategies
- Decision framework (when to add complexity, refactor, optimize)
- Deployment philosophy (zero-downtime, migrations, monitoring)
- Anti-patterns and common mistakes
- Phase 1, 2, 3 evolution roadmap
- Accountability and review schedule
- 19-point customization checklist

**API_STANDARDS_TEMPLATE.md**
- URL structure and versioning strategy (URL path, header, or query param)
- HTTP method semantics (GET, POST, PUT, PATCH, DELETE)
- Request/response format standards
- HTTP status codes and error handling
- Authentication and rate limiting
- Data type and format standards
- Pagination patterns (offset, cursor, limits)
- API versioning and deprecation policy
- Error codes and documentation requirements
- 15-point pre-ship testing checklist

**SECURITY_POLICY_TEMPLATE.md**
- Executive summary of security commitments
- Security governance roles and incident response (P1-P4 classification)
- Authentication & authorization methods (JWT, OAuth, SAML, OIDC)
- Data protection and encryption standards
- Input validation and output encoding
- Common vulnerabilities (SQL injection, XSS, CSRF, OWASP Top 10)
- Secrets management and rotation
- Security testing (SAST, DAST, penetration testing)
- Access control models (RBAC, ABAC)
- Third-party and dependency security
- Compliance requirements (GDPR, HIPAA, SOC2)
- 15-point pre-deploy security checklist
- Monitoring and alerting thresholds

#### Platform-Specific Architecture

| Template | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[FRONTEND_ARCHITECTURE_TEMPLATE.md](./common/architecture/originals/FRONTEND_ARCHITECTURE_TEMPLATE.md)** | Frontend patterns, component architecture, state management | Frontend engineers | ~31KB |
| **[MOBILE_ARCHITECTURE_TEMPLATE.md](./common/architecture/originals/MOBILE_ARCHITECTURE_TEMPLATE.md)** | Mobile-first design, offline-first patterns, device constraints | Mobile engineers | ~24KB |
| **[STATE_MANAGEMENT_TEMPLATE.md](./common/architecture/originals/STATE_MANAGEMENT_TEMPLATE.md)** | State patterns (Redux, MobX, service-based, Context API) | Frontend engineers | ~24KB |

**FRONTEND_ARCHITECTURE_TEMPLATE.md**
- "Dumb Terminal" principle — frontend is presentation-only
- Smart vs. dumb component patterns
- Data flow architecture (one-way flow)
- State management strategies
- Service layer design for API communication
- Common frontend violations and fixes
- Testing strategies (unit, integration, E2E)
- Performance optimization techniques
- Anti-patterns to avoid

**MOBILE_ARCHITECTURE_TEMPLATE.md**
- Mobile-first design approach (constraint-driven)
- Platform support strategy (iOS/Android/cross-platform)
- Device constraints (screen size, battery, storage, connectivity)
- Offline-first architecture and sync patterns
- Real-time synchronization strategies
- Mobile API design (data minimization)
- Native integration points (camera, location, notifications)
- Performance optimization for mobile
- Testing strategies (device matrix, network conditions)
- App store deployment and distribution

**STATE_MANAGEMENT_TEMPLATE.md**
- Decision tree for choosing state management
- State classification (local, shared, global, transient)
- Service-based state patterns
- Redux/NgRx patterns and directory structure
- MobX reactive patterns
- Context API patterns (React)
- Vue Composition API patterns
- Immutability strategies
- Performance optimization (memoization, selectors)
- Testing state management (unit, integration)

#### Quality & Observability

| Template | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[TESTING_STANDARDS_TEMPLATE.md](./common/architecture/originals/TESTING_STANDARDS_TEMPLATE.md)** | Testing strategy, determinism, isolation, cross-tenant tests, coverage targets | QA engineers, developers | ~14KB |
| **[EVENTS_ARCHITECTURE_TEMPLATE.md](./common/architecture/originals/EVENTS_ARCHITECTURE_TEMPLATE.md)** | Event-driven patterns, emission rules, payload structure, consumers, idempotency | Backend engineers | ~15KB |

**TESTING_STANDARDS_TEMPLATE.md**
- Testing philosophy and tier-specific strategies
- Test classification (unit, integration, end-to-end)
- Test determinism, independence, repeatability requirements
- Regression testing workflow (test first, then fix)
- Cross-tenant denial testing (mandatory)
- Secrets protection in tests (mandatory)
- Code coverage targets by tier (UI, UIControllers, API, Services, Repositories)
- Test execution and CI/CD pipeline
- Maven/JaCoCo configuration integration

**EVENTS_ARCHITECTURE_TEMPLATE.md**
- Event emission rules (only after transaction succeeds)
- Event payload structure (mandatory fields, secrets protection)
- Event consumers and routing patterns
- Idempotency strategy (handle duplicate events safely)
- Event retention and replay policies
- Audit trail and compliance using events
- Monitoring and observability for events
- Testing event-driven code
- Common mistakes (orphaned events, secrets, non-idempotent)

#### Execution & Operations

| Template | Purpose | Audience | Size |
|----------|---------|----------|------|
| **[SPRINT_STRATEGY_TEMPLATE.md](./common/architecture/originals/SPRINT_STRATEGY_TEMPLATE.md)** | Sprint planning, task workflow, success metrics, risk management | Scrum masters, team leads | ~6KB |

**SPRINT_STRATEGY_TEMPLATE.md**
- Sprint cycle and duration
- Weekly schedule (planning, standups, reviews, retrospectives)
- Task management workflow (starting, progressing, completing tasks)
- Success metrics (velocity, code coverage, test pass rate)
- Red flags and blockers (stop & discuss criteria)
- Mitigation actions (pair programming, code review)
- Dependencies between sprints
- Team capacity planning
- Risk management strategy

#### Documentation

| File | Purpose |
|------|---------|
| **[README.md](./common/architecture/originals/README.md)** | Guide to using architecture templates |

### Template Usage Guidelines

**Quick Start (5 minutes):**
1. Review [Common Development Rules](./common/AI-rules/development.rules.md) for code enforcement
2. Copy template to your project's `/architecture` or `/docs` folder
3. Remove `_TEMPLATE` suffix from filename
4. Replace `[PLACEHOLDER]` values with your project details
5. Remove sections that don't apply
6. Get team review and approval

**Detailed Customization (30 minutes):**
1. Read template to understand structure
2. Identify all placeholders (marked with `[BRACKETS]`)
3. Gather project info (tech stack, team, targets, requirements)
4. Customize content for your domain
5. Add missing sections (domain-specific patterns, deployment, incident response)
6. Review and approve with team leads
7. Share with entire team

**Maintenance:**
- Quarterly architecture review
- Annual comprehensive review
- Immediate update for major decisions
- Archive old versions for reference

### Architecture ↔ Development Rules Integration

These templates work together with Common Development Rules:

| Architecture Decision | Development Rule | Enforcement |
|---|---|---|
| Three-tier separation | Tier definitions & isolation rules | Enforces layer separation in code |
| Framework-agnostic UI | No AppView in Swing classes | Prevents UI coupling to infrastructure |
| Business logic in backend | Database logic isolation to Repository | Prevents mixing concerns |
| Dependency injection | DI over direct instantiation | Ensures testability and modularity |
| REST API layer | @RestController, HTTP semantics | Contracts API layer behavior |
| No direct database access | Repositories only in data layer | Isolates data access |
| Configuration classes | Infrastructure bootstrap only | Separates concerns |

**How to use both:**
1. **Start with Architecture Templates** to understand *why* (strategic decisions)
2. **Review Development Rules** to see *how* (code-level enforcement)
3. **Implement using Code patterns** from the rules

---

## Global Rules & Local Extensions

**CRITICAL:** `/project-settings/common/` is PROTECTED and READ-ONLY. **NEVER EDIT global rules or design files.**

### Global vs Local Rules

**Global Rules** (this repository) — **READ-ONLY, PROTECTED**
- Apply to ALL projects across the organization
- Baseline standards everyone follows
- Language-agnostic patterns (architecture, security, testing)
- **FROZEN**: Changes only with technical leadership approval
- **PROTECTED**: No direct edits allowed
- Exception: Authorized organizational-wide standards updates only

**Local Rules** (in each project's `AI-rules/` directory) — **EDITABLE, PROJECT-SPECIFIC**
- Project-specific extensions that EXTEND global rules
- Domain-specific patterns (business logic, naming conventions, compliance)
- Override global rules only with explicit approval (documented with rationale)
- Change frequently (as project evolves)
- This is where you customize and extend standards for your project

### Example: Naming Standards

**Global Rule:**
```
Rule: PascalCase for all Java class names
Example: CategoryManagementPanel, SupplierService
```

**Local Extension** (in project's `AI-rules/naming-conventions.md`):
```
Rule: Prefix all domain-specific classes with domain name
Example: PaymentProcessorService, PaymentValidatorRule
(Extends global PascalCase rule with domain prefix requirement)
```

### Example: Security Rules

**Global Rule:**
```
Rule: Multi-tenant isolation required
Rule: No secrets in logs, events, or API responses
```

**Local Extension** (in project's `AI-rules/security-extensions.md`):
```
Rule: PCI-DSS compliance required (project processes payments)
Rule: All credit card data must use tokenization
Rule: Audit log all payment operations (regulatory requirement)
(Extends global multi-tenant rule with payment-specific requirements)
```

---

## Creating Local Rules for Your Project

### Step 1: Set Up Local Rules Directory

```bash
mkdir -p your-project/AI-rules/
```

### Step 2: Create Local Rules Files

Reference global rules, then extend them:

**File: `AI-rules/development-extensions.md`**
```markdown
# [PROJECT_NAME] Development Rules Extensions

Extends: [../path/to/global/development.rules.md](../path/to/global/development.rules.md)

## Domain-Specific Naming

**Rule:** All payment-related classes prefixed with `Payment`

Example: PaymentProcessor, PaymentValidator, PaymentGateway

**Why:** Makes payment domain explicit, prevents naming collisions

## Project-Specific Architecture

**Rule:** Event-driven order processing (extends global architecture)

Pattern:
1. Order API receives request → emits OrderCreated event
2. Inventory Service listens → checks stock
3. Fulfillment Service listens → prepares shipment
4. Payment Service listens → processes payment

(Applies to this project only; global architecture is still mandatory)
```

**File: `AI-rules/security-extensions.md`**
```markdown
# [PROJECT_NAME] Security Rules Extensions

Extends: [../path/to/global/security.rules.md](../path/to/global/security.rules.md)

## PCI-DSS Compliance (Payment Processing)

**Rule:** All credit card operations require PCI-DSS Level 1 compliance

- NO raw credit card numbers in database
- Use tokenization service for all cards
- Audit every payment operation
- Monthly security scan required

(Extends global secrets protection rule with payment-specific requirements)

## GDPR Compliance (User Data)

**Rule:** User personal data retention limited to 30 days after account deletion

- Logs: 7 days
- Backups: 30 days
- Audit trail: 1 year (anonymized)

(Extends global audit logging with GDPR timeline requirements)
```

**File: `AI-rules/testing-extensions.md`**
```markdown
# [PROJECT_NAME] Testing Rules Extensions

Extends: [../path/to/global/testing.rules.md](../path/to/global/testing.rules.md)

## Payment Processing Tests (MANDATORY)

Every payment operation requires:
- Unit test: business logic
- Integration test: payment gateway integration
- End-to-end test: full transaction workflow
- Security test: secrets not exposed

(Extends global test quality rule with payment-specific test matrix)

## Test Data

- Use [Payment Gateway Sandbox](https://sandbox.payment-api.com)
- Test cards: [See internal wiki](https://wiki.internal/test-cards)
- Never use production merchant credentials

## Coverage Targets

- Payment Service: 95% coverage (critical)
- Order Service: 85% coverage
- Inventory Service: 80% coverage

(Extends global coverage targets with project priorities)
```

### Step 3: Document for AI Assistants

**File: `AI-rules/README.md` (in your project)**
```markdown
# [PROJECT_NAME] Local Rules

This project uses:
1. **Global Rules** from [project-settings/common/AI-rules/originals/](../../project-settings/common/AI-rules/originals/)
2. **Local Extensions** in this directory

## For AI Assistants

When working on this project:
1. Read GLOBAL rules first: [development.rules.md](../../project-settings/common/AI-rules/originals/development.rules.md)
2. Read LOCAL extensions: [development-extensions.md](./development-extensions.md)
3. FOLLOW BOTH (local rules extend, not replace, global rules)

## Files in This Directory

- development-extensions.md — Domain naming, project-specific architecture
- security-extensions.md — PCI-DSS, GDPR, data retention
- testing-extensions.md — Payment testing matrix, coverage targets
- operations-extensions.md — Deployment, incident response
- README.md — This file

## When to Create New Local Rules

- Project-specific naming conventions
- Regulatory requirements (PCI-DSS, HIPAA, GDPR, SOC2)
- Business logic patterns (domain-driven events, workflows)
- Team preferences (code review process, testing approach)
- Performance targets (latency, throughput requirements)

## When NOT to Create New Local Rules (What NOT to Do)

- ❌ **NEVER edit files in `/project-settings/common/`** — Global rules are protected
- ❌ Don't override global architecture rules (three-tier layering)
- ❌ Don't weaken global security requirements
- ❌ Don't skip global testing standards
- ❌ Don't ignore global naming conventions
- ❌ Don't assume you can "fix" a global rule — go through proper change process
- ❌ Don't make "quick updates" to organizational standards

**If you find a problem with global rules:**
1. Document the issue with context
2. Propose a specific change
3. Get approval from technical leads
4. Update version history
5. Notify affected teams

## Requesting Exceptions

If you need to deviate from a global rule:
1. Document why (business requirement, technical constraint)
2. Get approval from tech lead and security team
3. Add exception to this README with rationale
4. Set expiration date for exception (quarterly review)
```

### Step 4: AI Assistants Read Both

**When AI (Claude Code) works on your project:**

```
AI reads global rules:
  ✅ development.rules.md (three-tier architecture, naming)
  ✅ testing-rules.md (determinism, isolation, regression)
  ✅ commands.rules.md (command safety, source protection)

AI reads local extensions:
  ✅ AI-rules/development-extensions.md (Payment prefix, event pattern)
  ✅ AI-rules/security-extensions.md (PCI-DSS, GDPR timeline)
  ✅ AI-rules/testing-extensions.md (Payment test matrix, 95% coverage)

AI implements following BOTH global + local rules
```

---

## How to Use This Repository

### For a New Project

1. **Review Development Standards**
   - Read [`common/AI-rules/originals/development.rules.md`](./common/AI-rules/originals/development.rules.md)
   - Understand mandatory architecture patterns
   - Note language-specific conventions

2. **Choose Architecture Templates**
   - Start with [ARCHITECTURE_PRINCIPLES_TEMPLATE](./common/architecture/originals/ARCHITECTURE_PRINCIPLES_TEMPLATE.md) for overall design
   - Add platform-specific templates (FRONTEND_ARCHITECTURE, MOBILE_ARCHITECTURE, etc.)
   - Include API_STANDARDS and SECURITY_POLICY

3. **Customize Templates**
   - Replace placeholders with project details
   - Adapt for your tech stack and team
   - Add domain-specific constraints
   - Define performance and security targets

4. **Implement & Enforce**
   - Structure code according to development rules
   - Use architecture templates as team guide
   - Reference during code review
   - Update as decisions change

### For Code Review

- **Check against Development Rules:**
  - Three-tier architecture respected?
  - UIControllers framework-agnostic?
  - Dependencies injected, not instantiated?
  - Database logic in Repository only?
  - Swing classes UI-only?

### For Architecture Decisions

- **Start with ARCHITECTURE_PRINCIPLES_TEMPLATE**
- **Reference related rules** in development standards
- **Document decision** and update templates
- **Share with team** and add to project guidelines

### For Security Review

- **Review against SECURITY_POLICY_TEMPLATE:**
  - Encryption standards met?
  - Authentication implemented correctly?
  - Input validation at boundaries?
  - No secrets in code?
  - Access controls documented?

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | 2026-07-17 | Moved AI-rules to `originals/` subdirectory for read-only protection; updated all cross-references |
| 1.0 | 2026-07-12 | Initial release: Common Development Rules (Java, architecture, security) + Architecture Templates (7 templates) |

---

## Quick Links

**Development Standards:**
- [Common Development Rules](./common/AI-rules/originals/development.rules.md) — Mandatory code standards (Java, architecture, naming, security)

**Architecture Templates:**
- [ARCHITECTURE_PRINCIPLES_TEMPLATE](./common/architecture/originals/ARCHITECTURE_PRINCIPLES_TEMPLATE.md) — System design (~23KB)
- [API_STANDARDS_TEMPLATE](./common/architecture/originals/API_STANDARDS_TEMPLATE.md) — REST API guidelines (~13KB)
- [SECURITY_POLICY_TEMPLATE](./common/architecture/originals/SECURITY_POLICY_TEMPLATE.md) — Security requirements (~16KB)
- [FRONTEND_ARCHITECTURE_TEMPLATE](./common/architecture/originals/FRONTEND_ARCHITECTURE_TEMPLATE.md) — Frontend patterns (~31KB)
- [MOBILE_ARCHITECTURE_TEMPLATE](./common/architecture/originals/MOBILE_ARCHITECTURE_TEMPLATE.md) — Mobile design (~24KB)
- [STATE_MANAGEMENT_TEMPLATE](./common/architecture/originals/STATE_MANAGEMENT_TEMPLATE.md) — State patterns (~24KB)
- [SPRINT_STRATEGY_TEMPLATE](./common/architecture/originals/SPRINT_STRATEGY_TEMPLATE.md) — Sprint execution (~6KB)
- [Architecture Templates Guide](./common/architecture/originals/README.md) — How to use templates

**External Resources:**
- [OWASP Top 10](https://owasp.org/Top10/) — Common security vulnerabilities
- [REST API Best Practices](https://restfulapi.net/) — RESTful design principles
- [The Twelve Factor App](https://12factor.net/) — Cloud-native architecture
- [Google Cloud Best Practices](https://cloud.google.com/docs) — Infrastructure patterns
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) — System design principles

---

## FAQ

**Q: Where do I start when joining a new project?**  
A: Read the Common Development Rules, review the project's customized architecture documents, then reference both during development and code review.

**Q: Can we customize the templates?**  
A: Yes! Templates are meant to be customized. Copy them to your project, replace placeholders, and adapt for your team's needs.

**Q: How often should we review these standards?**  
A: Quarterly for architecture, annually comprehensive. Update immediately when major decisions change.

**Q: Do development rules apply to all languages?**  
A: Development rules currently focus on Java and architecture patterns. Language-specific rules may be added to `common/AI-rules/originals/`.

**Q: What if we need to violate a development rule?**  
A: Document the exception with justification. Have it reviewed and approved by technical leads. Update the rules if it represents a legitimate pattern.

**Q: Are these templates production-ready?**  
A: No. Templates provide structure and guidance. Each project must customize extensively for their specific context.

---

## Contributing

**Improvements wanted!** These are living documents:

1. Use templates in your project
2. Document what works and what doesn't
3. Share improvements with other teams
4. Propose changes to this repository
5. Update documentation with lessons learned

---

## Support

**Questions about standards or templates?**
- Check the detailed documentation in each file
- Review examples in template files
- Ask your team lead or architect

**Found an issue or gap?**
- Document it with context
- Propose a specific change
- Update version history when approved

---

**Last Updated:** 2026-07-17  
**Maintained by:** Architecture Team  
**Status:** Production-ready for customization
