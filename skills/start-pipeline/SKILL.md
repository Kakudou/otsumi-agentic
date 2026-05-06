---
name: start-pipeline
description: Start a new pipeline for a given mode, language, and feature description.
---

You are receiving a `/start-pipeline` command.

## Hard Rules

- NEVER infer any selector from natural language alone.
- NEVER allow an unsupported `language_id` to proceed.
- NEVER overwrite an existing pipeline.
- NEVER combine multiple features into a single pipeline.
- NEVER proceed past validation if required flags (`--mode`, `--lang`, feature description) are absent.

## Usage

`/start-pipeline --mode <assisted|vibecoding> --lang <language-id> [--stages 4,5] [--pytest] <feature description>`

Shorthand:
- `/start-assisted ...` is equivalent to `/start-pipeline --mode assisted ...`
- `/start-vibecoding ...` is equivalent to `/start-pipeline --mode vibecoding ...`

### `--pytest` flag

Controls which test style Stage-3 uses. When omitted, the default is auto-detected.

| Flag | `test_style` stored | Stage-3 behaviour |
|------|--------------------|--------------------|
| `--pytest` | `"pytest"` | plain pytest · `_given_/_when_/_then_` helpers (no `.feature` files) |
| *(omitted)* | `"auto"` | Stage-3 auto-detects: pytest-bdd if `.feature` files / pytest-bdd imports exist, otherwise plain pytest |

## Purpose

Initialize one or more feature pipelines with explicit routing selectors and hand control to the first active stage.

## Activation Steps

1. Validate `--mode`, `--lang`, and the feature description. If nothing provided, inspect the project to suggest appropriate values based on the detected language and test framework.
2. Identify every distinct feature required to satisfy the user request. NEVER assume a single feature is sufficient; NEVER assume the description involves only one feature. If the description is vague, ask for clarification and suggest possible features.
3. Create a separate pipeline for each feature identified.
4. If multiple features were identified: MUST warn the user about potential shared-file conflicts. Features that will write to the same files (e.g., shared `conftest.py`, shared `src/` modules) MUST be pipelined sequentially, not in parallel. Present the dependency assessment before proceeding.

**Per-feature steps:**

1. Derive `<feature-name>` in kebab-case from the description.
2. NEVER overwrite an existing `.otsumi/<feature-name>/pipeline.json`.
3. Ensure `.otsumi/<feature-name>/` exists.
4. Validate the combination by checking that adapter skills exist for the given `language_id`. If no adapter is found for any required stage, halt and report which adapters are missing.
5. Determine the active stages from the description and mode:
   - Full feature delivery → all eight stages (`stage-01` through `stage-08`).
   - Quick fix (user says "fix" or requests minimal work) → `stage-04` and `stage-05` only.
   - Explicit `--stages` flag → use exactly those stages (e.g., `--stages 4,5`).
6. Resolve `test_style`:
   - `--pytest` provided → `"pytest"`
   - `--pytest` not provided → `"auto"` (Stage-3 will auto-detect at runtime)
7. Create `.otsumi/<feature-name>/pipeline.json`:

```json
{
  "feature_name": "<feature-name>",
  "mode": "<mode>",
  "language_id": "<language-id>",
  "test_style": "<pytest | auto>",
  "stages": ["stage-01", "stage-02", "..."],
  "started_at": "<ISO-8601>",
  "started_by": "/start-pipeline",
  "description": "<raw description>",
  "status": "<running if vibecoding, paused if assisted>",
  "current_stage": null,
  "last_completed_stage": null,
  "next_stage": "<first active stage>",
  "resumed_at": null
}
```

7. Log:
   - `/atomic-log <feature-name> pipeline.started "mode=<mode> language=<language-id> test_style=<test_style> next_stage=<first active stage>"`
8. **Assisted mode:** set `status` → `paused` and report that `/continue` will advance the pipeline to the next stage.
   **Vibecoding mode:** invoke the agent for the first active stage.
9. Execute the skill /env-setup
