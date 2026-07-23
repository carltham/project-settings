# AI Permission Gates & Hooks Configuration

This guide describes how to configure `.claude/settings.json` to enforce permission gates and hooks that prevent Claude from taking unasked actions on files.

## Overview

Two mechanisms work together to enforce proper permission discipline:

1. **Permission Ask Rules** — Force Claude to ask before executing specific operations
2. **PreToolUse Hooks** — Display warnings/reminders before file modifications

## Permission Ask Rules

### Configuration

Add Edit and Write operations to the "ask" list in `.claude/settings.json`:

```json
{
  "permissions": {
    "ask": [
      "Edit(*)",
      "Write(*)"
    ]
  }
}
```

### What This Does

- **Edit(*)** — Requires explicit approval before editing ANY file
- **Write(*)** — Requires explicit approval before creating ANY file

When Claude attempts to edit or write a file, the user sees a permission prompt and must explicitly say "yes" to proceed.

### Scope Control

You can restrict permissions to specific directories:

```json
{
  "permissions": {
    "ask": [
      "Edit(//path/to/critical/dir/**)",
      "Write(//path/to/critical/dir/**)"
    ]
  }
}
```

## PreToolUse Hooks

### Configuration

Add warning hooks for Edit and Write operations in the "hooks" section:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit(*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️  WARNING: About to modify file - explicit permission required'"
          }
        ]
      },
      {
        "matcher": "Write(*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️  WARNING: About to write file - explicit permission required'"
          }
        ]
      }
    ]
  }
}
```

### What This Does

- Displays a warning message **before** any file modification
- Reminds Claude (and the user) that explicit permission is required
- Provides visual feedback that a critical operation is about to occur

## Complete Example

Here's a complete `.claude/settings.json` with both permission gates and hooks:

```json
{
  "permissions": {
    "ask": [
      "Edit(*)",
      "Write(*)"
    ],
    "allow": [
      "Read(//path/to/project/**)",
      "Bash(ls *)",
      "Bash(find *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit(*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️  WARNING: About to modify file - explicit permission required'"
          }
        ]
      },
      {
        "matcher": "Write(*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️  WARNING: About to write file - explicit permission required'"
          }
        ]
      }
    ]
  }
}
```

## Best Practices

### 1. Always Use Both Permission Gates + Hooks

Combine both mechanisms for defense-in-depth:
- **Hooks** provide visual feedback and warnings
- **Permission ask** enforces actual approval requirement

### 2. Default to "Ask"

Start with "ask" for all file operations. Selectively add "allow" rules only for operations that are safe to run unattended:

```json
{
  "permissions": {
    "ask": ["Edit(*)", "Write(*)"],
    "allow": ["Read(**)", "Bash(ls *)", "Bash(find *)"]
  }
}
```

### 3. Document Critical Operations

Use CLAUDE.md to document which operations require explicit approval:

```markdown
## File Modification Rules

All Edit and Write operations require explicit user approval before proceeding.
Claude will ask before:
- Creating new files
- Modifying existing files
- Writing to configuration files

This is enforced via .claude/settings.json permission gates.
```

### 4. Scope by Directory (Optional)

For complex projects, restrict ask rules to critical directories:

```json
{
  "permissions": {
    "ask": [
      "Edit(//config/**)",
      "Edit(//src/core/**)",
      "Write(//config/**)"
    ],
    "allow": [
      "Edit(//docs/**)",
      "Edit(//tests/**)"
    ]
  }
}
```

## Testing Permission Gates

### Verify "Ask" Rule

Try editing a file. Claude should:
1. Display the warning hook message
2. Ask for permission before proceeding
3. Require explicit "yes" to continue

### Verify Hooks

Run any operation that matches a hook matcher. You should see:

```
⚠️  WARNING: About to modify file - explicit permission required
```

## Troubleshooting

### Hooks Not Displaying

- Ensure hook event name is correct: `PreToolUse` (case-sensitive)
- Verify matcher pattern matches the tool call (e.g., `Edit(*)`)
- Check that the command is valid shell syntax

### Permission Prompts Not Appearing

- Confirm "ask" rules are in permissions section
- Verify pattern syntax: `Edit(*)` not `Edit`
- Check that settings.json is valid JSON (no syntax errors)
- Restart Claude Code after modifying settings

### Too Many Permission Prompts

If you're being prompted too often:
- Move safe operations to "allow" list
- Use more specific patterns: `Edit(//src/safe/**)`
- Consider whether prompt discipline is worth the friction

## Related Documentation

- [CLAUDE.md Guidelines](./CLAUDE-Guidelines.md) — Document project rules
- [Settings.json Schema](https://docs.claude.com/settings) — Full permission configuration
- [Hooks Reference](https://docs.claude.com/hooks) — All available hook events
