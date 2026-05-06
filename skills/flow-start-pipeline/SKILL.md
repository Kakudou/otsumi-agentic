---
name: flow-start-pipeline
description: "Start one or more explicit workflow pipelines with mode, language, test style, stages, state root, and routing selectors."
---

# Flow Start Pipeline

Initialize one or more pipelines with explicit routing selectors. Create state for feature/workflow delivery without guessing language, overwriting work, or merging unrelated features.

## Usage

`/flow-start-pipeline --mode <assisted|vibecoding> --lang <language_id> [--stages 1,2,3] [--pytest] [--requirements-contract <json|path>] <description>`

Shorthand aliases that may map to this skill:

- `/start-assisted ...`
- `/start-vibecoding ...`

The `--requirements-contract` flag carries Kinshō's PO contract (acceptance criteria, quality thresholds, definition-of-done, non-functional requirements) when the orchestrator runs Kinshō upstream of the pipeline. The contract is persisted in the state root and read by stage adapters (especially `dev-bdd-gherkin` at stage-01).

## Hard Rules

- NEVER infer selectors from natural language alone.
- NEVER allow an unsupported `language_id` to proceed.
- NEVER overwrite an existing pipeline.
- NEVER combine multiple distinct features into a single pipeline.
- NEVER proceed if required flags or description are absent, unless interactive discovery is explicitly requested.
- If multiple features target shared files, they MUST be sequenced rather than run in parallel.

## Steps

1. Validate `--mode`, `--lang`, and description.
2. If absent, inspect project context only to suggest values — do NOT silently choose.
3. Identify distinct features in the request. Ask for clarification if the split affects safety.
4. For each feature:
   - derive a non-generic kebab-case feature name
   - create `.otsumi/<feature-name>/`
   - refuse if `.otsumi/<feature-name>/pipeline.json` already exists
   - validate that required adapter skills exist for requested active stages
5. Resolve active stages:
   - full delivery: `stage-01` through `stage-08`
   - quick fix: `stage-04`, `stage-05`
   - explicit `--stages`: exact requested list
6. Resolve `test_style`:
   - `--pytest` → `pytest`
   - omitted → `auto`
7. Write `.otsumi/<feature-name>/pipeline.json`:

```json
{
  "feature_name": "",
  "mode": "assisted|vibecoding",
  "language_id": "",
  "test_style": "pytest|auto",
  "stages": [],
  "started_at": "",
  "started_by": "/flow-start-pipeline",
  "description": "",
  "status": "running|paused",
  "current_stage": null,
  "last_completed_stage": null,
  "next_stage": "",
  "resumed_at": null,
  "requirements_contract_path": null
}
```

8. If `--requirements-contract` is provided:
   - persist the contract content to `.otsumi/<feature-name>/kinsho-contract.json`
   - set `requirements_contract_path` in `pipeline.json` to that path
   - stage adapters (especially `dev-bdd-gherkin`) MUST read this file when present
9. Log `pipeline.started`.
10. Return the next action for the host workflow.

## Optimization Note

Assisted mode SHOULD pause before user-owned work or explicit approval gates, NOT necessarily immediately after initialization. This preserves assisted delivery without contradicting spec/test automation.
