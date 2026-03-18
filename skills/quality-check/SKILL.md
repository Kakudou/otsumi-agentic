---
name: quality-check
description: Run language-aware code quality checks for a feature or the full project.
---

Run the four tool-backed quality dimensions through the selected execution stack.

## Dimensions

| Dimension | What It Checks |
|-----------|----------------|
| `linting` | Static analysis warnings and errors |
| `formatting` | Code formatting compliance |
| `import_hygiene` | Import ordering and hygiene |
| `type_safety` | Type checking or equivalent |

The concrete tools for each dimension are resolved from the project's language conventions and existing tool configurations (e.g., `pyproject.toml`, `.flake8`, `mypy.ini`). The Pre-Pipeline Context Protocol identifies installed tools. Adapter skills carry the default tool knowledge for their language.

## Usage

- `/quality-check <feature-name>`
- `/quality-check --lang <language-id>`

## Steps

1. Resolve `language_id`:
   - feature-scoped: read `.otsumi/<feature-name>/pipeline.json` and extract `language_id`.
   - full-project: require explicit `--lang`.
2. Resolve the four check commands from the project's conventions and the adapter skills' knowledge of the language's standard quality tools.
3. Run each of the four checks against the feature scope or the explicit full-project scope.
4. Report one section per dimension with the tool name, pass/fail status, and raw failing output.
5. If feature-scoped, log the check run through `/atomic-log`.

## Hard Rules

- Keep the report shape stable even when the underlying tools differ.
- Never merge dimensions together.
- Never skip a dimension silently.
- Always resolve check commands from the project's conventions and language-appropriate defaults.