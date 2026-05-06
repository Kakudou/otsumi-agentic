---
name: run-tests
description: Run language-aware tests for a feature or the full suite.
---

Run the test suite through the project's detected test framework.

## Hard Rules

- MUST keep report shape stable across languages.
- NEVER fake RED/GREEN state.
- ALWAYS resolve test paths and runner commands from the project structure and adapter skill knowledge — NEVER from hardcoded assumptions.
- This skill MAY be invoked even when the pipeline's stages list does not include test stages.

## Usage

- `/run-tests <feature-name>`
- `/run-tests --lang <language-id>`

## Steps

1. Resolve routing inputs:
   - feature-scoped: read `.otsumi/<feature-name>/pipeline.json`, extract `language_id` and `stages`
   - full-suite: MUST have explicit `--lang`
2. MUST run `/env-setup` if the runtime is not ready.
3. Resolve test paths and runner from project structure and adapter skills:
   - Scan for test files related to the feature (test bindings, step files, feature files).
   - Discover the test framework from project configuration (e.g., `pyproject.toml`, `package.json`, `Cargo.toml`).
   - Apply adapter skills' knowledge of the test framework's conventions.
   - Full-suite mode: use explicit `--lang` flag, run against the project's test directory.
4. Determine test target:
   - feature-scoped: resolve the binding or test file from the project's test layout, substituting `<feature-name>` with the actual feature name
   - full-suite: use `tests/` or the project's default suite target
5. Execute the resolved runner command against the target.
6. Report:
   - tests collected
   - passed / failed / errored counts
   - each failing test or scenario name with short reason
   - overall status: RED or GREEN
7. If feature-scoped: append the result to `/atomic-log`.