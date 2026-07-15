# Common AI Rules

Comprehensive AI-friendly guidelines for development standards, testing practices, and command execution safety.

**Purpose:** Enable consistent, safe, and high-quality development across all projects through documented rules that guide both human developers and AI assistants.

---

## Overview

This directory contains three complementary rule sets:

1. **Development Rules** — Code standards, architecture patterns, naming conventions, **document organization** (MANDATORY)
2. **Testing Rules** — Test structure, quality, data safety, Java-specific patterns
3. **Commands Rules** — AI command execution, logging, source protection, safety practices

Together, they define how code is written, tested, built, documented, and the operational practices for maintaining consistency and safety.

---

## Rules Files

### 1. Development Rules

**File:** [`development.rules.md`](./development.rules.md)

**Purpose:** Mandatory code-level standards and architectural patterns

**Audience:** All developers, architects, code reviewers, AI assistants

**Contents:**

#### Git & Automation Safety (MANDATORY)

**AI Commits** — CRITICAL SAFETY RULE
- **AI MUST NEVER commit unless explicitly asked to do so**
- Never commit as part of task completion
- Never assume changes should be committed
- Wait for explicit user instruction: "commit these changes", "make a commit", etc.
- User retains full control over when changes persist to git

**Commit Message Format** (MANDATORY)
- Short: ~10 rows maximum
- Copyable: Plain text format for direct pasting
- Scope: Summarize changes since last commit
- Structure: Subject + details + Co-Authored-By footer
- Delivery: Ready-to-use format, no markdown formatting of message text

#### Java Standards
- **Class Naming (Rule 8)**
  - PascalCase for all class names
  - Correct spelling required (SupplierManagementPanel, not SupplyerPanel)
  - Common misspellings to avoid listed

- **File Naming (Rule 9)**
  - File name must match public class name exactly
  - Java Language Specification requirement
  - Improves IDE support and code discoverability

#### Architecture and Layering Standards (MANDATORY)

**Three-Tier Architecture Pattern** — Enforces strict separation
```
Tier 1 (UI):         Swing/AWT (JFrame, JPanel, JDialog)
Tier 2 (UI Logic):   UIControllers (HttpClient delegation)
Tier 3 (API):        REST @RestController (HTTP endpoints)
Tier 4 (Business):   @Service classes (business logic)
Tier 5 (Data):       @Repository (persistence, queries)
Tier 6 (Database):   MySQL, PostgreSQL, etc.
```

**Key Rules:**
- **Swing Classes** — UI concerns ONLY, no business logic, no database access
- **UIControllers** — Framework-agnostic, delegate via HttpClient
- **Services** — Business logic and validation
- **Repositories** — ALL database access (SQL, RecordSet, JDBC)
- **NO database** in UI, UIController, or API layers
- **NO Spring context** access from Swing components
- **NO service instantiation** in UI code

**Framework-Agnostic UIControllers** (Mandatory)
- ZERO Swing/AWT imports
- ZERO direct database access
- Only DTOs, interfaces, standard Java
- Communication via HttpClient for REST calls
- Testable without UI framework or database

**No AppView in Swing Classes** (Mandatory)
- Never import `gs.Application.AppView`
- Prevents UI coupling to infrastructure
- Maintains testability

**Dependency Injection** (Mandatory)
- Constructor-based injection only
- Never `new ServiceClass()` in Swing code
- Enables testability and modularity

**Database Logic Isolation** (Mandatory)
- ALL database operations in Repository layer
- Services delegate to repositories
- No JDBC/SQL in UI, API, or Service layers

**Configuration Classes** (Mandatory)
- Database connections in @Configuration classes
- Spring context initialization separate
- Application bootstrap logic isolated from UI

#### Document Organization and Structure (MANDATORY)

All projects MUST follow strict folder organization for documentation:

**Architecture Documents**
- Location: `/architecture` folder in project root
- All system design, patterns, API design, database schema documentation
- Must have `/architecture/README.md` index

**Planning Documents**
- Location: `/planning` folder in project root
- All sprints, tickets, roadmap, requirements
- Sub-folders: `/planning/sprints/`, `/planning/tickets/`
- Must have `/planning/README.md` index

**Violations:**
- ❌ Architecture docs scattered in root or source folders
- ❌ Planning docs mixed with code directories
- ❌ Undocumented decisions

**Size:** ~28KB
**Last Updated:** 2026-07-13

---

### 2. Testing Rules

**File:** [`testing-rules.md`](./testing-rules.md)

**Purpose:** Testing standards, quality gates, Java-specific patterns

