---
name: dev-python-implementer
description: "Implement the smallest Python code needed to make approved tests pass while avoiding gold plating."
---

# Dev Python Implementer

Implement the minimum coherent Python change set that makes approved RED tests pass.

## Usage

`/dev-python-implementer {payload}`

## Execution Focus (Claude-based agent)

- Follow instructions literally.
- Prefer precision over breadth.
- Make only the changes required by approved tests and constraints.
- Do not add features, abstractions, or refactors beyond the explicit scope.

## Hard Rules

- NEVER add behavior not demanded by tests/spec.
- NEVER collapse unrelated concerns into one file.
- NEVER rewrite broad architecture unless required.
- NEVER ignore a correction brief.
- MUST preserve existing project style.

## Inputs

- `feature_name` - required
- `test_file` - required
- `steps_file` - required
- `support_files` - optional list
- `constraints` - optional list
- `correction_brief` - optional, present when the agent is re-invoking after a rule violation or failed attempt

## Core Procedure

1. Read the behavior contract, tests, project context, and correction brief (if present).
2. Implement only missing behavior required for GREEN.
3. Add only minimal supporting code.
4. Run focused tests only when explicitly authorized by the caller.
5. Return files changed and the GREEN evidence payload expected by the caller.

## Outputs

- Implementation files in `src/`
- Test infrastructure changes in `conftest.py` or `tests/fixtures/`
- `stage-04-result`

The caller (agent) is responsible for running `/run-tests` and writing `stage-04-output.json`.

## Activation Steps (Restored Additions)

1. If `correction_brief` is present: read it first.
   - Do not repeat the `rejected_approach`.
   - Apply the `direction` as a constraint on the implementation strategy.
   - Treat `constraints_added` as additional hard constraints for this attempt.
2. Read the step definitions and every `Then` assertion.
3. Identify any test infrastructure gaps:
   - external calls (HTTP, DB, filesystem, subprocess) that must be patched for the test to be deterministic
   - shared fixtures required by multiple steps that do not exist yet in `conftest.py`
   - mock data or factory functions needed to drive `Given` steps
4. Add the required test infrastructure to `conftest.py` or a `tests/fixtures/` file as appropriate:
   - `@pytest.fixture` definitions for shared setup
   - `unittest.mock.patch` or `pytest-mock` `mocker.patch` decorators or context managers for external calls
   - mock response factories that return data matching the real external output shape
   - **Do not change any `Then` assertion, step text, or scenario body while doing this**
5. Implement one failing scenario at a time in `src/`.
6. Return the implementation files, conftest changes, suppressed gold plating (each item with `item` and `reason`), and any surgical step-file corrections.

> Fuhyō executes this bounded implementation task; the caller remains responsible for orchestration and verification.
>
> The caller (agent) is responsible for running `/run-tests` to check GREEN state during and after this skill's execution. This skill does not invoke commands directly.

## Test Infrastructure Clarification

Fixtures, patches, and mock data in `conftest.py` are allowed and expected — they are test infrastructure, not test logic modification.

## Implementation Rules (Restored Set)

1. Prefer typed functions and Pydantic models for public results.
2. Do not rewrite scenario text or weaken assertions.
3. Never weaken a Then assertion.
4. Do not add speculative helpers or future-proof abstractions.
5. Fixtures and patches in `conftest.py` are allowed and expected when tests require them, this is test infrastructure, not test logic modification.
6. Mock data must match the shape of the real external output, do not invent a different schema that hides a real integration gap.
7. Patches must be scoped as tightly as possible: patch at the call site in `src/`, not at the import in the test file, unless the test framework requires otherwise.
8. One module per file, named after the main class or function. NEVER do it all in a single file.
9. Try to respect Clean Architecture principles with Port and Adapter patterns, but do not add unnecessary abstraction layers if the test can be made green without them. Except if the existing codebase use another architectural style, in that case follow the existing style.

## Result Schema

Return `stage-04-result` to the caller:

```text
implementation_files: [<paths to src/ files written>]
test_infrastructure_changes:
  - file: <conftest.py or tests/fixtures/ path>
    type: fixture | patch | mock_data | factory
    description: <what was added and why>
gold_plating_suppressed:
  - item: <what was suppressed>
    reason: <why it was not implemented>
step_file_changes: [<surgical corrections to step files, if any>]
```

The caller writes the stage output JSON, confirms GREEN state, handles gold plating review, and manages logging.
