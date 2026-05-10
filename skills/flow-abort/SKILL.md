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
- MUST halt with error when pipeline status is `closed`.
- MUST halt with error when pipeline status is `aborted`.
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

## State Machine Reference

- This skill implements transitions T7 (`running` → `aborted`) and T8 (`paused` → `aborted`).
- Sole precondition: `abort_reason` is provided (non-empty string).
- No override path and no terminal-state bypass.
- Aborting a `closed` pipeline: MUST halt with error.
- Aborting an `aborted` pipeline: MUST halt with error.
