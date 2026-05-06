---
name: abort
description: Gracefully abort a running or paused pipeline.
---

Abort a pipeline that is running or paused. Records the abort cleanly and preserves all existing artifacts.

## Usage

`/abort <feature-name>`
`/abort <feature-name> --reason "<reason>"`

## Steps

1. Validate `<feature-name>`.
2. Read `.otsumi/<feature-name>/pipeline.json`.
3. If the file does not exist: halt and report that no pipeline exists for this feature name.
4. Confirm `status` is `running` or `paused`. If `status` is `closed`: halt and report that the pipeline is already closed. If `status` is `aborted`: halt and report that the pipeline is already aborted.
5. STOP and present this confirmation prompt to User before any writes:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Abort Pipeline · <feature-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Current status:     <status>
Current stage:      <current_stage>
Last completed:     <last_completed_stage>

Artifacts preserved:
  • <list existing stage-XX-output.json files>
  • <list feature files, test files, src files>

This will set the pipeline status to `aborted`. All existing artifacts remain untouched.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: confirm / cancel
```

6. On `confirm`:
   - Update `.otsumi/<feature-name>/pipeline.json`:
     - `status` → `aborted`
     - `current_stage` → `null`
     - `next_stage` → `null`
   - Log via Invoking the Command `/atomic-log <feature-name> pipeline.aborted "reason=<reason or 'user-requested'> last_completed=<last_completed_stage>"`.
   - Report the abort with a summary of what was completed and what remains.

7. On `cancel`: do nothing, report that the abort was cancelled.

## Hard Rules

- Never abort without explicit User confirmation.
- Never delete any artifacts. Abort preserves everything.
- Never abort a closed pipeline. Closed is final.
- A pipeline can be restarted after abort by invoking the Command `/start-pipeline` with a different feature name, or by manually resetting `pipeline.json` status to `paused` and using `/continue`.
