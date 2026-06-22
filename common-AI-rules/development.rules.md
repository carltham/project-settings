# Common Development Rules

## Git

- Do not commit anything automatically. Always wait for an explicit commit instruction from the user.

## AI and external input

- AI and OCR output is untrusted advisory data. It MAY propose or pre-fill a command but MUST NOT post or persist data directly.
- Accepting a suggestion MUST enter the same authorization, validation, and audit workflow as manually entered input.

## API design

- Define public endpoints and schemas before implementing controllers or clients.
- Use one common structured error model with a stable code, error ID, timestamp, severity, source, and field details.
- UI guards improve navigation but never replace server-side authorization.

## Security and isolation

- Enforce authorization in both API and business-service layers.
- Reject cross-tenant or out-of-scope identifiers before repository access, and scope every repository query by tenant/company as applicable.
- Record the actor and correlation context for every write.
- Never store refresh tokens in browser local storage.
- Keep provider secrets and tokens encrypted at rest and out of configuration files, logs, errors, events, and API responses.
- Audit authentication events, permission failures, writes, imports, and privileged access.

## Events and observability

- Emit business events only after the associated transaction succeeds.
- Event payloads MUST include stable aggregate identity and enough context for audit correlation without leaking secrets.
- Structured logs SHOULD include tenant ID, correlation ID, actor ID, and event type when permitted, but MUST NOT expose confidential content.

## Versioning and change discipline

- `PATCH`: clarification or compatible correction with no new runtime responsibility, schema migration, endpoint, event, or report contract.
- `MINOR`: backward-compatible behavior or contract addition, including a forward schema extension.
- `MAJOR`: breaking compatibility, changed module responsibilities, or a substantial new solution surface.
