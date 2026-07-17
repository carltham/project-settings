# Java Class Naming Standards - MANDATORY

**Note:** Java-specific standards. See `./language-specific/` for other languages (C++, TypeScript/Node, Angular).

## Rule 1: PascalCase (UpperCamelCase) - MANDATORY

All Java class names must use PascalCase with no underscores or lowercase starts.

- ✅ Correct: `CategoryManagementPanel`, `DatabaseConnection`, `PasswordChangeHandler`
- ❌ Never: `categoryPanel`, `category_panel`, `Category_Panel`, `categoryPANEL`

## Rule 2: Swing/AWT Component Type Suffixes - MANDATORY

All Swing/AWT component classes must end with their component type: `Frame`, `Dialog`, `Panel`, `Renderer`, `Model`, `Listener`.

- ✅ Correct: `class GSPosApplicationFrame extends JFrame`, `class UserRegistrationDialog extends JDialog`, `class CategoryManagementPanel extends JPanel`
- ❌ Never: `class Application extends JFrame`, `class UserRegistration extends JDialog`, `class CategoryManagement extends JPanel`

## Rule 3: No J-Prefix Abbreviations - MANDATORY

Do not use legacy J-prefix Swing convention on custom classes (applies only to JDK classes like JFrame, JPanel, etc.).

- ✅ Correct: `ApplicationConfigurationFrame`, `CategoryManagementPanel`, `NumberKeypadPanel`
- ❌ Never: `JFrmConfig`, `JPanelCategory`, `JNumberKeys` (as custom class names)

## Rule 4: Descriptive Component Names - MANDATORY

Class names must clearly describe purpose and type. Avoid vague single-word names.

- ✅ Correct: `MainApplicationFrame`, `CategoryManagementPanel`, `PasswordChangeHandler`, `ContentViewPanel`
- ❌ Never: `Main`, `Panel`, `Handler`, `View`, `Processor` (without context)

## Rule 5: No Generic Abbreviations - MANDATORY

Avoid vague abbreviations that obscure purpose. Use full descriptive terms.

- ✅ Correct: `DataUtility`, `ValidationHelper`, `ImageUtility`, `DatabaseManager`, `ApplicationConfiguration`, `DatabaseConnection`
- ❌ Never: `DataUtil`, `Helper`, `Utils`, `Mgr`, `Cfg`, `Conn`

## Rule 6: No Underscore Separators in Class Names - MANDATORY

Use CamelCase for class names, not underscore_case. Applies to both public and inner classes.

- ✅ Correct: `FrameBase`, `ConfigurationViewPanel`, `SelectorRenderer`, `NullFormat`
- ❌ Never: `Frame_Base`, `View_Configuration`, `Renderer_Selector`, `Formats_NULL`

## Rule 7: Inner Classes Follow Same Rules - MANDATORY

Private static inner classes must follow the same naming standards as public classes.

- ✅ Correct: `private static class NullFormat extends Formats`, `private static class IntegerFormat extends Formats`
- ❌ Never: `private static class FormatsNULL extends Formats`, `private static class FormatsINT extends Formats`

## Rule 8: Correct Spelling - MANDATORY

All class names must use correct spelling. Common misspellings to avoid:
- "Supplyer" → "Supplier"
- "Reciept" → "Receipt"
- "Verifiy" → "Verify"

- ✅ Correct: `SupplierManagementPanel`, `ReceiptDocument`, `VerificationHandler`
- ❌ Never: `SupplyerPanel`, `RecieptPrinter`, `VerifiyHandler`

## Rule 9: File Name Must Match Public Class Name - MANDATORY

Each Java file must be named exactly after its public class (case-sensitive). This is a Java Language Specification requirement.

- ✅ Correct: File `GSPosApplicationFrame.java` contains `public class GSPosApplicationFrame`
- ❌ Never: File `MainFrame.java` containing `public class GSPosApplicationFrame`

**Impact:** Improves IDE support, code discoverability, and developer experience

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard (Java-specific)
