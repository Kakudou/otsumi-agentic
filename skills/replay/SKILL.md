---
name: replay
description: Re-run a closed or aborted pipeline from a specific stage.
---

Replay a pipeline from a specific stage, reusing existing artifacts for all prior stages.

## Usage

`/replay <feature-name> --from <stage-id>`

## Purpose

When you want to re-run part of a pipeline without starting from scratch. Common cases:
- re-run quality scoring after manual fixes
- re-run implementation after changing the spec
- re-generate documentation after code changes

## Steps

1. Validate `<feature-name>` and `--from <stage-id>`.
2. Read `.otsumi/<feature-name>/pipeline.json`.
3. If the file does not exist: halt and report.
4. Confirm `status` is `closed` or `aborted`. If `status` is `running` or `paused`: halt and report that the pipeline is still active — use `/continue` instead.
5. Confirm that `<stage-id>` is present in the pipeline's `stages` list. If the stage is not in the list: halt and report that the stage is not part of this pipeline.
6. Confirm that all stages prior to `<stage-id>` (per the pipeline's `stages` list ordering) have output JSONs in `.otsumi/<feature-name>/`. If any required upstream stage is missing: halt and report which stage output is missing.
7. Ask User for confirmation:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Replay Pipeline · <feature-name> from <stage-id>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Previous status:    <closed | aborted>
Replay from:        <stage-id>
Reusing:            <list of prior stage outputs that will be kept>
Will re-run:        <list of stages from <stage-id> onward per the pipeline's stages list>

Existing output files for re-run stages will be overwritten.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: confirm / cancel
```

8. On `confirm`:
   - Update `.otsumi/<feature-name>/pipeline.json`:
     - `status` → `running`
     - `current_stage` → `null`
     - `last_completed_stage` → the stage immediately before `<stage-id>` in the pipeline's `stages` list, or `null` if replaying from the first stage
     - `next_stage` → `<stage-id>`
     - `resumed_at` → `<ISO-8601>`
   - Log via `/atomic-log <feature-name> pipeline.replayed "from=<stage-id> previous_status=<status>"`.
   - Invoke the agent for `<stage-id>`, or pause if mode is `assisted`.

9. On `cancel`: do nothing.

## Hard Rules

- Never replay a running or paused pipeline. Use `/continue` for those.
- Never delete prior stage output files. Only overwrite the stages being re-run.
- Never skip upstream validation. Every stage before the replay point must have its output.
- The pipeline resumes normal flow after the replay stage completes — subsequent stages run in order per the pipeline's `stages` list.