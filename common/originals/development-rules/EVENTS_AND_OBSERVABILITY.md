# Events and Observability Standards - MANDATORY

## Event Emission

- Emit business events ONLY AFTER the associated transaction succeeds
- Prevents events for failed operations
- Maintains consistency between events and database state

## Event Payloads

Event payloads MUST:
- Include stable aggregate identity (ID, version)
- Include enough context for audit correlation
- MUST NOT leak secrets, tokens, or sensitive data

## Structured Logging

Structured logs SHOULD include (when permitted):
- Tenant ID
- Correlation ID
- Actor ID
- Event type

Structured logs MUST NOT:
- Expose confidential content
- Include secrets or tokens
- Include personal data

---

**Last Updated:** 2026-07-17  
**Status:** Mandatory organizational standard
