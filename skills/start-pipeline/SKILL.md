---
name: start-pipeline
description: Start a new pipeline for a given mode, language, and feature description.
---

You are receiving a `/start-pipeline` command.

## Usage

`/start-pipeline --mode <assisted|vibecoding> --lang <language-id> [--stages 4,5] <feature description>`

Shorthand:
- `/start-assisted ...` is equivalent to `/start-pipeline --mode assisted ...`
- `/start-vibecoding ...` is equivalent to `/start-pipeline --mode vibecoding ...`

## Purpose

Initialize one or more feature pipelines with explicit routing selectors and hand control to the first active stage.

## Activation Steps

1. Validate `--mode`, `--lang`, and the feature description. If nothing provided, inspect the project to suggest appropriate values based on the detected language and test framework.
2. From the description identify the different features involved to satisfy the user request. Never assume a single feature is sufficient to satisfy the request, and never assume the description only involves a single feature. If the description is vague, ask for clarification and suggest possible features.
3. For each feature identified, create a separate pipeline. Never combine multiple features into a single pipeline.
4. If multiple features were identified: warn User about potential shared-file conflicts. Features that will write to the same files (e.g., shared `conftest.py`, shared `src/` modules) should be pipelined sequentially, not in parallel. Present the dependency assessment before proceeding.

**Per-feature steps:**

1. Derive `<feature-name>` in kebab-case from the description.
2. Refuse to overwrite an existing `.otsumi/<feature-name>/pipeline.json`.
3. Ensure `.otsumi/<feature-name>/` exists.
4. Validate the combination by checking that adapter skills exist for the given `language_id`. If no adapter is found for any required stage, halt and report which adapters are missing.
5. Determine the active stages from the description and mode:
   - Full feature delivery → all eight stages (`stage-01` through `stage-08`).
   - Quick fix (user says "fix" or requests minimal work) → `stage-04` and `stage-05` only.
   - Explicit `--stages` flag → use exactly those stages (e.g., `--stages 4,5`).
6. Create `.otsumi/<feature-name>/pipeline.json`:

```json
{
  "feature_name": "<feature-name>",
  "mode": "<mode>",
  "language_id": "<language-id>",
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
   - `/atomic-log <feature-name> pipeline.started "mode=<mode> language=<language-id> next_stage=<first active stage>"`
8. **Assisted mode:** set `status` → `paused` and report that `/continue` will advance the pipeline to the next stage.
   **Vibecoding mode:** invoke the agent for the first active stage.

## Hard Rules

- Never infer any selector from natural language alone.
- Never allow an unsupported `language_id` to proceed.
- Never overwrite an existing pipeline.
