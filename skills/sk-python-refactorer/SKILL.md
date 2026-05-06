---
name: sk-python-refactorer
description: Improves Python source quality without changing behavior. Tests must stay GREEN after every atomic change.
---

# python-refactorer

One job: improve the Python implementation while keeping behavior fixed.

## Hard Rules

- NEVER batch multiple refactors into a single change.
- NEVER introduce new behavior, rewrite tests, or add speculative helpers.
- NEVER apply more than one atomic change per file at a time.
- ALWAYS abort and record any refactor that breaks GREEN.
- ALWAYS follow Clean Architecture with Port and Adapter patterns — UNLESS the existing codebase uses a different architectural style, in which case MUST follow that style.
- One module per file, named after the main class or function.

## Inputs

- `feature_name` — required
- `implementation_files` — required
- `test_file` — required
- `steps_file` — required
- `support_files` — optional list

## Outputs

- Modified implementation files
- `stage-05-result`

The caller (agent) runs `/run-tests` after each atomic change and writes `stage-05-output.json`. This skill does not invoke commands directly.

## Allowed Refactors

- Rename for domain clarity
- Extract duplication
- Split oversized functions
- Improve type hints on touched code

## Activation Steps

1. Read implementation and test artifacts.
2. Apply one atomic refactor at a time.
3. Return each atomic change to the caller for GREEN verification; abort and record any that do not preserve GREEN.
4. Return applied changes, aborted refactors, and deferred improvements.

## Result Schema

Return `stage-05-result` to the caller:

```text
applied_changes:
  - file: <path>
    change: <what was refactored>
    type: rename | extract | split | type_hint | cleanup
aborted_refactors:
  - file: <path>
    change: <what was attempted>
    reason: <why it was reverted — e.g. broke GREEN>
deferred_features:
  - description: <what improvement was identified but not applied>
    reason: <why it was deferred>
```

The caller writes the stage output JSON and manages logging.
