# API Design Standards - MANDATORY

## Endpoints and Schemas

Define public endpoints and schemas before implementing controllers or clients.

## Error Model

Use one common structured error model with:
- Stable code
- Error ID
- Timestamp
- Severity
- Source
- Field details

Consistent error handling reduces client bugs.

## Authorization

UI guards improve navigation but never replace server-side authorization.

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
