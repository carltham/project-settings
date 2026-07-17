# Common AI Rules

**PROTECTED READ-ONLY SOURCE:** These are organizational baseline standards for all projects.

**Location:** `./originals/` — Protected from modification

**⚠️ IMPORTANT:** Do not edit files in this directory. See [Global Rules & Local Extensions](#global-rules--local-extensions) for customization patterns.

---

## Quick Navigation

### Baseline Rules (Apply to ALL projects)
- **Development Rules** → [`originals/development.rules.md`](./originals/development.rules.md)
- **Testing Rules** → [`originals/testing-rules.md`](./originals/testing-rules.md)
- **Commands Rules** → [`originals/commands.rules.md`](./originals/commands.rules.md)
- **Full Documentation** → [`originals/README.md`](./originals/README.md)

### Language-Specific Extensions (Optional per tech stack)
- **Java** → [`language-extensions/java-rules-template.md`](./language-extensions/java-rules-template.md)
- **C++** → [`language-extensions/cpp-rules-template.md`](./language-extensions/cpp-rules-template.md)
- **TypeScript/Node.js** → [`language-extensions/typescript-node-rules-template.md`](./language-extensions/typescript-node-rules-template.md)
- **Angular** → [`language-extensions/angular-rules-template.md`](./language-extensions/angular-rules-template.md)
- **Extensions Guide** → [`language-extensions/README.md`](./language-extensions/README.md)

---

## Global Rules & Local Extensions

**These global rules apply to ALL projects.** Each project should have LOCAL rules in its own `AI-rules/` directory.

### Global Rules (this directory) — READ-ONLY

- Apply to all projects
- Baseline standards everyone follows
- Located in `./originals/`
- Protected from modification
- Change only through organizational approval process

### Local Rules (in your project) — EDITABLE

- Project-specific extensions
- Located in your project's `AI-rules/` directory
- EXTEND global rules, don't replace them
- Domain-specific patterns and customizations
- Change as your project evolves

### Creating Local Rules for Your Project

See [`originals/README.md`](./originals/README.md) for:
- Global vs local rule pattern
- How to create project-specific extensions
- Examples of extending naming, security, and testing rules

---

## Why These Rules Exist

- **Consistency** — Same standards across all projects
- **Safety** — Critical rules for automation and AI assistance
- **Compliance** — Security, testing, and operational standards
- **Quality** — Code quality, testing, and architecture patterns

---

## For AI Assistants

When working on a project:

1. **Read global rules** from `./originals/` (this is the baseline)
2. **Read local rules** from the project's `AI-rules/` directory
3. **Follow both** — Local rules extend, don't replace, global rules
4. **Report what changed** — Identify all files modified

**MANDATORY:** Never edit files in `./originals/` or any global rules directory.

---

## Support

**Questions about these rules?**
- Read the detailed documentation in `./originals/README.md`
- Ask your team lead or architect
- Check your project's local rules for extensions

**Found an issue or want to improve a rule?**
- Document the issue with context
- Propose a specific change
- Get approval from technical leads
- Request the update through proper change process

---

**Status:** Organizational baseline standards (protected)  
**Last Updated:** 2026-07-17  
**Maintained by:** Architecture & Development Team
