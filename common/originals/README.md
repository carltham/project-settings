# Common AI Rules

Comprehensive AI-friendly guidelines for development standards, testing practices, and command execution safety.

**Purpose:** Enable consistent, safe, and high-quality development across all projects through documented rules that guide both human developers and AI assistants.

---

## Overview

This directory contains three complementary rule sets:

1. **Development Rules** — Code standards, architecture patterns, naming conventions, **document organization**, **test-driven development (TDD)** (MANDATORY)
2. **Testing Rules** — Test structure, quality, data safety, Java-specific patterns
3. **Commands Rules** — AI command execution, logging, source protection, safety practices

Together, they define how code is written, tested, built, documented, and the operational practices for maintaining consistency and safety.

---

## Rules Files

### 1. Development Rules

**Location:** [`development-rules/`](./development-rules/) (organized by category)

**Purpose:** Mandatory code-level standards, architectural patterns, and development practices

**Audience:** All developers (human and AI), architects, code reviewers

**See** [`development-rules/README.md`](./development-rules/README.md) for full details organized by category:
- Architecture and Layering (six-tier separation pattern)
- API Design (endpoints, error models, authorization)
- Security and Isolation (multi-tenant, audit, secrets)
- Events and Observability (event patterns, logging)
- Versioning and Change Discipline (semantic versioning)
- Naming Conventions (Java class naming - see development-rules/language-specific/ for other languages)
- Document Organization (architecture/ and planning/ structure)
- Test-Driven Development (RED-GREEN-REFACTOR cycle)

---

### 2. Testing Rules

**File:** [`AI-rules/testing-rules.md`](./AI-rules/testing-rules.md)

**Purpose:** Testing standards, quality gates, determinism, isolation, security

**Audience:** QA engineers, developers writing tests, code reviewers, CI/CD teams

**File:** [`AI-rules/testing-rules.md`](./AI-rules/testing-rules.md)

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

**File:** [`AI-rules/commands.rules.md`](./AI-rules/commands.rules.md)

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
- Test organization and naming (language-agnostic)
- Test quality standards and safety
- Isolation and security gates
- Language-specific setup: See `development-rules/language-specific/` (Java, C++, TypeScript/Node, Angular)

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
| 1.1 | 2026-07-17 | README.md | Removed Java-specific Maven configuration; moved to language-extensions; kept language-agnostic testing principles |
| 1.0 | 2026-07-12 | development.rules.md, testing-rules.md, commands.rules.md | Initial release with architecture patterns, testing guidelines, and command safety rules |

---

## FAQ

**Q: Do all three rule files apply to my project?**  
A: Yes. Development Rules define code structure, Testing Rules ensure quality, Commands Rules ensure safe execution and logging.

**Q: Can I edit these global rules?**  
A: **NO.** Global rules in `/project-settings/common/` are PROTECTED and READ-ONLY. These are organizational baseline standards that apply to ALL projects. To customize for your project, create LOCAL rules in your project's `AI-rules/` directory that EXTEND (not replace) these global standards.

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

**Q: I found a problem with a global rule. Can I fix it?**  
A: No. Global rules are protected from direct edits. Instead: (1) Document the issue with context, (2) Propose a specific change, (3) Get approval from technical leads, (4) Have the approved change made through the proper process, (5) Update version history. This ensures changes don't break other projects using these standards.

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
