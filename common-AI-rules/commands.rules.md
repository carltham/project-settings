# Common User Command History Rules

## Logging order

- **Before starting any job**, log the user's command in `logs/commands.log.md` in the local project, then begin the work.

## History

- Store the actual command history in `logs/commands.log.md` in the local project.
- Keep only the latest 40 entries, newest first.
- The newest entry is always numbered 40. When a new entry is added, renumber all existing entries downward by 1. Drop the oldest entry if it would be pushed below 1.

## Entry format

```text
N. YY-MM-DD HH:MM CEST (prefix): ~ command
```

- Always use system time in the Swedish timezone (CEST/CET). Use `time unavailable` only if system time cannot be obtained.
- Preserve the user's command text exactly as written.

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
