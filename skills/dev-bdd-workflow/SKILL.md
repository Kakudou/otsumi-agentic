---
name: dev-bdd-workflow
description: "BDD delivery workflow doctrine: feature specification, traps, RED tests, minimal implementation, refactor, decisions, quality score, docs."
---

# Dev BDD Workflow

Provide deterministic BDD workflow doctrine to the host workflow.
This is a workflow skill, NOT an agent identity.

## Purpose Lock

- Define the canonical BDD delivery stages and routing rules.
- Enforce RED → GREEN → refactor discipline.
- Keep orchestration state explicit, machine-readable, and auditable.

## Hard Rules

- NEVER infer language from natural language alone.
- Every pipeline MUST persist `language_id` and `stages`.
- Stage routing MUST use explicit state and adapter skills.
- NEVER implement before RED tests exist when the active stages include tests.
- NEVER refactor before tests are GREEN when test-driven stages are active.
- User-facing questions MUST be handled by the host workflow, not by hidden stage workers.

## Canonical Stages

These stage IDs are canonical and ordered.

| ID | Name | Purpose |
|---|---|---|
| `stage-01` | gherkin-spec | behavior contract |
| `stage-02` | trap-analysis | edge cases and adversarial scenarios |
| `stage-03` | tests-red | failing tests |
| `stage-04` | implementation | minimal GREEN implementation |
| `stage-05` | refactor | behavior-preserving cleanup |
| `stage-06` | decisions | ADR/TDR records |
| `stage-07` | review-score | evidence-backed quality verdict |
| `stage-08` | documentation | user-facing docs |

## Required Workflow Skills

- `/flow-start-pipeline`
- `/flow-continue`
- `/flow-complete-stage`
- `/flow-abort`
- `/flow-replay`
- `/flow-diff-stage`
- `/core-atomic-log`
- `/core-status`
- `/core-backlog`

## Development Skills

- `/dev-env-setup`
- `/dev-run-tests`
- `/dev-quality-check`
- `/dev-delivery-review`
- `/dev-expert-code-review`
- `/dev-quality-score`
- `/doc-writer`
- `/doc-decision-record`

## Stage Adapter Skills

- `/dev-stage-router`
- `/dev-bdd-gherkin`
- `/dev-python-test-generator`
- `/dev-python-implementer`
- `/dev-python-refactorer`

## Correction Brief Protocol

When implementation or refactor output violates a rule, compose a correction brief BEFORE retry:

```yaml
correction_brief:
  violation: ""
  rejected_approach: ""
  direction: ""
  constraints_added: []
```

The brief MUST cite a concrete violation, describe the rejected pattern, give a different strategy, and preserve newly discovered constraints.

## Remediation Loop

When quality scoring requires remediation:

1. Read the latest quality output.
2. Stop after three remediation cycles and escalate manually.
3. Route failures by dimension:
   - linting/formatting/import hygiene/type safety/architecture → refactor or user remediation
   - test quality → test generator or user remediation
   - docs quality → docs/decision record update
4. Re-run evidence collection.
5. Re-score.
6. Close only when all blocking dimensions pass.

## Pre-Pipeline Context Scan

Before Stage-1, scan project structure and conventions: source directories, tests, docs, config files, existing style, existing workflow records. The scan is advisory and is NOT a formal stage artifact.

## Pipeline State Canonical Schema

Root: `.otsumi/<feature-name>/pipeline.json`

```json
{
  "feature_name": "kebab-case",
  "mode": "assisted | vibecoding",
  "language_id": "<string>",
  "test_style": "pytest | auto",
  "stages": ["stage-01", "stage-02", "stage-03", "stage-04", "stage-05", "stage-06", "stage-07", "stage-08"],
  "started_at": "ISO-8601",
  "started_by": "flow-start-pipeline",
  "description": "raw description",
  "status": "running | paused | closed",
  "current_stage": "stage-01 | stage-02 | stage-03 | stage-04 | stage-05 | stage-06 | stage-07 | stage-08 | null",
  "last_completed_stage": "stage-01 | stage-02 | stage-03 | stage-04 | stage-05 | stage-06 | stage-07 | stage-08 | null",
  "next_stage": "stage-01 | stage-02 | stage-03 | stage-04 | stage-05 | stage-06 | stage-07 | stage-08 | null",
  "resumed_at": "ISO-8601 | null"
}
```

## Feature Backlog Protocol

When a Gold Plating item is escalated, Kakugyō records a planned feature:

1. Derive kebab-case feature name from the escalation text, or use the explicit user-provided name.
2. Write `.otsumi/<feature-name>-planned.json`:

```json
{
  "feature_name": "<kebab-case>",
  "status": "planned",
  "description": "<escalation summary>",
  "origin": "gold_plating_suppressed",
  "source_feature": "<source feature>",
  "source_stage": "stage-04",
  "planned_at": "<ISO-8601>"
}
```

3. Log via `core-atomic-log` with `feature.planned` event metadata.
4. After the current pipeline closes, Ōshō reports all planned features to User.

A planned feature is not a pipeline. It is a record of intent.

## Todolist Aggregation Protocol

Keep pipeline progress visible as a single canonical execution record:

1. Add one parent task per stage run.
2. Add child items for discovered sub-tasks or stage checkpoints.
3. Keep parent task `in_progress` while the stage is running.
4. Track planned features as `[PLANNED] <feature-name> - <short description>`.
5. Mark completed items immediately when done.
6. Add newly discovered sub-tasks immediately when discovered.

## User Interaction Protocol

