# Common Development Rules

## Git & Automation (MANDATORY)

### AI and Automation Safety

- **AI MUST NEVER commit unless explicitly asked to do so.** (MANDATORY)
  - Never assume commit is needed after changes
  - Never commit as part of task completion
  - Wait for explicit user instruction: "commit these changes" or "create a commit"
  - Exception: Only commit if user explicitly requests in current turn

- Do not commit anything automatically. Always wait for an explicit commit instruction from the user.
- AI and OCR output is untrusted advisory data. It MAY propose or pre-fill a command but MUST NOT post or persist data directly.
- Accepting a suggestion MUST enter the same authorization, validation, and audit workflow as manually entered input.

### Commits Require Explicit Authorization
- User must say: "commit these changes", "make a commit", "create a commit with...", etc.
- Implications: Never create commits as part of task completion workflow
- User retains full control over when changes are persisted to git
- Always report what changed, let user decide commit timing

### Global Rules & Design Files are Read-Only (MANDATORY)

**NEVER edit, modify, rename, move, or delete files in `/project-settings/common/`** (CRITICAL)

- Global rules in `/project-settings/common/AI-rules/` are baseline standards for ALL projects
- Global architecture templates in `/project-settings/common/architecture/originals/` are reference material
- These files are FROZEN and PROTECTED from modification
- Exception: Only update when explicitly authorized by technical leadership for organizational-wide change
- Process for organizational changes:
  1. Document justification and impact analysis
  2. Get approval from technical leads (multiple reviewers)
  3. Update version history with change rationale
  4. Notify all projects that depend on these standards
  5. Allow grace period for projects to adapt

**What to do instead:**
- Create LOCAL rules in your project's `AI-rules/` directory
- LOCAL rules EXTEND (not replace) global rules
- Reference global rules in your local documentation
- Propose improvements to global standards through proper change process

**Why this rule exists:**
- Global rules must remain stable across all projects
- Accidental edits break consistency for all projects using these standards
- Changes to baseline standards require coordinated team review
- Protects organizational standards from drift and corruption

Violations:
- ❌ Editing `/project-settings/common/AI-rules/*.md` files
- ❌ Editing `/project-settings/common/architecture/originals/*.md` files
- ❌ Modifying any file in `/project-settings/common/` without explicit authorization
- ❌ "Quick fixes" to global rules without team approval

### Commit Message Format (MANDATORY)
When requested to provide a commit message, follow this format:
- **Length**: Concise, ~10 rows maximum
- **Format**: Copyable plain text (can be pasted directly)
- **Scope**: Summarize changes since last commit
- **Structure**: 
  - Line 1: Subject (imperative, present tense)
  - Lines 2-N: Details, bullet points, rationale
  - Last line: Co-Authored-By footer
- **Example Request**: "give me a short 10 rows commit message since last commit, copyable"
- **Delivery**: Plain text code block with no markdown formatting of the message itself

**Specialized Formats:**
- Projects may request custom commit message formats for specific workflows
- Submit specialized formats as project-level rules (in project's `AI-rules/` directory)
- Custom formats extend this generic standard but must follow ~10 row, copyable plain text principles

---

## Non-AI Development Rules

**All other development standards have been organized in the `development-rules/` folder:**

See [`development-rules/README.md`](./development-rules/README.md) for:
- Architecture and Layering Standards
- API Design
- Security and Isolation
- Events and Observability
- Versioning and Change Discipline
- Naming Conventions (Java-specific)
- Document Organization and Structure
- Test-Driven Development (TDD)

These rules apply to **all developers** (human and AI), architects, and code reviewers.

---

**Last Updated:** 2026-07-17  
**Status:** This file contains AI/Automation-specific rules. See `development-rules/` for all other development standards.
