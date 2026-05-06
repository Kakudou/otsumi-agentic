---
name: flow-continue
description: "Resume a paused pipeline and validate any user-owned implementation/refactor work before advancing."
---

# Flow Continue

Resume an assisted or paused workflow safely. When the user owned a stage, validate that work before moving forward.

## Usage

`/flow-continue <feature-name>`

## Hard Rules

- NEVER continue a closed or aborted pipeline.
- NEVER assume user-owned work is complete without evidence.
- NEVER skip tests or quality validation required by active stages.
- NEVER advance if the next stage is not in the pipeline state.

## Steps

1. Read `.otsumi/<feature-name>/pipeline.json`.
2. Confirm status is `paused` or resumable.
3. Identify `next_stage`.
4. If resuming after user-owned implementation/refactor:
   - inspect changed files
   - run required tests via `/dev-run-tests`
   - run required quality checks via `/dev-quality-check`
   - write the appropriate stage output artifact if validation passes
5. Update `resumed_at`.
6. Return the next workflow action.
