# Development Rules - Non-AI Specific

Comprehensive development standards for code quality, architecture, and organizational practices.

**Status:** Protected read-only baseline standards  
**Applies to:** All developers, architects, code reviewers  
**Location:** Part of `common/originals/development-rules/`

---

## Overview

This directory contains mandatory development standards applicable to all developers (human and AI), organized by category. Language-specific templates are available in `language-specific/` subdirectory for projects to copy and customize.

---

## Rules by Category

### Architecture & Code Organization

- **[ARCHITECTURE_AND_LAYERING.md](./ARCHITECTURE_AND_LAYERING.md)** — Six-tier separation pattern
  - UI, UIControllers, REST API, Services, Repositories, Database
  - Dependency injection requirements
  - Layering violations

- **[NAMING_CONVENTIONS.md](./NAMING_CONVENTIONS.md)** — Code naming standards (Java-specific, see language-specific/ for others)
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

## Language-Specific Templates

**Location:** [`language-specific/`](./language-specific/) (copy & customize for your project)

These are **TEMPLATES** that projects should copy to their local `AI-rules/` directory and customize for their specific language and tech stack.

- **[java-template.md](./language-specific/java-template.md)** — Java development rules
  - Maven configuration (Surefire, Failsafe, JaCoCo)
  - Test naming conventions (Unit vs Integration)
  - Code coverage setup and requirements
  - Integration test patterns

- **[cpp-template.md](./language-specific/cpp-template.md)** — C++ development rules
  - Memory safety (smart pointers, RAII)
  - Naming conventions
  - Error handling
  - Testing patterns

- **[typescript-node-template.md](./language-specific/typescript-node-template.md)** — TypeScript/Node.js development rules
  - Type safety and strict mode
  - Layer separation
  - Dependency injection
  - Error handling
  - Express, Fastify, NestJS patterns

- **[angular-template.md](./language-specific/angular-template.md)** — Angular development rules
  - Component lifecycle management
  - Observable subscriptions and memory leak prevention
  - Type safety and strict mode
  - Testing requirements
  - Change detection strategies

---

## Not Included Here

**AI/Automation-Specific Rules:**
- Commit message format
- Command execution safety
- Source protection
- See `../AI-rules/development.rules.md` in parent directory

**Testing Rules:**
- Test organization
- Test quality standards
- Cross-tenant denial tests
- See `../AI-rules/testing-rules.md` in parent directory

**Commands & Safety:**
- AI command execution safety
- Logging practices
- See `../AI-rules/commands.rules.md` in parent directory

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

### For Language-Specific Setup
→ Copy template from `language-specific/` and customize for your project

---

**Last Updated:** 2026-07-17  
**Maintained by:** Architecture & Development Team  
**Status:** Production-ready baseline standards
