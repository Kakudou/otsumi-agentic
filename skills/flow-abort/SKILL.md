---
name: flow-abort
description: "Abort a running or paused pipeline safely while preserving artifacts and recording the reason."
---

# Flow Abort

Stop a workflow without deleting evidence. Aborted state remains inspectable and replayable.

## Usage

`/flow-abort {feature-name} "{reason}"`

## Hard Rules

- NEVER delete artifacts.
- NEVER abort a closed pipeline without explicit force semantics.
- MUST record a reason.
- MUST log the abort event.

## Steps

1. Read `.otsumi/{feature-name}/pipeline.json`.
2. Confirm the workflow is running or paused.
3. Set `status` to `aborted`.
4. Store:
   - `aborted_at`
   - `abort_reason`
5. Log `pipeline.aborted`.
6. Report how to inspect or replay later.
