# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Quick Start

**Repository Type:** Organizational standards and architecture templates (documentation)

**Primary Content:**
- Development standards and best practices
- Architecture design templates  
- Testing and security guidelines
- AI/automation safety rules

**Key Principle:** `common/originals/` is READ-ONLY. Never edit these protected baseline standards.

---

## Project Structure

```
project-settings/
├── common/
│   ├── README.md              (Navigation guide - START HERE)
│   └── originals/             (Protected baseline standards)
│       ├── AI-rules/          (AI safety, testing, commands)
│       ├── development-rules/ (Code standards, architecture, security)
│       └── architecture/      (System design templates)
├── CLAUDE-TEMPLATE.md         (Generic template for all projects)
└── CLAUDE.md                  (This file)
```

---

## Critical Rule: Protect `originals/`

⚠️ **Never edit, move, or delete files in `common/originals/`**

These are organizational baseline standards used by all projects.

✅ **Instead:**
- Read files as reference
- Copy templates to your project
- Create local rules in your project's `AI-rules/` directory

---

## Common Tasks

**Understand organizational standards:**
```bash
cat common/README.md                           # Overview & navigation
cat common/originals/development-rules/README.md
cat common/originals/AI-rules/README.md
```

**Find a template:**
```bash
ls common/originals/architecture/              # System design templates
ls common/originals/development-rules/language-specific/  # Language templates
```

**Copy template to a project:**
```bash
cp common/originals/CLAUDE-TEMPLATE.md your-project/CLAUDE.md
# Then customize the copy
```

**Create local project rules:**
```bash
mkdir -p your-project/AI-rules/
# Reference global rules, extend with domain-specific patterns
```

---

## Key Architecture

**Global Rules** (protected, baseline) → **Local Rules** (editable, project-specific)

Every project:
1. Follows mandatory global rules from `common/originals/`
2. Extends with local rules in its own `AI-rules/` directory
3. Local rules specialize, never weaken, global standards

---

## Git Workflow

```bash
# View changes to standards
git log --oneline common/originals/

# Review what changed
git diff HEAD~1 common/originals/

# Commit standard updates (with justification)
git commit -m "Update standard: [what changed]

Why: [rationale]
Affected: [which projects/teams]"
```

**Never:**
- Force push to main
- Amend shared commits
- Edit `originals/` without approval

---

**Repository Type:** Organizational Standards & Templates  
**Status:** Active - continuously evolving  
**Last Updated:** 2026-07-23
