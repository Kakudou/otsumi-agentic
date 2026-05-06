---
name: dev-github-issue-safe-fix
description: Handle a GitHub issue by ID through the BDD delivery workflow with mandatory issue verification, test-suite convention scanning, prompt refinement, user-selected pipeline mode, and no commit or push.
---

# dev-github-issue-safe-fix

Fix a GitHub issue by ID through the BDD delivery workflow while preserving the existing codebase, test conventions, and project style.

This skill is intentionally pre-pipeline. It gathers issue context, verifies the current source, scans test conventions, asks the user for the pipeline mode, refines the feature request through the prompt-refinement skill, then starts the pipeline with the correct stages.

## Usage

```text
/dev-github-issue-safe-fix <issue-id>
```

Use when repository context is already known and the requested work is anchored to a GitHub issue ID.

## Required Companion Skills

- `/agent-prompt-master` — mandatory prompt refinement before starting the pipeline.
- `/flow-start-pipeline` — mandatory execution entrypoint after the checklist is complete.
- `/dev-bdd-workflow` — the expected workflow semantics for RED → GREEN delivery.

If the host environment still exposes unprefixed compatibility names, the caller may map these names to local equivalents, but the behavior in this skill remains mandatory.

## Hard Rules

- NEVER commit or push.
- NEVER call `/flow-start-pipeline` without running `/agent-prompt-master` first.
- NEVER skip stage 3, even for trivial, obvious, or one-line fixes.
- NEVER shortcut directly to stages 4 and 5. The minimum stage set is 3, 4, 5.
- NEVER decide stages without scanning the test suite first.
- NEVER use direct implementation mode. The BDD workflow is always the execution path.
- NEVER assume no tests exist. Always scan `tests/` or the project equivalent and confirm.
- NEVER introduce a test framework that the project does not already use.
- NEVER assume the issue text matches current implementation. Always verify behavior against source first.
- NEVER expand scope beyond the issue intent unless tightly required for correctness.
- NEVER proceed past Step 4 without an explicit user answer on pipeline mode.
- NEVER write source or test code before `/flow-start-pipeline` has been called.

## Stop Conditions

- STOP and ask the user at Step 4. Do not proceed until the user chooses `assisted` or `vibecoding`.
- STOP after `/flow-start-pipeline` is called, whether the pipeline completes, pauses, or returns a handoff.
- STOP if the issue cannot be fetched, cannot be reproduced, or is invalid after source verification.
- STOP if the repository or issue context is ambiguous and cannot be resolved safely.
- STOP if the existing test framework or project language cannot be identified.

## Execution Protocol

Complete every step in order. Do not reorder, merge, or skip steps.

### Step 1 — Resolve Issue Context

Fetch the issue title, body, labels, and comments.

Then produce a compact issue context block:

```json
{
  "issue_id": "",
  "title": "",
  "labels": [],
  "behavior_must_change": "one sentence",
  "behavior_must_not_change": "one sentence",
  "candidate_source_files_from_issue": [],
  "verified_source_files_in_scope": [],
  "uncertainties": []
}
```

Required checks:

- Restate in one sentence what behavior must change.
- Restate in one sentence what behavior must not change.
- Identify source files suggested by the issue text.
- Verify that the identified source files exist.
- If the issue references stale paths, renamed modules, or missing files, search nearby source structure before stopping.

### Step 2 — Verify Current Source Behavior

Before choosing stages, inspect the current implementation.

Required checks:

- Open the source files most likely affected by the issue.
- Confirm whether the issue behavior is plausible in the current code.
- Identify the smallest source area that can reasonably explain the issue.
- Record whether the issue is reproducible, likely reproducible, not reproducible, or invalid.

Return:

```json
{
  "source_verification": {
    "status": "reproducible | likely_reproducible | not_reproducible | invalid | unknown",
    "evidence": [],
    "affected_units": [],
    "scope_risk": "low | medium | high",
    "notes": []
  }
}
```

If the issue is invalid or cannot be reproduced/verified enough to design a RED test, stop and report the evidence.

### Step 3 — Scan Test Suite Conventions

Always scan the test suite. Never assume.

Required checks:

- Find `tests/` or the project equivalent.
- List the test structure one level deep.
- Open at least two existing test files closest to the affected code.
- Identify the test style in use:
  - `pytest-bdd` with `.feature` files → BDD with Gherkin syntax.
  - `pytest` with `_given_*`, `_when_*`, `_then_*` helpers → BDD-compatible raw pytest.
  - Plain pytest with no BDD helper structure → plain pytest.
  - `unittest` or another framework → document and match it.
