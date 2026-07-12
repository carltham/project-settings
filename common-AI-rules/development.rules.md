# Common Development Rules

## Git & Automation

- Do not commit anything automatically. Always wait for an explicit commit instruction from the user.
- AI and OCR output is untrusted advisory data. It MAY propose or pre-fill a command but MUST NOT post or persist data directly.
- Accepting a suggestion MUST enter the same authorization, validation, and audit workflow as manually entered input.

## Java Class Naming Standards (MANDATORY)

### Rule 1: PascalCase (UpperCamelCase) - MANDATORY
All Java class names must use PascalCase with no underscores or lowercase starts.
- Correct: `CategoryManagementPanel`, `DatabaseConnection`, `PasswordChangeHandler`
- Never: `categoryPanel`, `category_panel`, `Category_Panel`, `categoryPANEL`

### Rule 2: Swing/AWT Component Type Suffixes - MANDATORY
All Swing/AWT component classes must end with their component type: `Frame`, `Dialog`, `Panel`, `Renderer`, `Model`, `Listener`.
- Correct: `class GSPosApplicationFrame extends JFrame`, `class UserRegistrationDialog extends JDialog`, `class CategoryManagementPanel extends JPanel`
- Never: `class Application extends JFrame`, `class UserRegistration extends JDialog`, `class CategoryManagement extends JPanel`

### Rule 3: No J-Prefix Abbreviations - MANDATORY
Do not use legacy J-prefix Swing convention on custom classes (applies only to JDK classes like JFrame, JPanel, etc.).
- Correct: `ApplicationConfigurationFrame`, `CategoryManagementPanel`, `NumberKeypadPanel`
- Never: `JFrmConfig`, `JPanelCategory`, `JNumberKeys` (as custom class names)

### Rule 4: Descriptive Component Names - MANDATORY
Class names must clearly describe purpose and type. Avoid vague single-word names.
- Correct: `MainApplicationFrame`, `CategoryManagementPanel`, `PasswordChangeHandler`, `ContentViewPanel`
- Never: `Main`, `Panel`, `Handler`, `View`, `Processor` (without context)

### Rule 5: No Generic Abbreviations - MANDATORY
Avoid vague abbreviations that obscure purpose. Use full descriptive terms.
- Correct: `DataUtility`, `ValidationHelper`, `ImageUtility`, `DatabaseManager`, `ApplicationConfiguration`, `DatabaseConnection`
- Never: `DataUtil`, `Helper`, `Utils`, `Mgr`, `Cfg`, `Conn`

### Rule 6: No Underscore Separators in Class Names - MANDATORY
Use CamelCase for class names, not underscore_case. Applies to both public and inner classes.
- Correct: `FrameBase`, `ConfigurationViewPanel`, `SelectorRenderer`, `NullFormat`
- Never: `Frame_Base`, `View_Configuration`, `Renderer_Selector`, `Formats_NULL`

### Rule 7: Inner Classes Follow Same Rules - MANDATORY
Private static inner classes must follow the same naming standards as public classes.
- Correct: `private static class NullFormat extends Formats`, `private static class IntegerFormat extends Formats`
- Never: `private static class FormatsNULL extends Formats`, `private static class FormatsINT extends Formats`

### Rule 8: Correct Spelling - MANDATORY
All class names must use correct spelling. Common misspellings to avoid: "Supplyer"→"Supplier", "Reciept"→"Receipt", "Verifiy"→"Verify".
- Correct: `SupplierManagementPanel`, `ReceiptDocument`, `VerificationHandler`
- Never: `SupplyerPanel`, `RecieptPrinter`, `VerifiyHandler`

### Rule 9: File Name Must Match Public Class Name - MANDATORY
Each Java file must be named exactly after its public class (case-sensitive). This is a Java Language Specification requirement.
- Correct: File `GSPosApplicationFrame.java` contains `public class GSPosApplicationFrame`
- Never: File `MainFrame.java` containing `public class GSPosApplicationFrame`
- Impact: Improves IDE support, code discoverability, and developer experience

## Architecture and Layering Standards - MANDATORY

### Three-Tier Architecture Pattern

All applications must follow strict six-tier separation: **Swing/UI → UIControllers → REST API → Services → Repositories → Database**

```
Tier 1 (UI):         Swing/AWT components (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation, state management)
Tier 3 (API):        REST @RestController (HTTP endpoints, validation)
Tier 4 (Business):   @Service classes (business logic, rules)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   MySQL, PostgreSQL, etc.
```

**Core Rule**: Each tier handles ONE responsibility. No tier crosses into another's domain.

### Tier 1: UI Layer (Swing/AWT)
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

### Tier 2: UIControllers (State Management)
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

### Tier 3: REST API (@RestController)
- ✅ HTTP endpoints, request/response handling
- ✅ API-level validation
- ✅ Delegates to Services
- ❌ NO database access, NO direct Repository instantiation

### Tier 4: Business Services (@Service)
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

### Tier 5: Repositories (@Repository)
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

### Tier 6: Database
- ✅ Raw data storage (MySQL, PostgreSQL, etc.)
- Only accessed through Repository layer

### Dependency Injection (MANDATORY)
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

### Violations (Never Do These)
- ❌ Database code in UI layer
- ❌ Service instantiation in Swing code
- ❌ AppView import in Swing classes
- ❌ Direct database access from Service or API layers
- ❌ Spring context access from Swing components
- ❌ PreparedStatement in UI code
- ❌ Database initialization in JFrame

## API Design

- Define public endpoints and schemas before implementing controllers or clients.
- Use one common structured error model with a stable code, error ID, timestamp, severity, source, and field details.
- UI guards improve navigation but never replace server-side authorization.

## Security and Isolation

- Enforce authorization in both API and business-service layers.
- Reject cross-tenant or out-of-scope identifiers before repository access, and scope every repository query by tenant/company as applicable.
- Record the actor and correlation context for every write.
- Never store refresh tokens in browser local storage.
- Keep provider secrets and tokens encrypted at rest and out of configuration files, logs, errors, events, and API responses.
- Audit authentication events, permission failures, writes, imports, and privileged access.

## Events and Observability

- Emit business events only after the associated transaction succeeds.
- Event payloads MUST include stable aggregate identity and enough context for audit correlation without leaking secrets.
- Structured logs SHOULD include tenant ID, correlation ID, actor ID, and event type when permitted, but MUST NOT expose confidential content.

## Versioning and Change Discipline

- `PATCH`: clarification or compatible correction with no new runtime responsibility, schema migration, endpoint, event, or report contract.
- `MINOR`: backward-compatible behavior or contract addition, including a forward schema extension.
- `MAJOR`: breaking compatibility, changed module responsibilities, or a substantial new solution surface.
