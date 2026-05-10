---
name: dev-run-tests
description: "Run the project test suite or selected tests and report RED/GREEN state with counts and failures."
---

# Dev Run Tests

Provide reliable test evidence for RED/GREEN workflow gates.

## Usage

- `/dev-run-tests {feature-name}`
- `/dev-run-tests --target {path-or-selector}`

## Hard Rules

- NEVER fake test results.
- NEVER summarize failures without preserving the actionable failure message.
- NEVER treat collection errors as GREEN.
- MUST report command, counts, and status.

## Steps

1. Ensure runtime readiness via `/dev-env-setup` or verified equivalent.
2. Resolve test command from project context and language.
3. Execute the narrowest relevant test target when possible.
4. Capture:
   - command
   - collected
   - passed
   - failed
   - errored
   - skipped
   - exit code
   - failure excerpts
5. Classify:
   - `RED` if tests fail as expected
   - `GREEN` if all required tests pass
   - `ERROR` if infrastructure/collection failed

## Output Contract

```json
{
  "test_command": "string — the command that was executed",
  "status": "RED|GREEN — overall test suite result",
  "collected": "number — total tests collected",
  "passed": "number — tests passed",
  "failed": "number — tests failed",
  "errors": "string[] — failure messages/stack traces",
  "execution_time": "string — duration of test run"
}
```