- Record naming patterns, fixture style, factory libraries, decorators, marker usage, and assertion conventions.

Return:

```json
{
  "test_conventions": {
    "test_root": "",
    "structure_one_level_deep": [],
    "files_opened": [],
    "framework": "pytest-bdd | raw-pytest-bdd | plain-pytest | unittest | other",
    "naming_pattern": "",
    "factory_libraries": [],
    "fixture_style": "",
    "decorators_and_markers": [],
    "assertion_conventions": [],
    "must_not_introduce": []
  }
}
```

### Step 4 — Determine Minimum Pipeline Stages

The minimum stage set for every fix is `3,4,5`.

| Condition | Required stages |
| --- | --- |
| Fix only, no new entities, no API changes | `3,4,5` |
| Fix plus new behavior, new endpoint, or new public contract | `1,2,3,4,5` |
| Fix plus architectural or technical decision | `1,2,3,4,5,6` |
| Full feature delivery including docs and closure | `1,2,3,4,5,6,7,8` |

Stage 3 is mandatory because every fix changes observable behavior. A RED test must prove the behavior was wrong before stage 4 makes it right.

Return:

```json
{
  "stage_decision": {
    "stages": [3, 4, 5],
    "reason": "",
    "stage_3_test_style": "pytest-bdd | raw-pytest-bdd | plain-pytest | unittest | other",
    "scope_limits": [],
    "source_files_allowed": [],
    "test_files_expected": []
  }
}
```

### Step 5 — Ask For Pipeline Mode

Ask the user exactly:

```text
Which pipeline mode? assisted (you approve each stage) or vibecoding (fully automated)?
```

Rules:

- Do not bundle this with other questions.
- Do not proceed until the user answers.
- Do not default to `vibecoding`.
- Accept only `assisted` or `vibecoding` unless the host workflow defines additional valid modes.

### Step 6 — Run Prompt Refinement

Run `/agent-prompt-master` before starting the pipeline.

Input must be the issue summary, not the raw issue body. Include:

- One-sentence behavior change.
- One-sentence non-change boundary.
- Verified source scope.
- Detected test convention.
- Required stage set.
- Any uncertainty that the pipeline must preserve.

Target the prompt for an agentic AI workflow.

Return:

```json
{
  "prompt_refinement": {
    "skill_called": "agent-prompt-master",
    "optimized_prompt": "",
    "preserved_constraints": [],
    "notes": []
  }
}
```

### Step 7 — Start The Pipeline

Call `/flow-start-pipeline` with:

```text
--mode <user_answer>
--lang <detected_project_language>
--stages <stage_decision.stages>
<optimized prompt from /agent-prompt-master>
```

Include the detected test convention in the feature description or pipeline context.

After calling `/flow-start-pipeline`, stop. The pipeline owns all source and test edits from that point onward.

## Stage-3 Guidance For Raw Pytest BDD-Compatible Suites

When the existing test suite uses `_given_*`, `_when_*`, and `_then_*` helper functions, even without `.feature` files or `pytest-bdd`, treat that as a BDD-equivalent convention and follow it exactly.

Required conventions:

- One `_given_<scenario>()` function per setup path.
- One `_when_<action>()` function per action under test.
- One or more `_then_<assertion>()` functions per expected outcome.
- Preserve existing marker conventions such as `@pytest.mark.order(1)` when present.
- Use existing factory libraries such as `polyfactory` or `factory_boy` when present.
- Preserve existing assertion conventions such as `# noqa: S101` when present.
- Tests must fail against the current unfixed code before stage 4.

Never introduce `pytest-bdd` or `.feature` files if the project does not already use them.

## Stage Boundaries

- Stage 3 writes tests only.
- Stage 4 writes source only.
- Stage 5 refactors source only while preserving behavior.
- Do not combine test and source edits in one stage.
- Limit edits to files directly related to the issue.
- If correctness requires touching a file outside declared scope, surface the reason before touching it.

## Final Output Before Pipeline Start

Before Step 7, provide this handoff object to the caller or workflow:

```json
{
  "issue_id": "",
  "issue_summary": "",
  "source_verification": {},
  "test_conventions": {},
  "stage_decision": {},
  "pipeline_mode": "assisted | vibecoding",
  "optimized_prompt": "",
  "flow_start_pipeline_command": "",
  "stop_after_pipeline_start": true
}
```