Ōshō is the ONLY agent that talks to User. Fuhyō workers return output to Ōshō; Ōshō presents and collects decisions.

Use this framed display format for scenario approvals, trap reviews, scorecard summaries, and remediation summaries:

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Stage · Item type N/total · feature-name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<full content under review>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: <explicit allowed actions>
```

## Stage Agent Contract

Each stage Fuhyō MUST follow this scaffold:

1. Read `.otsumi/<feature-name>/pipeline.json`.
2. Confirm owned stage exists in `stages`.
3. Extract `mode` and `language_id`.
4. Verify required upstream stage output exists.
5. Invoke `dev-env-setup`.
6. Invoke the stage skill via `dev-stage-router` or direct stage skill.
7. Collect stage result.
8. Write `.otsumi/<feature-name>/stage-XX-output.json`.
9. Invoke `core-atomic-log`.
10. Invoke `flow-complete-stage`.

Universal hard rules:

- Never run a stage not present in `stages`.
- Never invoke the next stage directly.
- Never have stage skills read `.otsumi` directly; pass explicit inputs.
- Must invoke `dev-env-setup` before any code execution/test/lint/type activity.
- If any pre-condition fails, halt and report failure immediately.

## Correction Brief Protocol (Field Rules)

When implementation or refactor output violates a rule, compose a correction brief BEFORE retry:

```yaml
correction_brief:
  violation: ""
  rejected_approach: ""
  direction: ""
  constraints_added: []
```

Field rules:

- `violation` must cite a specific rule, test failure, or concrete observation (not a vague label).
- `rejected_approach` must describe the actual implementation pattern used (not "the previous code").
- `direction` must be actionable and specific about the next approach.
- `constraints_added` persists for the entire pipeline run.

## Remediation Loop Routing Table

When quality scoring returns remediation required, dispatch by failing dimension:

| Dimension | Route |
|---|---|
| `linting`, `formatting`, `import_hygiene`, `type_safety`, `architecture` | Fuhyō via `dev-python-refactorer` |
| `test_quality` | Fuhyō via `dev-python-test-generator` and re-run Stage-3 |
| `docs_quality` | Fuhyō via `doc-decision-record` |

## Pre-Handoff Check

Before Stage-1 routing:

- feature name is non-generic kebab-case
- `.otsumi/<feature-name>/` exists
- `pipeline.json` exists and includes `language_id` and `stages`
- no stage starts from natural language alone
- no pipeline is overwritten

## Gold Plating Review Protocol

When a Stage-4 Fuhyō run (executing `dev-python-implementer`) returns non-empty `gold_plating_suppressed`, Ōshō must present each item to User before writing stage output.

Per item, User responds with exactly one action:

- `suppress` — accept suppression
- `escalate` — queue as planned feature
- `escalate as "<name>"` — queue with explicit custom feature name

Only after all items are reviewed may stage output be written. Every escalated item is routed through the Feature Backlog Protocol.

## Stage Output JSON Schemas (Reference)

This section is canonical reference for stage artifacts. Other skills point here.

### stage-01-output.json

```json
{
  "feature_file": "features/<feature>.feature",
  "scenarios": [
    {
      "name": "<scenario name>",
      "approved": true
    }
  ],
  "review_status": "approved | needs_changes"
}
```

### stage-02-output.json

```json
{
  "traps": [
    {
      "category": "<string>",
      "severity": "low | medium | high | critical",
      "description": "<string>",
      "scenario_name": "<scenario>",
      "approved": true
    }
  ],
  "review_status": "approved | needs_changes"
}
```

### stage-03-output.json

```json
{
  "tests_collected": 0,
  "failed": 0,
  "errored": 0,
  "unexpected_passes": 0,
  "test_file": "tests/<path>.py",
  "steps_file": "tests/steps/<path>.py",
  "support_files": ["tests/conftest.py"]
}
```

### stage-04-output.json

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-04",
  "language_id": "<language-id>",
  "implementation_files": ["src/<path>.py"],
  "test_infrastructure_changes": [],
  "step_file_changes": [],
  "gold_plating_suppressed": [
    {
      "item": "<string>",
      "reason": "<string>",
      "escalated_as_feature": false,
      "feature_name": null
    }
  ],
  "green_state_confirmed": true
}
```

### stage-05-output.json

```json
{
  "feature_name": "<feature-name>",
  "stage": "stage-05",
  "language_id": "<language-id>",
  "completion_source": "dev-python-refactorer",
  "implementation_files": ["src/<path>.py"],
  "tests_still_pass": true,
  "verification_method": "dev-run-tests",
  "verified_at": "<ISO-8601>",
  "aborted_refactors": []
}
```

### stage-06-output.json

```json
{
  "decision_required": false,
  "records_written": [],
  "records_updated": [],
  "no_record_reasoning": "<string>",
  "language_id": "<language-id>"
}
```

### stage-07-output.json

```json
{
  "pipeline_mode": "assisted | vibecoding",
  "language_id": "<language-id>",
  "scorecard": {},
  "quality_tools": {},
  "review_assessments": {},
  "expert_review_summary": "<string>",
  "expert_scorecard": {},
  "overall": "<summary>",
  "threshold_met": true,
  "remediation_cycle": 0,
  "remediation": [],
  "verdict": "CLOSED | REMEDIATION REQUIRED | ESCALATED"
}
```

### stage-08-output.json

```json
{
  "docs_written": [
    {
      "type": "readme | guide | changelog | adr | tdr",
      "path": "docs/<path>.md"
    }
  ]
}
```
