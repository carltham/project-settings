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
