# Common User Command History Rules

## CRITICAL rules

🚫 STOP: FORBIDDEN TO GIT COMMIT UNLESS EXPLICITLY ASKED 🚫

🚫 STOP: FORBIDDEN TO EVER GIT PUSH UNLESS EXPLICITLY ASKED 🚫

## Logging order

- **Before starting any job**, log the user's command in `logs/commands.log.md` in the local project, then begin the work.

## History

- Store the actual command history in `logs/commands.log.md` in the local project.
- Keep only the latest 40 entries, newest first (discard older entries as new ones are added).

## Entry format

```text
N. YY-MM-DD HH:MM (prefix): ~ command
```

- Use system timezone and time
- Preserve the user's command text exactly as written
- Use `time unavailable` only if system time cannot be obtained

## AI prefixes

| Prefix | AI |
|---|---|
| `(cc)` | Claude Code |
| `(cx)` | Codex |
| `(gc)` | Gemini Code |

## Protect project sources

- Treat every directory named `originals/` as read-only source material.
- Never edit, rename, move, overwrite, or delete a file below an `originals/` directory.
- Never include an `originals/` directory in cleanup, generated-output, formatting, or migration commands.
- Read supplied source material in place. Do not copy its contents into production code, tests, fixtures, or documentation unless the task explicitly requires a legally permissible excerpt.
- Before a recursive delete or cleanup, inspect the resolved targets and prove that no target is an `originals/` directory or one of its ancestors.

## Protect global baseline standards

- **NEVER edit, modify, rename, move, or delete files in `/project-settings/common/`** (MANDATORY)
- Global rules (`/project-settings/common/AI-rules/`) and architecture templates (`/project-settings/common/architecture/originals/`) are organizational baseline standards
- These are frozen and protected from modification across all projects
- Exception: Only authorized by technical leadership for organizational-wide standards updates
- When standards need improvement: Create local extensions in your project's `AI-rules/` directory, propose changes through proper review process

## Command safety

- Inspect repository status before modifying files and preserve unrelated user changes.
- Prefer scoped, non-interactive commands with explicit paths.
- Do not use destructive Git commands, force pushes, or broad filesystem deletion unless the user explicitly requests the exact operation.
- Do not discard, rewrite, or stage unrelated changes.
- Do not expose credentials, tokens, personal data, or provider payloads in command output, logs, fixtures, or commits.
- Keep secrets out of command arguments when they may be recorded in shell history or process listings.
- Ask for approval before installing software, accessing external systems, changing infrastructure, or performing an irreversible operation when that authority was not already given.

## Generated and build output

- Generate output only in the module's configured build directory or another documented temporary directory.
- Do not commit build output, dependency caches, runtime secrets, uploaded documents, database volumes, or local identity-provider state.

## Completion evidence

- Run the smallest relevant verification first, followed by the required module or release gates when practical.
- Report commands that failed or were not run. Never claim a check passed without its successful output.
- Leave the working tree understandable: identify files changed, checks performed, and any known risk or follow-up.