**Audience:** QA engineers, developers writing tests, code reviewers, CI/CD teams

**Contents:**

#### Test Organization

**Uploads**
- Test files placed in project's configured test-upload directory
- Recursively fetch uploaded files, never hardcode paths
- Clear test-upload before each test/suite

**Logging**
- Create log file in `logs/` at test suite start
- Filename format: `Testing-YYMMDD-HH:MM CEST.log.md`
- List test suite name and one-line description for each test

#### Java-Specific Rules

**Test Naming**
- Integration tests: suffix with `IT` (e.g., `CashierControllerIT`)
- Never use `ITTest` or `TestIT`
- Unit tests: NO `IT` suffix

**Maven Configuration**
- `mvn test` must NOT run integration tests (`*IT.java`)
- Exclude `*IT.java` explicitly in Surefire plugin
- `mvn test` MUST produce JaCoCo code coverage report
- Add JaCoCo `prepare-agent` + `report` to default build (not in profile)
- Integration tests in dedicated profile (e.g., `integration-tests`)
- Use Failsafe plugin for `*IT.java` files
- Collect JaCoCo coverage separately (`jacoco-it.exec`)
- Use `@{argLine}` (late-binding) for JaCoCo agent in both plugins

#### Test Quality Standards

**Safety**
- Never use production credentials, personal data, live accounts, real documents
- Clear test-upload directory before each test

**Determinism & Independence**
- Tests MUST be deterministic, independent, repeatable
- Do NOT depend on execution order, wall-clock timing, public networks
- Freeze/inject time, UUID generation, provider clients, exchange rates
- Prefer observable business outcomes over implementation details

**Regression Testing**
- Regression fix must first be represented by failing test
- Never weaken, delete, skip, or broadly mock failing gates to get green builds

**Isolation & Security**
- Every data-access feature MUST include cross-tenant denial tests
- Assert secrets, tokens, confidential payloads do NOT appear in:
  - Logs
  - Audit events
  - Errors
  - Snapshots
  - API responses

**Size:** ~2KB
**Last Updated:** 2026-07-12

---

### 3. Commands Rules

**File:** [`commands.rules.md`](./commands.rules.md)

**Purpose:** AI command execution, logging, source protection, and safety practices

**Audience:** AI assistants (Claude Code, other AI tools), developers, DevOps engineers

**Contents:**

#### Command Logging

**Before Starting Work**
- Log user's command in `logs/commands.log.md` (local project)
- Begin work only after logging command

**History Management**
- Store actual command history in `logs/commands.log.md`
- Keep only latest 40 entries, newest first
- Renumber entries: newest = 40, oldest dropped at 1
- Format: `N. YY-MM-DD HH:MM CEST (prefix): ~ command`
- Always use Swedish timezone (CEST/CET)

**AI Prefixes**
- `(cc)` — Claude Code
- `(cx)` — Codex
- `(gc)` — Gemini Code

#### Source Protection

**Originals/ Directory Rules** (Mandatory)
- Treat `originals/` directories as read-only source material
- NEVER edit, rename, move, overwrite, or delete files in `originals/`
- NEVER include `originals/` in cleanup, generated-output, formatting, or migration commands
- Read supplied source material in place
- Do NOT copy contents into production code/tests/documentation unless explicitly required
- Before recursive delete/cleanup: inspect targets and prove NO target is `originals/` or ancestor

#### Command Safety

**Repository & Filesystem**
- Inspect repository status before modifying files
- Preserve unrelated user changes
- Use scoped, non-interactive commands with explicit paths
- Do NOT use destructive git commands, force pushes, broad filesystem deletion unless explicitly requested
- Do NOT discard, rewrite, or stage unrelated changes

**Credentials & Secrets**
- NEVER expose credentials, tokens, personal data, provider payloads in:
  - Command output
  - Logs
  - Fixtures
  - Commits
- Keep secrets out of command arguments (avoid shell history/process listing exposure)

**Approval & Authorization**
- Ask for approval before:
  - Installing software
  - Accessing external systems
  - Changing infrastructure
  - Performing irreversible operations
- Exception: Authority already given by user

**Generated Output**
- Generate output ONLY in module's configured build directory or documented temp directory
- Do NOT commit build output, dependency caches, runtime secrets, uploaded documents, database volumes, local identity-provider state

#### Completion & Verification

**Evidence Required**
- Run smallest relevant verification first
- Follow with module or release gates when practical
- Report commands that failed or were not run
- Never claim check passed without successful output
- Leave working tree understandable:
  - Identify files changed
  - Identify checks performed
  - Flag any known risk or follow-up

