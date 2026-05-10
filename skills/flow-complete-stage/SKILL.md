---
name: flow-complete-stage
description: "Finalize a workflow stage by updating state, logging completion, resolving the next active stage, and applying pause/close behavior."
---

# Flow Complete Stage

Centralize state updates after one stage finishes. Records completion and computes the next routing state — does NOT perform the stage work.

## Usage

`/flow-complete-stage {feature-name} {stage-id} "{summary}"`

## Hard Rules

- NEVER mark a stage complete unless its required output artifact exists.
- MUST NOT mark a stage complete when pipeline status is not `running`. A pipeline in `paused`, `closed`, or `aborted` status cannot have stages completed — return a blocker with reason: contract_violation.
- NEVER skip active stages in the state root.
- NEVER invoke the next stage directly.
- NEVER close a pipeline with missing required active stage outputs.

## Steps

1. Read `.otsumi/{feature-name}/pipeline.json`.
2. Verify `{stage-id}` is in `stages`.
3. Verify the stage output exists when required:
   - `.otsumi/{feature-name}/stage-XX-output.json`
4. Set:
   - `last_completed_stage`
   - `current_stage`
   - `next_stage`
   - `status`
5. Apply mode/pause rules:
   - Evaluation order: check T4 first (no active stages remain → `closed`), then T3 guard (assisted mode + user gate → `paused`), then T2 (`running`)
   - pause at user approval gates
   - pause before user-owned stages
   - close when no active stages remain
   - When `mode=vibecoding`, NEVER set status to `paused`. Transition T3 (`running→paused`) MUST NOT fire when `mode=vibecoding`. Only T2 (`running→running`) or T4 (`running→closed`) are valid.
6. Log `stage.completed`.
7. Return state summary and next action.

## State Machine Reference

- This skill implements transitions T2 (`stage.completed` → `running`), T3 (`stage.completed` → `paused`), and T4 (`stage.completed` → `closed`).
- T3 is guarded: MUST NOT fire when `mode=vibecoding` (OQ-2).
