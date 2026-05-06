---
name: sk-python-implementer
description: Writes the minimal Python implementation required to make RED pytest-bdd tests pass.
---

# python-implementer

One job: make `python + pytest-bdd` tests GREEN with minimal code.

## Hard Rules

- MUST use typed functions and Pydantic models for public results.
- NEVER rewrite scenario text or weaken assertions.
- NEVER add speculative helpers or future-proof abstractions.
- MUST scope patches as tightly as possible: patch at the call site in `src/`, not at the import in the test file, unless the test framework requires otherwise.
- MUST match mock data to the shape of the real external output; NEVER invent a schema that hides a real integration gap.
- NEVER put everything in a single file: MUST use one module per file, named after the main class or function.
- MUST follow Clean Architecture with Port and Adapter patterns unless the existing codebase uses a different architectural style — in that case MUST follow the existing style. NEVER add abstraction layers beyond what is required to make tests GREEN.
- Fixtures and patches in `conftest.py` are test infrastructure, NOT test logic modification — they are allowed and expected.

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
   - NEVER repeat the `rejected_approach`.
   - Apply the `direction` as a hard constraint on the implementation strategy.
   - Treat `constraints_added` as additional hard constraints for this attempt.
2. Read the step definitions and every `Then` assertion; STOP and report error if `test_file` or `steps_file` is missing.
3. Identify test infrastructure gaps:
   - external calls (HTTP, DB, filesystem, subprocess) that MUST be patched for deterministic tests
   - shared fixtures required by multiple steps not yet in `conftest.py`
   - mock data or factory functions needed to drive `Given` steps
4. Add required test infrastructure to `conftest.py` or `tests/fixtures/`:
   - `@pytest.fixture` definitions for shared setup
   - `unittest.mock.patch` or `pytest-mock` `mocker.patch` decorators or context managers for external calls
   - mock response factories returning data matching the real external output shape
   - NEVER change any `Then` assertion, step text, or scenario body while doing this
5. Implement one failing scenario at a time in `src/`.
6. Return the implementation files, conftest changes, suppressed gold plating (each item with `item` and `reason`), and any surgical step-file corrections.

> The caller (agent) is responsible for running `/run-tests` to verify GREEN state. This skill NEVER invokes commands directly.

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
