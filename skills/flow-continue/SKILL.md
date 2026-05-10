---
name: flow-continue
description: "Resume a paused pipeline and validate any user-owned implementation/refactor work before advancing."
---

# Flow Continue

Resume a paused workflow safely. If the user owned implementation/refactor work, validate it before advancing.

## Usage

`/flow-continue {feature-name}`

## Mission

Execute deterministic resume logic against `.otsumi/{feature-name}/pipeline.json`, enforce stage validation requirements, and return the next workflow action.

## Hard Rules

- NEVER continue a closed or aborted pipeline.
- NEVER assume user-owned work is complete without evidence.
- NEVER skip tests or quality validation required by active stages.
- NEVER advance if the next stage is not in the pipeline state.

## Core Procedure

1. Read `.otsumi/{feature-name}/pipeline.json`.
2. Confirm status is `paused`.
3. Identify `next_stage`.
4. If resuming after user-owned implementation/refactor:
   - inspect changed files
   - run required tests via `/dev-run-tests`
   - run required quality checks via `/dev-quality-check`
   - write the appropriate stage output artifact if validation passes
5. Update `resumed_at`.
6. Return the next workflow action.

## Restored Resume Contract (Assisted Mode)

This section is the authoritative assisted-mode behavior for `flow-continue`.
Fuhyō is the executor for validation/materialization steps; Ōshō is the user interface for explicit user confirmations.

Stage output schemas are canonically defined by `dev-bdd-workflow`; this skill must materialize outputs using those schema fields.

### Step 3 Confirmation Checks (Required)

When validating resumability from `.otsumi/{feature-name}/pipeline.json`, confirm all conditions below:
These checks enforce S4 transition legality for `pipeline.continued` and must pass before applying T5/T6.

- `mode` is `assisted`
- `status` is `paused`
  - if `status` is `closed`, halt with: `Pipeline "{feature-name}" is already closed. No action taken.`
  - if `status` is `running`, halt with: `Pipeline "{feature-name}" is already running at "{current_stage}". Use /flow-continue only when the pipeline is paused.`
- `language_id` exists
- `stages` list exists and is non-empty
- `next_stage` exists (or pipeline is eligible for closure when stages are exhausted)

## State Machine Reference

This skill implements S4 transition **T5**: `paused → pipeline.continued → running`.

- Guard: `mode=assisted` AND `next_stage` exists AND required validation passes
- Also covers **T6**: `paused → pipeline.continued → closed` (when no stages remain)
- A `flow-continue` against a `running` pipeline MUST be refused (running pipelines don't need continuation)

### Contiguous Block Materialization Rule

If `next_stage` starts a contiguous block of Stage-04/Stage-05, materialize every stage in that block, in order, before advancing.

> Domain-agnostic boundary: stage-specific schemas and resume logic are owned by the stage adapter skill referenced by the pipeline stage definition (for example, `dev-stage-router`), not by `flow-continue`. `flow-continue` remains a lifecycle skill that delegates stage-specific resume behavior to that adapter.

### Stage-04 Assisted Mode Procedure (6 Steps)

If `stage-04` is active and next in ordered `stages`:

1. Resolve implementation files from project structure (feature-related source files).
2. If Stage-03 exists upstream in `stages`, run `/dev-run-tests {feature-name}` and require `GREEN`.
3. If no upstream test stage exists, require explicit user confirmation that implementation is complete and record absence of automated GREEN proof.
4. If no implementation files are found, halt and report no files found.
5. Write full `.otsumi/{feature-name}/stage-04-output.json` using the schema below.
6. Append completion event via `/core-atomic-log`.

#### stage-04-output.json schema

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-04",
  "language_id": "<language-id>",
  "completed_by": "user",
  "completion_source": "/flow-continue",
  "implementation_files": ["<paths>"],
  "green_state_confirmed": true,
  "verification_method": "/dev-run-tests <feature-name> | user confirmation when no upstream test stage exists",
  "verified_at": "<ISO-8601>"
}
```

If no upstream test stage exists, `green_state_confirmed` must be `null` and `verification_method` must record user confirmation path.

### Stage-05 Assisted Mode Procedure (5 Steps)

If `stage-05` is active and next in ordered `stages`:

1. Require `.otsumi/{feature-name}/stage-04-output.json`.
2. Run tests with `/dev-run-tests {feature-name}` when tests exist, or require explicit user confirmation when no tests exist.
3. Write full `.otsumi/{feature-name}/stage-05-output.json` using the schema below.
4. Append completion event via `/core-atomic-log`.
5. Update `last_completed_stage` and `next_stage` based on ordered active `stages`.

#### stage-05-output.json schema

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-05",
  "language_id": "<language-id>",
  "completed_by": "user",
  "completion_source": "/flow-continue",
  "implementation_files": ["<paths>"],
  "tests_still_pass": true,
  "verification_method": "/dev-run-tests <feature-name> | user confirmation when no tests exist",
  "verified_at": "<ISO-8601>"
}
```

If no tests exist, `tests_still_pass` must be `null` and `verification_method` must record user confirmation path.

### Final Resume Status Transition Logic

After any required Stage-04/Stage-05 materialization:

1. If no active stages remain:
   - set `status` to `closed`
   - set `current_stage` to `null`
   - set `next_stage` to `null`
2. Else:
   - set `status` to `running`
   - set `current_stage` to resolved next active stage
   - set `resumed_at` to `<ISO-8601>`
3. Log with `/core-atomic-log` using command format:
   - `/core-atomic-log <feature-name> pipeline.continued "language=<language_id> next_stage=<stage-or-null>"`

## Additional Hard Rules (Restored)

- Never resume by repo inspection alone; use the stages list ordering and explicit validation.
- Never jump over an active stage silently.
- Never advance past Stage-04 or Stage-05 without materializing its stage output JSON.
