---
name: sk-python-test-generator
description: Generates RED pytest-bdd tests from an approved Gherkin file for `python + pytest-bdd`.
---

# python-test-generator

One job: write RED tests for `python + pytest-bdd`.

## Inputs

- `feature_name` - required
- `feature_file` - required

## Outputs

- `tests/test_<feature-name>.py`
- `tests/steps/<feature-name>_steps.py`
- `tests/conftest.py`
- `stage-03-result`

## Activation Steps

0. ensure you are using .venv.otsumi or invoke the command `/env-setup` to set up the environment before running this skill.
1. Read the approved feature file.
2. Reuse existing step definitions when the exact step text already exists.
3. Generate pytest-bdd scenario bindings, step definitions, and a typed `Context` fixture.
4. Identify external dependencies visible from the Gherkin steps (API calls, DB access, filesystem, subprocess). For each one, add a placeholder fixture or patch stub in `conftest.py` so Stage-4 has a clear hook to fill in:
   - Use `@pytest.fixture` stubs named after the dependency
   - Add a `# TODO: patch <target> in Stage-4` comment where a mock will be needed
   - Do not implement the mock itself, Stage-4 owns that
5. Make the suite fail for the right reason: missing imports, missing functions, or real assertions against absent outputs.
6. Return the Stage-3 result with artifact paths.

> The caller (agent) is responsible for invoking the command `/env-setup` before invoking this skill and for running `/run-tests` after to confirm RED state.

## Rules

- `Then` steps must use real asserts.
- Do not use `pass`, `assert True`, or `NotImplementedError` placeholders.
- Prefer Pydantic `BaseModel` for shared context and domain-shaped values.
- Scaffold `conftest.py` fixture stubs for visible external dependencies, Stage-4 fills in the implementation, Stage-3 marks the seam.

## Result Schema

Return `stage-03-result` to the caller:

```text
test_file: <path to tests/test_<feature-name>.py>
steps_file: <path to tests/steps/<feature-name>_steps.py>
support_files: [<paths to conftest.py and any other support files>]
tests_collected: <number of test scenarios generated>
red_reason: <why the tests fail: missing imports, missing functions, or absent outputs>
```

The caller writes the stage output JSON, confirms RED state via `/run-tests`, and handles logging.
