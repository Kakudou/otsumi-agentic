---
name: dev-python-test-generator
description: "Generate Python RED tests from approved behavior using pytest-bdd or plain pytest helper style."
---

# Dev Python Test Generator

Generate Python tests that start in RED from approved behavior before implementation.

## Usage

`/dev-python-test-generator {payload}`

## Mission

Turn approved behavior into failing (RED) Python tests only. Keep tests behavior-anchored, convention-aligned, and ready for later implementation stages.

## Hard Rules

- NEVER write placeholder tests.
- NEVER fake RED by importing impossible modules unless that is the accepted surface contract.
- NEVER skip caller RED confirmation.
- MUST respect `test_style`.

## Modes

| `test_style` | Behavior |
|---|---|
| `auto` | Use pytest-bdd when feature files or pytest-bdd conventions exist; otherwise plain pytest |
| `pytest` | Use plain pytest with `_given/_when/_then` helper organization |

## Execution Sequence (All Modes)

1. Read approved scenarios or behavior contract.
2. Detect existing test layout.
3. Generate tests in the appropriate location.
4. Keep tests behavior-focused.
5. Return written files and the expected RED reason.

## Output Contract

Return the result object in **Result Schema** format exactly.

## Mode A Activation (pytest-bdd, Fuhyō execution)

1. Ensure `.venv.otsumi` is active before generation.
2. Read the approved `.feature` file for the behavior under test.
3. Reuse existing step definitions only when the step text is an exact match.
4. Generate scenario bindings, step definitions, and a typed `Context` fixture.
5. Identify visible external dependencies from steps and scaffold `conftest.py` placeholder `@pytest.fixture` stubs with `# TODO: patch <target> in Stage-4` comments.
6. Ensure the generated suite fails RED for the correct reason and return artifact paths plus RED cause.

### Mode A Rules

- `Then` steps MUST use real assertions.
- NEVER use `pass`, `assert True`, or `NotImplementedError` placeholders.
- MUST prefer Pydantic `BaseModel` for shared context payloads.
- MUST scaffold `conftest.py` fixture stubs for visible external dependencies.

## Mode B Activation (raw pytest helper style, Fuhyō execution)

1. Scan existing test conventions in nearby files (naming patterns, factory usage, fixture style, decorators such as `@pytest.mark.order(N)` and `# noqa: S101`).
2. Determine placement by mirroring existing `tests/` structure for the affected source area.
3. Write new tests using the established raw pytest helper style.
4. Keep inputs and behavior-bound in `_given_*` helpers.
4. Keep code execution and behavior-bound in `_when_*` helpers.
4. Keep assertions real and behavior-bound in `_then_*` helpers.
5. Confirm RED state against current source and capture the concrete failure reason.
6. Do not modify existing test files; only add new files for this stage.
7. Never introduce pytest-bdd artifacts in Mode B.

### Mode B Rules

- MUST use real assertions only; no placeholders.
- MUST fail RED for the correct reason (wrong behavior, not import error).
- MUST match project naming conventions exactly.
- MUST define `polyfactory` `ModelFactory` subclasses per-file when that convention exists.

## Conftest Scaffolding Guidance

- When external dependencies are visible (API clients, DB adapters, filesystem access, subprocess calls), add placeholder fixtures in `tests/conftest.py`.
- Use `@pytest.fixture` names that clearly map to dependency targets.
- Every placeholder must include `# TODO: patch <target> in Stage-4`.
- Do not implement real patching in this stage.

## Result Schema

Return this result object:

```text
mode: pytest-bdd | raw-pytest
test_files: [<created test file paths>]
tests_collected: <number of generated test functions>
red_reason: <why tests fail now>
red_confirmed: true | false
```

## Completion Criteria

- Tests are newly created (no edits to existing tests for this stage).
- RED state is real and explicitly explained.
- Report uses the exact schema above.
