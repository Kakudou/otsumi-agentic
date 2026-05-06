---
name: flow-replay
description: "Replay a closed or aborted pipeline from a selected stage while preserving upstream artifacts and requiring overwrite confirmation."
---

# Flow Replay

Replay workflow stages from a chosen point.

## Usage

`/flow-replay <feature-name> --from <stage-id>`

## Purpose

Re-run downstream work while preserving earlier accepted artifacts.

## Hard Rules

- NEVER replay a running pipeline.
- NEVER overwrite downstream artifacts without explicit confirmation.
- NEVER mutate upstream artifacts before the replay point.
- ALWAYS record replay origin and replay point.

## Steps

1. Read `.otsumi/<feature-name>/pipeline.json`.
2. Confirm status is `closed` or `aborted`.
3. Validate `--from` is an active stage.
4. List downstream artifacts that will be overwritten.
5. Request explicit confirmation.
6. Mark pipeline `running` with replay metadata.
7. Return the next stage to execute.
