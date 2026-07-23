# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Quick Start

**Repository Type:** [e.g., TypeScript/React web app, Java backend service, Python CLI tool]

**Main Language:** [e.g., TypeScript, Java, Python]

**Build Command:** `[e.g., npm install && npm run build]`

**Test Command:** `[e.g., npm test]`

**Run Command:** `[e.g., npm start or java -jar target/app.jar]`

---

## Project Structure

```
[project-root]/
├── src/              (Source code)
├── tests/            (Test files)
├── docs/             (Documentation)
├── package.json      (or pom.xml, requirements.txt, etc.)
└── README.md         (Project overview)
```

[Add your specific directory structure and what each contains]

---

## Key Development Rules

**Reference:** See [project-settings/common/originals/development-rules/](../../../common/originals/development-rules/) for organizational standards.

**Local Extensions:** See `./AI-rules/` directory for project-specific rules.

### Mandatory Standards

- **Test-Driven Development:** Write failing test before implementation
- **Architecture:** [Reference your specific architecture pattern]
- **Security:** [Any domain-specific security requirements]
- **Code Style:** [Language-specific conventions or style guide]

---

## Common Commands

```bash
# Install dependencies
[e.g., npm install / mvn clean install / pip install -r requirements.txt]

# Run tests
[e.g., npm test / mvn test]

# Run single test
[e.g., npm test -- MyTest.ts / mvn test -Dtest=MyTest]

# Lint
[e.g., npm run lint / mvn checkstyle:check]

# Build
[e.g., npm run build / mvn clean package]

# Run locally
[e.g., npm start / java -jar target/app.jar]
```

---

## Architecture Overview

[Brief description of:
- Main layers/components
- Key entry points
- Critical file locations
- Design patterns used
]

---

## Working with Standards

**Global Rules (Read-Only):**
- All projects follow standards in [project-settings/common/originals/](../../../common/originals/)
- Never edit files in `originals/` directory

**Local Rules (Project-Specific):**
- Project-specific extensions in `./AI-rules/`
- Extends global rules, doesn't replace them

---

## Before You Begin

- [ ] Read [project-settings/common/README.md](../../../common/README.md) for organizational standards
- [ ] Check `./AI-rules/README.md` for local project rules
- [ ] Review architecture docs in `./docs/` or `./architecture/`

---

**Last Updated:** [DATE]

