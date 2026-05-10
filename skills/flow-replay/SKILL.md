---
name: flow-replay
description: "Replay a closed or aborted pipeline from a selected stage while preserving upstream artifacts and requiring overwrite confirmation."
---

# Flow Replay

Re-run downstream workflow stages while preserving earlier accepted artifacts.

## Usage

`/flow-replay {feature-name} --from {stage-id}`

## Hard Rules

- NEVER replay a running pipeline.
- NEVER overwrite downstream artifacts without explicit confirmation.
- NEVER mutate upstream artifacts before the replay point.
- MUST record replay origin and replay point.
- MUST treat replay as creation of a new pipeline instance.
- MUST NOT write `status: running` back into the original terminal pipeline file.

## Replay Semantics

- Replay MUST NOT be modeled as a state transition from `closed` or `aborted`.
- Replay creates a NEW pipeline instance. The original `pipeline.json` stays in its terminal state permanently.
- Replay operation flow:
  1. Read the terminal pipeline configuration.
  2. Identify the replay `--from` stage.
  3. Require explicit user confirmation for downstream overwrite.
  4. Create a NEW `pipeline.json` with `status: running`.
  5. Preserve upstream artifacts.
  6. Record `replayed_from` metadata.
- The original `pipeline.json` MUST NOT be mutated. Its terminal state is a permanent record.
- Skills MUST NOT implement replay as `status: running` written back into the original pipeline file.

### Replay Metadata Schema

`replayed_from` metadata MUST include all of the following required fields:

- `original_pipeline_id` (string)
- `replayed_from_stage` (string)
- `original_status` (enum: `closed` | `aborted`)
- `replay_timestamp` (ISO 8601 string)

## State Machine Reference

Replay is explicitly NOT a state transition in the S4 state machine spec. Closed/aborted pipelines remain terminal; replay starts a new pipeline instance that references the original.

## Steps

1. Read `.otsumi/{feature-name}/pipeline.json`.
2. Confirm status is `closed` or `aborted`.
3. Validate `--from` is an active stage.
4. List downstream artifacts that will be overwritten.
5. Request explicit confirmation.
6. Create a new pipeline instance as `running`, preserving upstream artifacts and recording replay metadata.
7. Return the next stage to execute.
