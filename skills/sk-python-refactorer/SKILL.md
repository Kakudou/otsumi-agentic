---
name: sk-python-refactorer
description: Improves Python source quality without changing behavior. Tests must stay GREEN after every atomic change.
---

# python-refactorer

One job: improve the Python implementation while keeping behavior fixed.

## Inputs

- `feature_name` - required
- `implementation_files` - required
- `test_file` - required
- `steps_file` - required
- `support_files` - optional list

## Outputs

- Modified implementation files
- `stage-05-result`

The caller (agent) is responsible for running `/run-tests` after each change and writing `stage-05-output.json`.

## Activation Steps

1. Read implementation and test artifacts.
2. Apply one atomic refactor at a time.
3. Return each atomic change for the caller to verify, and abort any that do not preserve GREEN.
4. Return applied changes, aborted refactors, and deferred features.

> The caller (agent) is responsible for running `/run-tests` after each atomic change to verify GREEN state. This skill does not invoke commands directly.

## Rules

- Allowed: rename for domain clarity, extract duplication, split oversized functions, improve touched type hints.
- Forbidden: new behavior, test rewrites, speculative helpers, batched refactors.
- One module per file, named after the main class or function.
- NEVER do it all in a single file.
- Try to respect Clean Architecture principles with Port and Adapter patterns. Except if the existing codebase use another architectural style, in that case follow the existing style.

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
