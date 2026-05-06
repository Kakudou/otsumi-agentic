---
name: dev-quality-check
description: "Run tool-backed development quality checks: linting, formatting, import hygiene, and type safety."
---

# Dev Quality Check

Collect objective quality evidence before final scoring or delivery review.

## Usage

`/dev-quality-check <feature-name>`

## Hard Rules

- NEVER mark a dimension passed if its command failed.
- NEVER invent a command the project does not support.
- NEVER hide missing tool configuration.
- A missing tool is a warning or failure per the workflow threshold.

## Dimensions

| Dimension | Meaning |
|---|---|
| `linting` | code smell and rule violations |
| `formatting` | formatter compliance |
| `import_hygiene` | import ordering, unused imports, dependency hygiene |
| `type_safety` | static type checks where supported |

## Steps

1. Ensure runtime readiness.
2. Resolve commands from project config.
3. Run each available dimension.
4. Report pass/fail/warning with evidence.
