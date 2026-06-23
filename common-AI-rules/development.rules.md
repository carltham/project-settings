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

## Versioning and change discipline

- `PATCH`: clarification or compatible correction with no new runtime responsibility, schema migration, endpoint, event, or report contract.
- `MINOR`: backward-compatible behavior or contract addition, including a forward schema extension.
- `MAJOR`: breaking compatibility, changed module responsibilities, or a substantial new solution surface.
