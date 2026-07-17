# Language-Specific AI Rules Extensions

Optional language-specific extensions to global baseline AI rules.

**Location:** `../originals/` (extends core rules)  
**Status:** Protected read-only source (in `language-extensions/` subdirectory)

---

## How to Use These Templates

1. **Copy to your project's `/AI-rules/` folder**:
   ```bash
   cp -r templates/AI-rules/* your-project/AI-rules/
   ```

2. **Customize for your project**:
   - Replace generic examples with your project's tech stack
   - Add project-specific constraints
   - Remove rules that don't apply

3. **Extend with architectural patterns**:
   - Add cross-layer patterns specific to your architecture
   - Document your security model
   - Define your layering constraints

4. **Integrate with development workflow**:
   - Reference in CLAUDE.md
   - Use checklists in code review process
   - Enforce via settings.json hooks

---

## Available Templates

### Language-Specific Rules

1. **java-rules-template.md** — Java Mandatory Rules
   - Maven configuration (Surefire, Failsafe, JaCoCo)
   - Test naming conventions (Unit vs Integration)
   - Code coverage setup and requirements
   - Integration test patterns
   - Code review checklist
   - Customizable for different Java projects

2. **cpp-rules-template.md** — C++ Mandatory Rules
   - Memory safety (smart pointers, RAII)
   - Naming conventions
   - Error handling
   - Code review checklist
   - Customizable for different C++ projects

3. **typescript-node-rules-template.md** — TypeScript/Node.js Mandatory Rules
   - Type safety and strict mode
   - Layer separation
   - Dependency injection
   - Error handling
   - Code review checklist
   - Customizable for Express, Fastify, NestJS, etc.

4. **angular-rules-template.md** — Angular Mandatory Rules
   - Component lifecycle management
   - Observable subscriptions
   - Type safety
   - Testing requirements
   - Code review checklist
   - Customizable for different Angular versions

---

## Template Features

Each template includes:
- ✅ **Mandatory constraints** (language-specific best practices)
- ✅ **Clear examples** (what NOT to do, what to do instead)
- ✅ **Enforcement mechanisms** (how to verify compliance)
- ✅ **Code review checklist** (actionable verification steps)
- ✅ **Why statements** (reasoning behind each rule)

---

## Customization Guide

### Remove or Adapt Rules
- If a rule doesn't apply to your project, delete it
- If you need to relax a constraint, document why
- If you need to add constraints, follow the same format

### Add Examples
- Replace generic examples with your project's patterns
- Show correct vs incorrect code for your use cases
- Link to existing code in your repository

### Extend with Architecture
- Add cross-layer patterns (e.g., API design, data flow)
- Define your security model
- Document your composition or orchestration patterns

### Enforce in Practice
- Reference rules in code review process
- Use checklist items to gate merges
- Run linters/hooks to verify compliance

---

## Structure

Each rules template follows this format:

```
# [Language] Rules for [Project/Team]

Mandatory constraints for [context].

## Category (MANDATORY)

### Rule N: [Constraint Description]
- **MANDATORY**: [What must be done]
- **NEVER**: [What must not be done]
- **Why**: [Reasoning]
- **Enforcement**: [How to verify]

...more rules...

## Code Review Checklist

- ✅ Item 1
- ✅ Item 2
...

**Last Updated:** [Date]
**Status:** Mandatory for all implementation
```

---

## Best Practices

1. **Keep rules focused** — One responsibility per rule
2. **Use MANDATORY sparingly** — Only for constraints that block merge
3. **Provide examples** — Show both correct and incorrect code
4. **Document why** — Explain the reasoning (security, maintainability, performance)
5. **Make checklists actionable** — Specific, verifiable items for code review
6. **Version control** — Track changes to rules in git
7. **Communicate changes** — Announce rule updates to the team

---

## When to Use Templates

✅ **Starting a new project** — Base your rules on templates  
✅ **Adding a new language** — Copy template, customize for your needs  
✅ **Building team standards** — Use as baseline, extend with team-specific rules  
✅ **Training new developers** — Reference templates to explain best practices  

❌ **Don't use templates as-is** — Always customize for your project  
❌ **Don't ignore rules** — Treat them as mandatory, not suggestions  

---

**Template Version:** 1.0  
**Last Updated:** 2026-07-17  
**License:** Open for team use
