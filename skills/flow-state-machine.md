# XC-002 — Canonical Pipeline State Machine Specification

## Purpose

Define the single canonical lifecycle state machine for flow pipeline instances. This specification is referenced by:

- `flow-start-pipeline`
- `flow-complete-stage`
- `flow-continue`
- `flow-abort`
- `flow-replay` (replay semantics only; not a state transition)

## State Set

- **(none)**: pseudo-state before a pipeline instance exists
- **running**: active, routable pipeline state
- **paused**: assisted-mode hold state awaiting user-owned work or approval gate completion
- **closed**: terminal completed state
- **aborted**: terminal stopped state

## Transition Table

| ID | From | Event | Guard / Preconditions | To | Implemented by |
|---|---|---|---|---|---|
| **T1** | (none) | `pipeline.started` | valid initialization request; new pipeline instance created | `running` | `flow-start-pipeline` |
| **T2** | `running` | `stage.completed` | active stages remain; T4 not applicable; T3 guard not satisfied | `running` | `flow-complete-stage` |
| **T3** | `running` | `stage.completed` | assisted mode + pause condition (approval gate or user-owned stage) | `paused` | `flow-complete-stage` |
| **T4** | `running` | `stage.completed` | no active stages remain | `closed` | `flow-complete-stage` |
| **T5** | `paused` | `pipeline.continued` | `mode=assisted` AND `next_stage` exists AND required validation passes | `running` | `flow-continue` |
| **T6** | `paused` | `pipeline.continued` | resumed and no active stages remain | `closed` | `flow-continue` |
| **T7** | `running` | `pipeline.aborted` | non-empty `abort_reason` | `aborted` | `flow-abort` |
| **T8** | `paused` | `pipeline.aborted` | non-empty `abort_reason` | `aborted` | `flow-abort` |

## Deterministic Evaluation Order (stage completion)

For `flow-complete-stage`, evaluate in this order:

1. **T4 first**: if no active stages remain, close.
2. **T3 second**: if assisted-mode pause guard is satisfied, pause.
3. **T2 fallback**: otherwise remain running.

## Guard and Invariant Rules

### OQ-2 (Required)

- **T3 MUST NOT fire when `mode=vibecoding`.**
- In vibecoding mode, only **T2** (`running→running`) or **T4** (`running→closed`) are valid from `running` on `stage.completed`.

### OQ-3 (Required)

- Abort has **no `--force` override path**.
- No terminal-state bypass is allowed.
- Aborting a `closed` pipeline MUST halt with error.
- Aborting an `aborted` pipeline MUST halt with error.

### Continue invariants

- `flow-continue` is valid for paused/resumable state handling only.
- Invoking continue on `running` MUST be refused.
- Continuing `closed` or `aborted` pipelines MUST be refused.

### Terminal-state invariants

- `closed` and `aborted` are terminal for that pipeline instance.
- No transition returns a terminal instance to `running`.

## Replay Semantics (Explicitly Outside State Transitions)

Replay is **not** a transition from `closed` or `aborted`.

- Replay creates a **new pipeline instance** with `status: running`.
- The original terminal `pipeline.json` remains unchanged permanently.
- The new instance records `replayed_from` metadata:
  - `original_pipeline_id`
  - `replayed_from_stage`
  - `original_status` (`closed` or `aborted`)
  - `replay_timestamp` (ISO-8601)

## Compliance Checklist

An implementation is compliant with XC-002 only if:

1. All states above are represented, including pseudo-state `(none)`.
2. All transitions **T1–T8** are implemented with the declared guards.
3. OQ-2 and OQ-3 are enforced.
4. Replay is implemented as new-instance creation, not terminal-state mutation.
