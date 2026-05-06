---
name: flow-complete-stage
description: "Finalize a workflow stage by updating state, logging completion, resolving the next active stage, and applying pause/close behavior."
---

# Flow Complete Stage

Centralize state updates after one stage finishes. Records completion and computes the next routing state — does NOT perform the stage work.

## Usage

`/flow-complete-stage <feature-name> <stage-id> "<summary>"`

## Hard Rules

- NEVER mark a stage complete unless its required output artifact exists.
- NEVER skip active stages in the state root.
- NEVER invoke the next stage directly.
- NEVER close a pipeline with missing required active stage outputs.

## Steps

1. Read `.otsumi/<feature-name>/pipeline.json`.
2. Verify `<stage-id>` is in `stages`.
3. Verify the stage output exists when required:
   - `.otsumi/<feature-name>/stage-XX-output.json`
4. Set:
   - `last_completed_stage`
   - `current_stage`
   - `next_stage`
   - `status`
5. Apply mode/pause rules:
   - pause at user approval gates
   - pause before user-owned stages
   - close when no active stages remain
6. Log `stage.completed`.
7. Return state summary and next action.
