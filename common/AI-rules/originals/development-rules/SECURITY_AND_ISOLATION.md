# Security and Isolation Standards - MANDATORY

## Authorization Enforcement

- Enforce authorization in both API and business-service layers
- Reject cross-tenant or out-of-scope identifiers before repository access
- Scope every repository query by tenant/company as applicable

## Audit & Actor Tracking

Record the actor and correlation context for every write operation.

## Secrets Protection

- Never store refresh tokens in browser local storage
- Keep provider secrets and tokens encrypted at rest
- Never include secrets in:
  - Configuration files
  - Logs
  - Errors
  - Events
  - API responses

## Audit Logging

Audit the following events:
- Authentication events
- Permission failures
- All write operations
- Imports
- Privileged access

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
