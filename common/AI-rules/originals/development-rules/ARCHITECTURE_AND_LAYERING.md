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

## Violations (Never Do These)

- ❌ Database code in UI layer
- ❌ Service instantiation in Swing code
- ❌ AppView import in Swing classes
- ❌ Direct database access from Service or API layers
- ❌ Spring context access from Swing components
- ❌ PreparedStatement in UI code
- ❌ Database initialization in JFrame

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
