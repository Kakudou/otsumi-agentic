---
name: complete-stage
description: Finalize a stage by updating pipeline routing, logging completion, and handling pause/close transitions.
---

You are receiving a `/complete-stage` command.

## Usage

`/complete-stage <feature-name> <stage-id> "<log-details>"`

## Hard Rules

- Stage ordering MUST come from the `stages` list in `pipeline.json`. NEVER use stage-id arithmetic.
- NEVER set `status` to `closed` when active stages remain.
- NEVER modify the stage output JSON. This command only touches `pipeline.json` and the atomic log.

## Purpose

Updates pipeline routing, logs completion, and resolves whether the pipeline pauses, continues, or closes after a stage agent writes its output JSON.

## Steps

1. **Validate inputs — halt if any check fails:**
   - `$1` (feature-name) MUST not be empty.
   - `$2` (stage-id) MUST be a valid stage identifier (`stage-01` through `stage-08`).
   - `$3` (log-details) MUST not be empty. The caller MUST provide a meaningful summary built from stage output data.

2. **Read pipeline state:**
   - Read `.otsumi/$1/pipeline.json` and extract `mode` and `stages`.

3. **Determine next stage from `stages` list:**
   - Find the stage after `$2` in the `stages` list.
   - If no stage follows: `next_stage` = `null`. Otherwise: `next_stage` = the next active stage.

4. **Update `.otsumi/$1/pipeline.json`:**
   - `current_stage` → `null`
   - `last_completed_stage` → `$2`
   - `next_stage` → resolved value or `null`

5. **Resolve pipeline status:**
   - If `next_stage` is `null`: set `status` → `closed`.
   - Else if `mode` is `assisted`: set `status` → `paused`.
   - Else if `mode` is `vibecoding`: leave `status` as `running`.

6. **Log:**
   - Run `/atomic-log $1 $2.completed "$3"`.

7. **Report to the calling agent:**
   - `paused`: report pipeline is paused and the handoff target.
   - `closed`: report pipeline is closed.
   - Otherwise: silent.