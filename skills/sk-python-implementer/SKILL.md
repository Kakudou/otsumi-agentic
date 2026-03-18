---
name: sk-python-implementer
description: Writes the minimal Python implementation required to make RED pytest-bdd tests pass.
---

# python-implementer

One job: make `python + pytest-bdd` tests GREEN with minimal code.

## Inputs

- `feature_name` - required
- `test_file` - required
- `steps_file` - required
- `support_files` - optional list
- `constraints` - optional list
- `correction_brief` - optional, present when the agent is re-invoking after a rule violation or failed attempt

## Outputs

- Implementation files in `src/`
- Test infrastructure changes in `conftest.py` or `tests/fixtures/`
- `stage-04-result`

The caller (agent) is responsible for running `/run-tests` and writing `stage-04-output.json`.

## Activation Steps

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

> The caller (agent) is responsible for running `/run-tests` to check GREEN state during and after this skill's execution. This skill does not invoke commands directly.

## Rules

- Prefer typed functions and Pydantic models for public results.
- Do not rewrite scenario text or weaken assertions.
- Do not add speculative helpers or future-proof abstractions.
- Fixtures and patches in `conftest.py` are allowed and expected when tests require them, this is test infrastructure, not test logic modification.
- Mock data must match the shape of the real external output, do not invent a different schema that hides a real integration gap.
- Patches must be scoped as tightly as possible: patch at the call site in `src/`, not at the import in the test file, unless the test framework requires otherwise.
- One module per file, named after the main class or function.
- NEVER do it all in a single file.
- Try to respect Clean Architecture principles with Port and Adapter patterns, but do not add unnecessary abstraction layers if the test can be made green without them. Except if the existing codebase use another architectural style, in that case follow the existing style.

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
