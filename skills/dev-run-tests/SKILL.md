---
name: dev-run-tests
description: "Run the project test suite or selected tests and report RED/GREEN state with counts and failures."
---

# Dev Run Tests

Execute tests and report state.

## Usage

- `/dev-run-tests <feature-name>`
- `/dev-run-tests --target <path-or-selector>`

## Purpose

Provide reliable test evidence for RED/GREEN workflow gates.

## Hard Rules

- NEVER fake test results.
- NEVER summarize failures without preserving the actionable failure message.
- NEVER treat collection errors as GREEN.
- ALWAYS report command, counts, and status.

## Steps

1. Ensure runtime readiness through `/dev-env-setup` or verified equivalent.
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
