---
name: continue
description: Resume an assisted pipeline from its paused state.
---

You are receiving a `/continue` command.

## Usage

`/continue <feature-name>`

## Hard Rules

- NEVER resume by repo inspection alone. ALWAYS use the `stages` list ordering and explicit validation.
- NEVER jump over an active stage silently.
- NEVER advance past Stage-04 or Stage-05 without materializing its stage output JSON.
- NEVER advance past Stage-04 without GREEN tests when an upstream test stage exists in `stages`.

## Purpose

Advance an assisted pipeline from its paused state. When `next_stage` is Stage-04 or Stage-05, validate user work, materialize the missing stage output JSON files, and only then advance.

## Steps

### 1. Validate and Load

1. Validate the feature name — halt if empty.
2. Read `.otsumi/<feature-name>/pipeline.json`.
3. Confirm — halt with the stated message if any check fails:
   - `mode` MUST be `assisted`
   - `status` MUST be `paused`. If `closed`: halt — "Pipeline `<feature-name>` is already closed. No action taken." If `running`: halt — "Pipeline `<feature-name>` is already running at `<current_stage>`. Use `/continue` only when the pipeline is paused."
   - `language_id` MUST exist
   - `stages` MUST exist and be non-empty
   - `next_stage` MUST exist or the pipeline MUST be closeable

### 2. Assisted Mode: Materialize Stage-04 / Stage-05

If `next_stage` begins a contiguous block of Stage-04/Stage-05, materialize every such stage in order before advancing.

#### Stage-04 Materialization

If `stage-04` is next and present in `stages`:

1. Resolve implementation files — scan for source files related to the feature.
2. If Stage-03 is upstream in `stages`: run `/run-tests <feature-name>` — MUST be `GREEN` to proceed.
3. If no upstream test stage: require explicit user confirmation; record that no automated GREEN proof exists.
4. If no implementation files exist: halt and report.
5. Write `.otsumi/<feature-name>/stage-04-output.json`:

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-04",
  "language_id": "<language-id>",
  "completed_by": "user",
  "completion_source": "/continue",
  "implementation_files": ["<paths>"],
  "green_state_confirmed": "true | null",
  "verification_method": "/run-tests <feature-name> | user confirmation when no upstream test stage exists",
  "verified_at": "<ISO-8601>"
}
```

6. Append to `/atomic-log`: `stage-04.completed` with `completed_by=user`.

#### Stage-05 Materialization

If `stage-05` is next and present in `stages`:

1. MUST have `.otsumi/<feature-name>/stage-04-output.json` — halt if missing.
2. If tests exist: run `/run-tests <feature-name>` — MUST be `GREEN` to proceed.
3. If no tests: require explicit user confirmation; record that no automated pass proof exists.
4. Write `.otsumi/<feature-name>/stage-05-output.json`:

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-05",
  "language_id": "<language-id>",
  "completed_by": "user",
  "completion_source": "/continue",
  "implementation_files": ["<paths>"],
  "tests_still_pass": "true | null",
  "verification_method": "/run-tests <feature-name> | user confirmation when no tests exist",
  "verified_at": "<ISO-8601>"
}
```

5. Append to `/atomic-log`: `stage-05.completed` with `completed_by=user`.
6. Update `last_completed_stage` to the final materialized stage.
7. Advance `next_stage` to the next active stage in `stages`, or `null` if exhausted.

### 3. Final Resume

After materialization (if any), or if `next_stage` is not Stage-04/Stage-05:

1. If no active stages remain:
   - set `status` → `closed`, `current_stage` → `null`, `next_stage` → `null`
2. Otherwise:
   - set `status` → `running`, `current_stage` → `<resolved next stage>`, `resumed_at` → `<ISO-8601>`
3. Log: `/atomic-log <feature-name> pipeline.continued "language=<language_id> next_stage=<stage-or-null>"`
4. Invoke the resolved next stage agent, or report pipeline closure.