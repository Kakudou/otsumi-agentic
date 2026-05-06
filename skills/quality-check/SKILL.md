---
name: quality-check
description: Run language-aware code quality checks for a feature or the full project.
---

## Hard Rules

- MUST keep the report shape stable even when the underlying tools differ.
- NEVER merge dimensions together.
- NEVER skip a dimension silently.
- MUST resolve check commands from the project's conventions and language-appropriate defaults.

Run four quality dimensions through the resolved execution stack.

## Dimensions

| Dimension | What It Checks |
|-----------|----------------|
| `linting` | Static analysis warnings and errors |
| `formatting` | Code formatting compliance |
| `import_hygiene` | Import ordering and hygiene |
| `type_safety` | Type checking or equivalent |

Resolve concrete tools per dimension from project conventions and existing configs (e.g., `pyproject.toml`, `.flake8`, `mypy.ini`). Adapter skills carry language-specific tool defaults.

## Usage

- `/quality-check <feature-name>`
- `/quality-check --lang <language-id>`

## Steps

1. Resolve `language_id`:
   - feature-scoped: read `.otsumi/<feature-name>/pipeline.json` and extract `language_id`.
   - full-project: require explicit `--lang`. Halt and report if absent.
2. Resolve the four check commands from the project's conventions and the adapter skills' knowledge of the language's standard quality tools.
3. Run each of the four checks against the feature scope or the explicit full-project scope.
4. Report one section per dimension: tool name, pass/fail status, and raw failing output.
5. If feature-scoped, log the check run through `/atomic-log`.