# Common Testing Rules

## Uploads

- Test uploads must be written to the test-upload directory defined in the project's local `AI-rules/testing-rules.md`.
- Tests that process an uploaded file must fetch the file recursively from that directory; do not hardcode individual file paths in test code.
- Clear the project's test-upload directory before each test or test suite that uses it.

## Logging

- At the start of every test run, create a new log file in `logs/` relative to the project root.
- The filename must start with `Testing-` followed by a timestamp in the format `YYMMDD-HH:MM CEST`, for example `Testing-260618-15:50 CEST.log.md`.
- The file must list the test suite name and a one-line description of what each test in that run does.

## Java rules

### Naming

- Integration tests must be named with the suffix `IT` (e.g. `CashierControllerIT`). Never use `ITTest` or `TestIT`.
- Unit tests must NOT use the `IT` suffix.

### Maven setup

- Normal `mvn test` must never run integration tests (`*IT.java`). Exclude them explicitly in the Surefire plugin configuration.
- Normal `mvn test` must always produce a JaCoCo code coverage report. Add JaCoCo `prepare-agent` + `report` goals to the default build — not inside a profile.
- Integration tests belong in a dedicated Maven profile (e.g. `integration-tests`) using the Failsafe plugin targeting `*IT.java` files.
- That integration-tests profile must also collect JaCoCo coverage, using `prepare-agent-integration` + `report-integration` with a separate output file (e.g. `jacoco-it.exec`).
- Use `@{argLine}` (late-binding) in both Surefire and Failsafe `<argLine>` so the JaCoCo agent is picked up automatically.

## Test data safety

- Never use production credentials, personal data, live accounts, or real documents in automated tests.

## Test quality

- Tests MUST be deterministic, independent, and repeatable. Do not depend on execution order, wall-clock timing, public networks, or mutable external data.
- Freeze or inject time, UUID generation, provider clients, and exchange-rate sources where outcomes depend on them.
- Prefer observable business outcomes over implementation-detail assertions.
- A regression fix MUST first be represented by a test that fails for the reported defect.
- Do not weaken, delete, skip, or broadly mock a failing gate merely to obtain a green build.

## Isolation and security gates

- Every data-access feature MUST include a cross-tenant denial test and, where applicable, cross-scope tests.
- Assert secrets, tokens, confidential payloads, and document contents do not appear in logs, audit events, errors, snapshots, or API responses.
