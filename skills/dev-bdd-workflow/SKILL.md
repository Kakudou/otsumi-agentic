---
name: dev-bdd-workflow
description: "BDD delivery workflow doctrine: feature specification, traps, RED tests, minimal implementation, refactor, decisions, quality score, docs."
---

# Dev BDD Workflow

Provide BDD workflow orchestration doctrine to the host workflow. This is a workflow skill, NOT an agent identity.

## Hard Rules

- NEVER infer language from natural language alone.
- Every pipeline MUST persist `language_id` and `stages`.
- Stage routing MUST use explicit state and adapter skills.
- NEVER implement before RED tests exist when the active stages include tests.
- NEVER refactor before tests are GREEN when test-driven stages are active.
- User-facing questions MUST be handled by the host workflow, not by hidden stage workers.

## Canonical Stages

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

`/flow-start-pipeline`, `/flow-continue`, `/flow-complete-stage`, `/flow-abort`, `/flow-replay`, `/flow-diff-stage`, `/core-atomic-log`, `/core-status`, `/core-backlog`

## Development Skills

`/dev-env-setup`, `/dev-run-tests`, `/dev-quality-check`, `/dev-delivery-review`, `/dev-expert-code-review`, `/dev-quality-score`, `/doc-writer`, `/doc-decision-record`

## Stage Adapter Skills

`/dev-stage-router`, `/dev-bdd-gherkin`, `/dev-python-test-generator`, `/dev-python-implementer`, `/dev-python-refactorer`

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
3. Route failures:
   - linting/formatting/import hygiene/type safety/architecture → refactor or user remediation
   - test quality → test generator or user remediation
   - docs quality → docs/decision record update
4. Re-run evidence collection.
5. Re-score.
6. Close only when all blocking dimensions pass.

## Pre-Pipeline Context Scan

Before Stage-1, scan project structure and conventions: source directories, tests, docs, config files, existing style, existing workflow records. The scan is advisory and is NOT a formal stage artifact.
