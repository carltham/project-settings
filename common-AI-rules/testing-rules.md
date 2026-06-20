# Common Testing Rules

## Uploads

- Test uploads must be written to the test-upload directory defined in the project's local `AI-rules/testing-rules.md`.
- Tests that process an uploaded file must fetch the file recursively from that directory; do not hardcode individual file paths in test code.
- Clear the project's test-upload directory before each test or test suite that uses it.

## Logging

- At the start of every test run, create a new log file in `logs/` relative to the project root.
- The filename must start with `Testing-` followed by a timestamp in the format `YYMMDD-HH:MM CEST`, for example `Testing-260618-15:50 CEST.log.md`.
- The file must list the test suite name and a one-line description of what each test in that run does.
