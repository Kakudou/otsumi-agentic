---
name: run-tests
description: Run language-aware tests for a feature or the full suite.
---

Run the test suite through the project's detected test framework.

## Usage

- `/run-tests <feature-name>`
- `/run-tests --lang <language-id>`

## Steps

1. Resolve routing inputs:
   - feature-scoped: read `.otsumi/<feature-name>/pipeline.json` and extract `language_id` and `stages`
   - full-suite: require explicit `--lang`
2. Run `/env-setup` if the runtime is not ready.
3. Resolve test paths and runner from the project structure and adapter skills:
   - Scan the project for test files related to the feature (test bindings, step files, feature files).
   - Discover the test framework from project configuration (e.g., `pyproject.toml`, `package.json`, `Cargo.toml`).
   - Use the adapter skills' knowledge of the test framework's conventions.
   - For full-suite mode: use the explicit `--lang` flag and run against the project's test directory.
4. Determine the test target:
   - feature-scoped: resolve the binding file or test file from the project's test layout, substituting `<feature-name>` with the actual feature name
   - full-suite: use `tests/` or the project's default suite target
5. Execute the resolved runner command against the target.
6. Report:
   - tests collected
   - passed / failed / errored counts
   - each failing test or scenario name with short reason
   - overall status: RED or GREEN
7. If feature-scoped, append the result to `/atomic-log`.

## Hard Rules

- Keep report shape stable across languages.
- Never fake RED/GREEN state.
- Always resolve test paths and runner commands from the project structure and adapter skill knowledge, not from hardcoded assumptions.
- This command may still be used even when the pipeline's stages list does not include test stages.