**Size:** ~2KB
**Last Updated:** 2026-07-12

---

## How Rules Work Together

### Development → Testing → Commands

**Architecture Decisions** (Development)
↓
**Test Quality Standards** (Testing)
↓
**Safe Execution** (Commands)

### Example: Three-Tier Architecture

| Rule Set | Standard | Purpose |
|---|---|---|
| **Development** | UIControllers framework-agnostic, Repository-only database | Code structure |
| **Testing** | Test isolation, cross-tenant denial tests | Verify architecture works |
| **Commands** | No destructive operations in tests, protect source | Safe test execution |

### Example: Security

| Rule Set | Standard | Purpose |
|---|---|---|
| **Development** | Input validation at boundaries, Repository isolation | Prevent vulnerabilities |
| **Testing** | Assert secrets not in logs/responses, test data safety | Verify no leaks |
| **Commands** | No credentials in command output/logs | Protect credentials |

---

## Quick Reference

### For Code Implementation
→ Use **Development Rules**
- Architecture patterns (three-tier separation)
- Code naming and organization
- Dependency injection requirements

### For Test Writing
→ Use **Testing Rules**
- Test organization and naming
- Maven configuration (Surefire, Failsafe, JaCoCo)
- Test quality standards and safety
- Isolation and security gates

### For Command Execution (AI or automation)
→ Use **Commands Rules**
- Command logging and history
- Source protection (`originals/` directories)
- Safety practices (no destructive commands without approval)
- Credential protection

### For Code Review
**Check against all three:**
1. **Development Rules** — Architecture, naming, dependency injection
2. **Testing Rules** — Adequate test coverage, test quality, determinism
3. **Commands Rules** — No exposed credentials, tests are isolated

---

## Integration with Architecture Templates

These rules complement architecture templates in `common/architecture/`:

| Document | How It Uses Rules |
|---|---|
| ARCHITECTURE_PRINCIPLES_TEMPLATE | References three-tier separation from Development Rules |
| API_STANDARDS_TEMPLATE | Enforces REST patterns from Development Rules |
| SECURITY_POLICY_TEMPLATE | Aligns with security isolation rules |
| FRONTEND_ARCHITECTURE_TEMPLATE | Enforces framework-agnostic patterns |
| TESTING_* templates | Uses Testing Rules as baseline |

---

## Version History

| Version | Date | Files | Changes |
|---------|------|-------|---------|
| 1.0 | 2026-07-12 | development.rules.md, testing-rules.md, commands.rules.md | Initial release with Java standards, architecture patterns, testing guidelines, and command safety rules |

---

## FAQ

**Q: Do all three rule files apply to my project?**  
A: Yes. Development Rules define code structure, Testing Rules ensure quality, Commands Rules ensure safe execution and logging.

**Q: Can I customize these rules?**  
A: Yes, but changes should be approved by team leads and documented. These are baseline standards that can be extended for domain-specific needs.

**Q: What happens if I violate a rule?**  
A: Document the exception with justification. Have it reviewed by technical leads. Update the rules if it represents a legitimate pattern.

**Q: Do these rules apply to all programming languages?**  
A: Some rules (architecture, commands, general testing) are language-agnostic. Others (Java naming, Maven setup) are Java-specific. Language-specific extensions should be added separately.

**Q: How do testing rules relate to code coverage?**  
A: Testing Rules require JaCoCo coverage reports in every build (`mvn test`). Separate integration test profiles allow measuring different coverage metrics. Code review gates should require minimum coverage thresholds.

**Q: What if a command rule conflicts with project requirements?**  
A: Document the conflict and get approval from both technical and operations leads before proceeding. Update the command rules if the exception represents a legitimate pattern.

**Q: Who enforces these rules?**  
A: Code reviewers enforce Development Rules. QA and CI/CD enforce Testing Rules. DevOps and AI assistants follow Commands Rules during execution.

---

## Support

**Questions about a specific rule?**
- Read the detailed section in the relevant rules file
- Ask your team lead or architect
- Check the current project's local rules for extensions

**Found an issue or gap?**
- Document it with context and affected project
- Propose a specific change
- Get technical lead approval
- Update version history

**Need language-specific rules?**
- Check if extensions exist in your project's local `AI-rules/` directory
- Propose new language-specific rule files following this structure

---

**Last Updated:** 2026-07-12  
**Maintained by:** Architecture & Development Team  
**Status:** Production-ready baseline for all projects
