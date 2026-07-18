# Architecture and Layering Standards - MANDATORY

All applications must follow strict six-tier separation: **Swing/UI → UIControllers → REST API → Services → Repositories → Database**

## Three-Tier Architecture Pattern

```
Tier 1 (UI):         Swing/AWT components (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation, state management)
Tier 3 (API):        REST @RestController (HTTP endpoints, validation)
Tier 4 (Business):   @Service classes (business logic, rules)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   MySQL, PostgreSQL, etc.
```

**Core Rule**: Each tier handles ONE responsibility. No tier crosses into another's domain.

## Tier 1: UI Layer (Swing/AWT)

- ✅ Layout, rendering, event handling ONLY
- ❌ NO business logic, NO database access, NO service instantiation
- ❌ NEVER import `gs.Application.AppView`
- ❌ NEVER import Spring or Repository classes
- Delegates to UIControllers only

**Example**:
```java
public class CategoryEditorPanel extends JPanel {
    private CategoryEditorUIController controller;  // Only dependency
    
    private void onSave() {
        controller.save(getFormData());  // Delegates only
    }
}
```

## Tier 2: UIControllers (State Management)

- ✅ Framework-agnostic (ZERO Swing/AWT imports, ZERO database imports)
- ✅ Communicate via HttpClient for REST API calls
- ✅ In-memory state and UI listeners
- ❌ NO database code, NO service instantiation, NO Spring context access
- Testable without UI framework or database

**Example**:
```java
public class CategoryEditorUIController implements EditorUIController {
    private final HttpClient httpClient;  // Only dependency
    
    public CategoryEditorUIController(HttpClient httpClient) {
        this.httpClient = httpClient;
    }
    
    public void save(CategoryDto dto) {
        httpClient.post("/api/categories", dto, CategoryDto.class);
    }
}
```

## Tier 3: REST API (@RestController)

- ✅ HTTP endpoints, request/response handling
- ✅ API-level validation
- ✅ Delegates to Services
- ❌ NO database access, NO direct Repository instantiation

## Tier 4: Business Services (@Service)

- ✅ Business logic, validation, rules
- ✅ Delegates to Repositories
- ❌ NO database code, NO direct Repository creation
- Use dependency injection for repository access

**Example**:
```java
@Service
public class CategoryService {
    private final ICategoryRepository repository;  // Injected
    
    @Autowired
    public CategoryService(ICategoryRepository repository) {
        this.repository = repository;
    }
    
    public List<CategoryDto> getCategories() {
        return repository.findAll();  // Delegates to repository
    }
}
```

## Tier 5: Repositories (@Repository)

- ✅ ALL database operations (SQL, RecordSet, JDBC) ONLY here
- ✅ Queries, persistence, data access
- ❌ NO business logic, NO service logic
- NO direct instantiation in other layers

**Example**:
```java
@Repository
public class CategoryRepository implements ICategoryRepository {
    public List<CategoryDto> findAll() {
        // Database query here - only place for SQL/JDBC
    }
}
```

## Tier 6: Database

- ✅ Raw data storage (MySQL, PostgreSQL, etc.)
- Only accessed through Repository layer

## Dependency Injection (MANDATORY)

- **Always use constructor-based injection**
- Never use `new ServiceClass()`, `new RepositoryImpl()`, or `new DatabaseConnection()`
- Exception: `new DatabaseConnection()` only in @Configuration classes

**Example**:
```java
@Configuration
public class DatabaseConfiguration {
    @Bean
    public DatabaseConnection databaseConnection() {
        return new DatabaseConnection(...);  // Only place for 'new'
    }
}
```

## Continuous Integration & Quality Gates (MANDATORY)

### External/Global Jenkins

All projects use a centralized, organization-wide Jenkins instance for CI/CD pipelines.

**Requirements:**
- **MANDATORY**: All projects configure CI/CD pipeline in global Jenkins
- **MANDATORY**: Build configuration stored in project repository (`Jenkinsfile` or equivalent)
- **MANDATORY**: Pipeline runs on every commit (unit tests, layer tests, integration tests)
- **MANDATORY**: Builds must pass before merge to main branch
- **NEVER**: Create project-specific Jenkins instances; use global instance
- **Why**: Centralized infrastructure, consistent build process, auditable deployment

### SonarQube Code Quality

All projects use a centralized SonarQube instance for continuous code quality analysis.

**Requirements:**
- **MANDATORY**: Every commit scanned by SonarQube
- **MANDATORY**: Code quality gate must pass before merge
- **MANDATORY**: Coverage reports from all test profiles sent to SonarQube:
  - Unit test coverage (`*Test`)
  - Layer test coverage (`*LT`)
  - Integration test coverage (`*IT`)
- **MANDATORY**: Security issues must be resolved before merge
- **NEVER**: Disable quality gates or merge with failed analysis
- **Why**: Early detection of bugs, security issues, and technical debt; enforces code standards

**Coverage Targets (Configurable per Project):**
- Minimum line coverage: 60%
- Minimum branch coverage: 50%
- Maximum technical debt ratio: 5%
- Zero security issues: BLOCKER

### CI/CD Pipeline Flow

```
Commit → Jenkins Build
         ↓
    Run all test profiles (unit, layer, integration)
         ↓
    Generate coverage reports
         ↓
    SonarQube scan (code quality, security, coverage)
         ↓
    Quality gate pass/fail
         ↓
    If pass: approve merge
    If fail: block merge, fix issues
```

---

## Violations (Never Do These)

- ❌ Database code in UI layer
- ❌ Service instantiation in Swing code
- ❌ AppView import in Swing classes
- ❌ Direct database access from Service or API layers
- ❌ Spring context access from Swing components
- ❌ PreparedStatement in UI code
- ❌ Database initialization in JFrame
- ❌ Merging code with failed SonarQube quality gate
- ❌ Creating project-specific Jenkins (use global instance)

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
