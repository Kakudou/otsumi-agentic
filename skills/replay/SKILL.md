---
name: replay
description: Re-run a closed or aborted pipeline from a specific stage.
---

Replay a pipeline from a specific stage, reusing existing artifacts for all prior stages.

## Hard Rules

- NEVER replay a running or paused pipeline — use `/continue` instead.
- NEVER delete prior stage output files. Only overwrite stages being re-run.
- NEVER skip upstream validation. Every stage before the replay point MUST have its output JSON.
- The pipeline MUST resume normal flow after the replay stage completes — subsequent stages run in order per the pipeline's `stages` list.

## Usage

`/replay <feature-name> --from <stage-id>`

## Common Use Cases

- Re-run quality scoring after manual fixes
- Re-run implementation after changing the spec
- Re-generate documentation after code changes

## Steps

1. Validate `<feature-name>` and `--from <stage-id>`.
2. Read `.otsumi/<feature-name>/pipeline.json`. If missing: halt and report.
3. Confirm `status` is `closed` or `aborted`. If `running` or `paused`: halt — use `/continue` instead.
4. Confirm `<stage-id>` is present in the pipeline's `stages` list. If not: halt and report.
5. Confirm all stages prior to `<stage-id>` have output JSONs in `.otsumi/<feature-name>/`. If any are missing: halt and report which.
6. Present confirmation to User:

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

7. On `confirm`:
   - Update `.otsumi/<feature-name>/pipeline.json`:
     - `status` → `running`
     - `current_stage` → `null`
     - `last_completed_stage` → the stage immediately before `<stage-id>` in the pipeline's `stages` list, or `null` if replaying from the first stage
     - `next_stage` → `<stage-id>`
     - `resumed_at` → `<ISO-8601>`
   - Log via `/atomic-log <feature-name> pipeline.replayed "from=<stage-id> previous_status=<status>"`.
   - Invoke the agent for `<stage-id>`, or pause if mode is `assisted`.

8. On `cancel`: do nothing.