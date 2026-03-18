---
name: continue
description: Resume an assisted pipeline from its paused state.
---

You are receiving a `/continue` command.

## Usage

`/continue <feature-name>`

## Purpose

Advance an assisted pipeline from its paused state. When the next active stage is Stage-04 or Stage-05 in assisted mode, `/continue` does not just trust the user blindly: it validates the user's implementation or refactor work, materializes the missing stage output JSON files, and only then advances.

## Steps

1. Validate the feature name.
2. Read `.otsumi/<feature-name>/pipeline.json`.
3. Confirm:
   - `mode` is `assisted`
   - `status` is `paused`. If `status` is `closed`, halt and report: "Pipeline `<feature-name>` is already closed. No action taken." If `status` is `running`, halt and report: "Pipeline `<feature-name>` is already running at `<current_stage>`. Use `/continue` only when the pipeline is paused."
   - `language_id` exists
   - `stages` list exists and is non-empty
   - `next_stage` exists or the pipeline can be closed
4. Read the `stages` list from `pipeline.json`. This is the ordered list of active stages for the pipeline.

## Assisted Mode: Implementation & Refactor Validation

In assisted mode, Stage-04 and Stage-05 are where the user implements and refactors. If `next_stage` is one of these stages, validate the user's work and materialize the stage output before advancing.

If `next_stage` begins a contiguous block of Stage-04 / Stage-05:

1. Materialize every such stage in order before advancing.

### Stage-04 in assisted mode

If `stage-04` is the next stage and present in the `stages` list:

1. Resolve implementation files from the project structure. Scan for source files related to the feature (e.g., modules matching the feature name, new files in the source directory).
2. If Stage-03 is present upstream in the `stages` list, run `/run-tests <feature-name>` and require `GREEN`.
3. If no upstream test stage exists in the `stages` list, require explicit user confirmation that the implementation block is complete and record that no automated GREEN proof exists for this stage.
4. If no implementation files exist, halt and report.
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

6. Append to `/atomic-log`:
   - `stage-04.completed` with `completed_by=user`

### Stage-05 in assisted mode

If `stage-05` is the next stage and present in the `stages` list:

1. Require `.otsumi/<feature-name>/stage-04-output.json`.
2. If tests exist for the project's test suite, run `/run-tests <feature-name>` and require `GREEN`.
3. If no tests exist for the project's test suite, require explicit user confirmation that refactor work is complete and record that no automated pass proof exists for this stage.
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

5. Append to `/atomic-log`:
   - `stage-05.completed` with `completed_by=user`

6. Update `last_completed_stage` to the final materialized stage.
7. Advance `next_stage` to the next active stage in the `stages` list ordering, or `null` if the list is exhausted.

## Final Resume Step

After materialization (if any), or if `next_stage` is not Stage-04/Stage-05:

1. If no active stages remain:
   - set `status` -> `closed`
   - set `current_stage` -> `null`
   - set `next_stage` -> `null`
2. Otherwise:
   - set `status` -> `running`
   - set `current_stage` -> `<resolved next stage>`
   - set `resumed_at` -> `<ISO-8601>`
3. Log:
   - `/atomic-log <feature-name> pipeline.continued "language=<language_id> next_stage=<stage-or-null>"`
4. Invoke the resolved next stage agent, or report pipeline closure.

## Hard Rules

- Never resume by repo inspection alone; use the `stages` list ordering and explicit validation.
- Never jump over an active stage silently.
- Never advance past Stage-04 or Stage-05 without materializing its stage output JSON.