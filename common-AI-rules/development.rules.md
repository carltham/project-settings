# Common Development Rules

## Git

- Do not commit anything automatically. Always wait for an explicit commit instruction from the user.

## AI and external input

- AI and OCR output is untrusted advisory data. It MAY propose or pre-fill a command but MUST NOT post or persist data directly.
- Accepting a suggestion MUST enter the same authorization, validation, and audit workflow as manually entered input.

## API design

- Define public endpoints and schemas before implementing controllers or clients.
- Use one common structured error model with a stable code, error ID, timestamp, severity, source, and field details.
- UI guards improve navigation but never replace server-side authorization.

## Security and isolation

- Enforce authorization in both API and business-service layers.
- Reject cross-tenant or out-of-scope identifiers before repository access, and scope every repository query by tenant/company as applicable.
- Record the actor and correlation context for every write.
- Never store refresh tokens in browser local storage.
- Keep provider secrets and tokens encrypted at rest and out of configuration files, logs, errors, events, and API responses.
- Audit authentication events, permission failures, writes, imports, and privileged access.

## Events and observability

- Emit business events only after the associated transaction succeeds.
- Event payloads MUST include stable aggregate identity and enough context for audit correlation without leaking secrets.
- Structured logs SHOULD include tenant ID, correlation ID, actor ID, and event type when permitted, but MUST NOT expose confidential content.

## Java Class Naming Standards

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

## Architecture and Layering Standards

### Three-Tier Architecture Pattern - MANDATORY
All applications must follow strict three-tier separation: **Swing/UI → UIControllers → REST API → Services → Repositories → Database**

```
Tier 1 (UI):         Swing/AWT components (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation, state management)
Tier 3 (API):        REST @RestController (HTTP endpoints, validation)
Tier 4 (Business):   @Service classes (business logic, rules)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   MySQL, PostgreSQL, etc.
```

**Rules**:
- Swing classes ONLY handle UI concerns (layout, rendering, events)
- UIControllers handle state management and delegate via HttpClient
- Services handle business logic and validation
- Repositories handle data persistence
- NO database connections in UI layer
- NO service instantiation in UI layer
- NO Spring context access from Swing components

### Rule: No AppView in Swing Classes - MANDATORY
Swing/AWT classes must NOT import or use `gs.Application.AppView`. This couples UI to application infrastructure and breaks testability.

**Correct Pattern**:
```java
// ✅ CORRECT: Framework-agnostic UIController
public class CategoryEditorUIController implements EditorUIController {
    private HttpClient httpClient;
    
    public CategoryEditorUIController(HttpClient httpClient) {
        this.httpClient = httpClient;
    }
    
    public void save(CategoryDto dto) {
        httpClient.post("/api/categories", dto, CategoryDto.class);
    }
}

// ✅ CORRECT: Swing class delegates to controller
public class CategoryEditorPanel extends JPanel {
    private CategoryEditorUIController controller;
    
    public CategoryEditorPanel(CategoryEditorUIController controller) {
        this.controller = controller;
    }
    
    private void onSave() {
        controller.save(getFormData());  // Delegates to controller
    }
}
```

**Violations**:
- Never: `private AppView m_App;` in Swing classes
- Never: `m_App.getRepository()` in panels
- Never: `m_App.getService()` in dialogs
- Never: Direct repository access from UI layer

### Rule: Framework-Agnostic UIControllers - MANDATORY
UIControllers must have ZERO Swing/AWT imports and ZERO direct database access.

**Correct**:
- UIController imports: Only DTOs, interfaces, standard Java
- Communication: HttpClient for REST API calls
- State: In-memory fields, listeners for UI updates
- NO database code
- NO Swing dependencies
- NO direct service instantiation

**Testing Benefit**: UIControllers can be tested without UI framework or database

### Rule: Dependency Injection Over Direct Instantiation - MANDATORY
Use constructor-based dependency injection. Never use `new ServiceClass()`.

**Correct Pattern**:
```java
// ✅ CORRECT: Constructor injection
@Service
public class CategoryService {
    private final ICategoryRepository repository;
    
    @Autowired
    public CategoryService(ICategoryRepository repository) {
        this.repository = repository;
    }
}

// ✅ CORRECT: Interface dependency
public class CategoryEditorUIController {
    private final HttpClient httpClient;  // Injected
    
    public CategoryEditorUIController(HttpClient httpClient) {
        this.httpClient = httpClient;
    }
}
```

**Violations**:
- Never: `new CategoryService()` in Swing code
- Never: `new RepositoryImpl()` anywhere
- Never: `new DatabaseConnection()` outside config

### Rule: Database Logic Isolation - MANDATORY
All database operations (SQL, RecordSet, JDBC) must be in Repository layer only.

**Correct**:
```java
// ✅ IN REPOSITORY LAYER
@Repository
public class CategoryRepository implements ICategoryRepository {
    public List<CategoryDto> findAll() {
        // Database query here
    }
}

// ✅ IN SERVICE LAYER
@Service
public class CategoryService {
    public List<CategoryDto> getCategories() {
        return repository.findAll();  // Delegates to repository
    }
}
```

**Violations**:
- Never: `executeQuery()` in Swing panels
- Never: `PreparedStatement` in UI code
- Never: `ResultSet` processing in dialogs
- Never: JDBC code outside Repository

### Rule: Configuration Classes for Infrastructure - MANDATORY
Database connections, Spring context, and application bootstrap logic belong in configuration classes, NOT in UI.

**Correct Pattern**:
```java
// ✅ CONFIGURATION CLASS
@Configuration
public class DatabaseConfiguration {
    @Bean
    public DatabaseConnection databaseConnection() {
        // Database setup here
        return new DatabaseConnection(...);
    }
}

// ✅ UI LAYER (No database concerns)
public class PointOfSalesApplicationFrame extends JFrame {
    // Only UI concerns here
    // No database, no Spring context, no AppView
}
```

**Violations**:
- Never: Database initialization in JFrame
- Never: Spring ApplicationContext in Swing
- Never: DatabaseConnection creation in panels

## Versioning and change discipline

- `PATCH`: clarification or compatible correction with no new runtime responsibility, schema migration, endpoint, event, or report contract.
- `MINOR`: backward-compatible behavior or contract addition, including a forward schema extension.
- `MAJOR`: breaking compatibility, changed module responsibilities, or a substantial new solution surface.
