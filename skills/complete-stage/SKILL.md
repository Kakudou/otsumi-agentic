---
name: complete-stage
description: Finalize a stage by updating pipeline routing, logging completion, and handling pause/close transitions.
---

You are receiving a `/complete-stage` command.

## Usage

`/complete-stage <feature-name> <stage-id> "<log-details>"`

## Purpose

Handle the common completion sequence after a stage agent writes its output JSON. This command updates pipeline routing, logs the event, and resolves whether the pipeline pauses, continues, or closes.

## Steps

1. **Validate inputs:**
   - `$1` (feature-name) must not be empty.
   - `$2` (stage-id) must be a valid stage identifier (`stage-01` through `stage-08`).
   - `$3` (log-details) must not be empty. The caller must provide a meaningful summary built from stage output data.

2. **Read pipeline state:**
   - Read `.otsumi/$1/pipeline.json` and extract `mode` and `stages`.
   - Read the `stages` list from `pipeline.json`. This is the ordered list of active stage IDs for this pipeline.

3. **Determine next stage:**
   - Find the stage after `$2` in the `stages` list.
   - If no stage follows: `next_stage` = `null`.
   - Otherwise: `next_stage` = the next active stage in the list.

4. **Update `.otsumi/$1/pipeline.json`:**
   - `current_stage` -> `null`
   - `last_completed_stage` -> `$2`
   - `next_stage` -> resolved next stage or `null`

5. **Resolve pipeline status:**
   - If `next_stage` is `null`: set `status` -> `closed`.
   - If `mode` is `assisted`: set `status` -> `paused`.
   - If `mode` is `vibecoding`: leave `status` as `running`.

6. **Log:**
   - Run `/atomic-log $1 $2.completed "$3"`.

7. **Report back to the calling agent:**
   - If `status` changed to `paused`: report that the pipeline is paused and the handoff target.
   - If `status` changed to `closed`: report that the pipeline is closed.
   - Otherwise: silent.

## Hard Rules

- Stage ordering must come from the `stages` list in `pipeline.json`, not from stage-id arithmetic.
- Never set `status` to `closed` when active stages remain.
- Never modify the stage output JSON. This command only touches `pipeline.json` and the atomic log.