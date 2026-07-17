# Development Rules - Non-AI Specific

Comprehensive development standards for code quality, architecture, and organizational practices.

**Status:** Protected read-only baseline standards  
**Applies to:** All developers, architects, code reviewers  
**Location:** Part of `common/AI-rules/originals/`

---

## Rules by Category

### Architecture & Code Organization

- **[ARCHITECTURE_AND_LAYERING.md](./ARCHITECTURE_AND_LAYERING.md)** — Six-tier separation pattern
  - UI, UIControllers, REST API, Services, Repositories, Database
  - Dependency injection requirements
  - Layering violations

- **[NAMING_CONVENTIONS.md](./NAMING_CONVENTIONS.md)** — Code naming standards (Java-specific, see language-extensions for others)
  - PascalCase for classes
  - Component type suffixes
  - File naming requirements
  - Spelling standards

- **[DOCUMENT_ORGANIZATION.md](./DOCUMENT_ORGANIZATION.md)** — Documentation structure
  - Architecture documents folder (`/architecture`)
  - Planning documents folder (`/planning`)
  - README files and indexing
  - Violations to avoid

### Design & API Standards

- **[API_DESIGN.md](./API_DESIGN.md)** — REST API design principles
  - Endpoint and schema definition
  - Structured error models
  - Authorization enforcement

### Quality & Safety

- **[SECURITY_AND_ISOLATION.md](./SECURITY_AND_ISOLATION.md)** — Security and multi-tenant standards
  - Authorization enforcement
  - Audit logging requirements
  - Secrets protection
  - Cross-tenant isolation

- **[EVENTS_AND_OBSERVABILITY.md](./EVENTS_AND_OBSERVABILITY.md)** — Event-driven patterns and logging
  - Event emission rules (after transaction succeeds)
  - Event payload structure
  - Structured logging standards
  - Secrets protection in logs

- **[VERSIONING_AND_CHANGE_DISCIPLINE.md](./VERSIONING_AND_CHANGE_DISCIPLINE.md)** — Semantic versioning
  - PATCH, MINOR, MAJOR version definitions
  - Breaking vs non-breaking changes

### Development Practices

- **[TDD.md](./TDD.md)** — Test-Driven Development (RED-GREEN-REFACTOR cycle)
  - RED-GREEN-REFACTOR workflow for all code changes
  - Regression fix workflow (test first, then fix)
  - Impact on development workflow

---

## Not Included Here

**AI/Automation-Specific Rules:**
- Commit message format
- Command execution safety
- Source protection
- See `ai-automation-safety.md` in parent directory

**Language-Specific Rules:**
- Java Maven configuration
- C++ memory management
- TypeScript/Node patterns
- Angular component lifecycle
- See `../language-extensions/` for language-specific extensions

**Testing Rules:**
- Test organization
- Test quality standards
- Cross-tenant denial tests
- See `../testing-rules.md` for baseline testing standards

---

## Quick Navigation by Role

### For Developers Writing Code
→ Read: Architecture & Layering, Naming Conventions, TDD, API Design

### For Code Reviewers
→ Check: All sections (architecture, naming, security, TDD compliance)

### For Architects
→ Read: Architecture & Layering, API Design, Document Organization, Versioning

### For Security Teams
→ Focus on: Security & Isolation, Events & Observability, API Design

### For DevOps/Compliance
→ Focus on: Document Organization, Security & Isolation, Events & Observability, Versioning

---

**Last Updated:** 2026-07-17  
**Maintained by:** Architecture & Development Team  
**Status:** Production-ready baseline standards
