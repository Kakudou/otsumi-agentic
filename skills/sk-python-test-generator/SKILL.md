---
name: sk-python-test-generator
description: Generates RED tests for Python projects. Defaults to pytest-bdd (Gherkin-driven). Use --pytest for raw pytest with _given_/_when_/_then_ BDD helper pattern.
---

# python-test-generator

One job: write RED tests for Python. Two modes: `pytest-bdd` (default) or `--pytest` (raw pytest BDD helper pattern).

## Usage

```
/sk-python-test-generator <feature_name> [--pytest]
```

- No flag → **pytest-bdd mode**: Gherkin `.feature` file required, generates scenario bindings + step definitions.
- `--pytest` flag → **raw pytest mode**: no `.feature` file; generates tests using `_given_*` / `_when_*` / `_then_*` helper functions matching the project's existing test conventions.

## Mode resolution when invoked from a pipeline

When called via `sk-stage-router` rather than directly, resolve the mode from the `test_style` field passed through from `pipeline.json`:

| `test_style` value | Mode |
|--------------------|------|
| `"pytest"` | raw pytest (`--pytest`) |
| `"auto"` or absent | auto-detect: scan `tests/` for `.feature` files and `pytest-bdd` imports; use pytest-bdd if found, raw pytest otherwise |

This is the only place in the pipeline where `test_style` is interpreted. `bdd-orchestrator` and `sk-stage-router` treat it as opaque.

---

## Mode A — pytest-bdd (default, no flag)

### Inputs
- `feature_name` — required
- `feature_file` — required (approved `.feature` file from Stage-2)

### Outputs
- `tests/test_<feature-name>.py`
- `tests/steps/<feature-name>_steps.py`
- `tests/conftest.py`
- `stage-03-result`

### Activation Steps

0. Ensure `.venv.otsumi` is active or invoke `/env-setup` first.
1. Read the approved feature file.
2. Reuse existing step definitions when the exact step text already exists.
3. Generate pytest-bdd scenario bindings, step definitions, and a typed `Context` fixture.
4. Identify external dependencies visible from the Gherkin steps (API calls, DB access, filesystem, subprocess). For each one, add a placeholder fixture or patch stub in `conftest.py`:
   - Use `@pytest.fixture` stubs named after the dependency
   - Add a `# TODO: patch <target> in Stage-4` comment where a mock will be needed
   - Do not implement the mock itself — Stage-4 owns that
5. Make the suite fail for the right reason: missing imports, missing functions, or real assertions against absent outputs.
6. Return the Stage-3 result with artifact paths.

### Rules
- `Then` steps must use real asserts.
- Do not use `pass`, `assert True`, or `NotImplementedError` placeholders.
- Prefer Pydantic `BaseModel` for shared context and domain-shaped values.
- Scaffold `conftest.py` fixture stubs for visible external dependencies.

---

## Mode B — raw pytest BDD helper pattern (`--pytest` flag)

Use this mode when the project uses plain `pytest` with `_given_*` / `_when_*` / `_then_*` helper functions instead of pytest-bdd. No `.feature` file is needed or produced.

### Inputs
- `feature_name` — required
- `affected_source_files` — required (list the mapper/module paths under test)

### Outputs
- One or more new test files under `tests/` mirroring the existing test directory structure
- `stage-03-result`

### Activation Steps

0. Ensure `.venv.otsumi` is active or invoke `/env-setup` first.
1. **Scan existing test conventions** — read at least two test files from the area closest to `affected_source_files`:
   - Note exact `_given_*` / `_when_*` / `_then_*` naming patterns
   - Note factory library in use (`polyfactory`, `factory_boy`, etc.) and `ModelFactory` subclass patterns
   - Note fixture style (`@pytest.fixture`, inline, etc.)
   - Note decorators: `@pytest.mark.order(N)`, `# noqa: S101` on assertions, etc.
2. **Determine test file placement** — mirror the existing test directory structure. Never put tests in a new location when an equivalent existing directory exists.
3. **Write tests** following exact project conventions:
   - One `_given_<scenario>(...)` setup function per test scenario
   - One `_when_<action>(...)` invocation function per action
   - One or more `_then_<assertion>(...)` assertion functions per expected outcome
   - `@pytest.mark.order(1)` on all test functions
   - `# noqa: S101` on all `assert` statements
   - Use project's factory library for all data generation (do not use raw `dict` or hardcoded values when factories exist)
4. **Confirm RED state** — every generated test must FAIL against the current (unmodified) source code for the right reason (wrong value, not missing import). Document the failure reason.
5. **Never modify existing test files** — only create new ones.
6. **Never introduce pytest-bdd**, `.feature` files, or any test library not already present in the project.
7. Return the Stage-3 result.

### Rules
- All assertions must be real (`assert x is None`, `assert x == expected`) — no placeholders.
- Tests must fail RED against current source for the *correct* reason (wrong behavior, not import error).
- Match the project's naming convention exactly — do not invent new patterns.
- `polyfactory` `ModelFactory` subclasses must be defined per-file following the project's factory class style.

---

## Result Schema (both modes)

Return `stage-03-result` to the caller:

```text
mode: pytest-bdd | raw-pytest
test_files: [<list of created test file paths>]
tests_collected: <number of test functions generated>
red_reason: <why the tests currently fail: wrong value / missing function / etc.>
red_confirmed: true | false
```

The caller writes the stage output JSON, confirms RED state via `/run-tests`, and handles logging.
