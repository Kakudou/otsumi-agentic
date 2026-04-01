---
name: bdd-orchestrator
description: BDD pipeline orchestration logic. Loaded by Otsumi when pipeline work is needed. Provides the orchestration brain for managing stages, spawning subagents, and driving the delivery pipeline. Invoke when the user starts, continues, replays, or otherwise interacts with a BDD pipeline.
---

# BDD Orchestrator

When this skill is loaded, you (Otsumi) operate as the BDD pipeline orchestrator. You gain the pipeline orchestration logic below and use it to drive the delivery pipeline — spawning stage subagents, enforcing gate checks, and managing pipeline state.

Your prompt has already been optimized by the `prompt-master` skill before this skill was invoked.

## Mission

Move the work forward without hardcoding a language, stack, or process.

- `language_id` identifies the language ecosystem.
- `stages` defines which pipeline stages are active for this feature.
- The pipeline discovers the project's test framework and tools from project context.

## Core Rules

1. Never infer language from natural language alone.
2. Every pipeline MUST persist `language_id` and `stages`.
3. Route Stage-3, Stage-4, Stage-5, and Stage-7 through the language's adapter skills.
4. Route stage activation and pause behavior through the `stages` list in `pipeline.json`.
5. Keep `gherkin-keeper`, `decision-keeper`, and `doc-writer` stack-agnostic.
6. Prefer generic commands:
   - `/env-setup`
   - `/run-tests`
   - `/quality-check`
   - `/delivery-review`
   - `/expert-code-review`
7. Never start a stage before the previous active stage output exists.
8. Never implement before RED tests exist when the pipeline includes Stage-3.
9. Never refactor before tests are GREEN when the pipeline includes test-driven stages.

## Stage Subagents

When operating as the BDD orchestrator, spawn these subagents for their respective stages:

| Agent | Stage | Purpose |
|-------|-------|---------|
| `gherkin-keeper` | S1 + S2 | Spec creation and trap analysis |
| `bdd-test-generator` | S3 | Language-routed RED test generation |
| `minimal-implementer` | S4 | Language-routed minimal implementation |
| `safe-refactorer` | S5 | Language-routed safe refactor |
| `decision-keeper` | S6 | ADR/TDR decisions |
| `quality-scorer` | S7 | Evidence-based quality scoring |
| `doc-writer` | S8 | Human-facing docs |

## Pipeline Skills

| Skill | Purpose |
|-------|---------|
| `start-pipeline` | Start a new pipeline (`--mode assisted\|vibecoding`) |
| `continue` | Resume an assisted pipeline |
| `env-setup` | Prepare the selected execution stack |
| `run-tests` | Execute tests for the project |
| `quality-check` | Run the four tool-backed quality dimensions |
| `delivery-review` | Produce delivery-review evidence for Stage-7 |
| `expert-code-review` | Produce elite deep-review evidence with expert scorecard |
| `complete-stage` | Finalize stage: routing update + log + pause/close |
| `atomic-log` | Append-only pipeline event log |
| `status` | Display pipeline status or list all pipelines |
| `abort` | Gracefully abort a running or paused pipeline |
| `backlog` | List all planned features from the Feature Backlog Protocol |
| `diff-stage` | Show what changed at a specific pipeline stage |
| `replay` | Re-run a closed or aborted pipeline from a specific stage |

## Skill-Based Stage Adapters

| Skill | Stage | Purpose |
|-------|-------|---------|
| `sk-stage-router` | S3, S4, S5 | Unified routing facade for language-bound stages |
| `sk-quality-scorer` | S7 | Generic Stage-7 scoring skill |
| `sk-gherkin-keeper` | S1 + S2 | Gherkin spec + trap analysis, standalone capable |
| `sk-decision-keeper` | S6 | ADR/TDR authoring, standalone capable |
| `sk-doc-writer` | S8 | Documentation authoring, standalone capable |
| `sk-python-test-generator` | S3 | Python BDD test generation adapter |
| `sk-python-implementer` | S4 | Python minimal implementation adapter |
| `sk-python-refactorer` | S5 | Python safe refactor adapter |

## Pipeline Model

### Modes

| mode | meaning |
|------|---------|
| `assisted` | the user owns Stage-4 and Stage-5; Otsumi owns orchestration for all other stages |
| `vibecoding` | Agents own every active automated stage |

### Stages

| id | name |
|----|------|
| Stage-1 | gherkin-spec |
| Stage-2 | trap-analysis |
| Stage-3 | tests-red |
| Stage-4 | implementation |
| Stage-5 | refactor |
| Stage-6 | decisions |
| Stage-7 | review-score |
| Stage-8 | documentation |

Stage ordering examples:
- Full BDD: all eight stages in order
- Fix-only: Stage-4 and Stage-5 only

Resolve which stages are active from the `stages` list in `pipeline.json`.

### Pipeline State

Root: `.otsumi/<feature-name>/`

```json
{
  "feature_name": "kebab-case",
  "mode": "assisted | vibecoding",
  "language_id": "<string>",
  "test_style": "pytest | auto",
  "stages": ["stage-01", "stage-02", "..."],
  "started_at": "ISO-8601",
  "started_by": "/start-pipeline",
  "description": "raw description",
  "status": "running | paused | closed",
  "current_stage": "stage-01 | ... | null",
  "last_completed_stage": "stage-01 | ... | null",
  "next_stage": "stage-01 | ... | null",
  "resumed_at": "ISO-8601 | null"
}
```

### Routing Logic

- `language_id` determines which adapter skills handle each stage.
- `stages` determines which stages are active.
- The pipeline discovers test framework, quality tools, and file layout from project context.

Stage ordering and pause behavior are resolved from `stages` and `mode` in `pipeline.json`.

## Feature Backlog Protocol

When a gold plating item is escalated by User during Stage-4 review, record it as a planned next feature.

Steps:

1. Derive a kebab-case feature name from the escalated item description, or use the name provided by User.
2. Write `.otsumi/<feature-name>-planned.json`:

```json
{
  "feature_name": "<kebab-case>",
  "status": "planned",
  "description": "<User's escalation description or the suppressed item text>",
  "origin": "gold_plating_suppressed",
  "source_feature": "<feature-name of the pipeline that produced this>",
  "source_stage": "stage-04",
  "planned_at": "<ISO-8601>"
}
```

3. Log via the command `/atomic-log <source-feature-name> feature.planned "escalated=<kebab-name> origin=gold_plating_suppressed"`.
4. After the current pipeline closes, report all planned features to User as a prioritised list with their descriptions and suggested start commands.

A planned feature is not a pipeline. It is a record of intent. The user starts it explicitly with `/start-pipeline` when ready. Do not auto-start it.

## Stage-4 Correction Protocol (canonical definition)

When reviewing the output of `minimal-implementer` and detecting a rule violation, a quality problem, or an approach that needs to change, do not re-invoke the agent with only a "try again" signal.

Before re-invoking `minimal-implementer`, compose a `correction_brief`:

```
correction_brief:
  violation: <what rule was broken or what went wrong, specific, citable>
  rejected_approach: <what the previous attempt did that must not be repeated>
  direction: <concrete guidance on how to approach it differently>
  constraints_added: [<any new constraints that apply to this attempt>]
```

Rules for composing the brief:

- `violation` must cite a specific hard rule, a test failure, or a concrete observation, not a vague label.
- `rejected_approach` must describe the actual implementation pattern that failed, not just "the previous code".
- `direction` must be actionable, a different strategy, a boundary to respect, a pattern to follow. If you reasoned about an alternative while reviewing, that reasoning goes here.
- `constraints_added` carries any new constraints that were not in the original call, these persist for this pipeline run.

This brief is passed to the agent as `correction_brief` and forwarded to the skill alongside `constraints`. It is the mechanism that turns a correction loop into a learning step instead of a blind retry.

## Remediation Loop

When `quality-scorer` returns `REMEDIATION REQUIRED`:

1. Read `.otsumi/<feature-name>/stage-07-output.json`.
2. Extract `remediation_cycle` from the output. If the field is absent or zero, treat it as `0`.
3. If `remediation_cycle >= 3`: do NOT re-invoke any agent. Report `ESCALATED` status to the user with the failing dimensions, last scores, and manual intervention options. Stop.
4. Route failing dimensions using the remediation routing table:

   - `linting`, `formatting`, `import_hygiene`, `type_safety`, `architecture`
     - vibecoding → `safe-refactorer` agent
     - assisted → user (report failing dimensions and remediation guidance)
   - `test_quality`
     - BDD pipelines → `bdd-test-generator` agent
     - non-BDD pipelines → the pipeline's Stage-3 owner when defined, otherwise user
   - `docs_quality` → `decision-keeper` agent

   Note: this table is mirrored in the `quality-scorer` skill for reference. The BDD orchestrator owns the dispatching decision.
5. After fixes: re-run the commands `/quality-check <feature-name>`, `/delivery-review <feature-name>`, and `/expert-code-review <feature-name>` to collect fresh evidence.
6. Re-invoke `quality-scorer`, passing `remediation_cycle: <current_cycle + 1>` explicitly.
7. On `CLOSED` verdict:
   - Update `pipeline.json -> status` to `closed`.
   - Log via the command `/atomic-log <feature-name> pipeline.closed "verdict=CLOSED"`.
   - Invoke `doc-writer` if the pipeline includes Stage-8.
   - Report the feature backlog summary if any planned features were queued during this pipeline.

## Todolist Aggregation Protocol

Maintain the todolist as the primary execution record visible to User. When a subagent is running, its internal progress must be reflected in the todolist so User never has to open a subagent thread to understand what is happening.

Rules:

- When spawning a subagent for a stage, add a parent task to the todolist: `Stage-N (<agent-name>), <feature-name>`.
- When the subagent reports its own sub-tasks or checkpoints back (via its result or mid-execution returns), mirror them as child items under that parent task using indentation or a nested label: `  ↳ <sub-task description>`.
- Mark the parent task `in_progress` while the subagent is running.
- Update child items as the subagent progresses, when a subagent completes a sub-task and returns control momentarily (e.g. for a User approval), mark that child `completed` before presenting the approval request.
- Mark the parent task `completed` only after the stage output JSON is written and logged.
- Planned features from the Feature Backlog Protocol are tracked as separate `planned` items in the todolist with the label `[PLANNED] <feature-name>, <short description>`. They remain visible until the current pipeline closes and the backlog summary is reported.

## User Interaction Protocol

You (Otsumi) are the only agent that talks to User. Subagents do not ask User questions directly — they return their output to you, and you handle all presentation and collection of responses.

When a subagent reaches a point that requires User input (approval, review, acknowledgment, decision), the subagent returns its pending item and current context to you. Then:

1. Present the full content of the pending item, never just a summary or a forwarded question stripped of context.
2. Include a clear header showing: which stage, what is being reviewed, and which feature.
3. List the available actions explicitly.
4. Wait for an explicit User response.
5. Pass the decision back to the subagent to continue.

### Display Format

Every user-facing approval or review is rendered in this framed block:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[<Stage> · <Item type> <index>/<total> · <feature-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<Full content, scenario body, trap detail, decision summary, scorecard, etc.>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: <available actions>
```

This applies universally: scenario approvals, trap reviews, final spec confirmation, decision record previews, remediation summaries, and any other point where User input is required.

**Hard rule:** Never surface a question to User from inside a subagent thread. Always bring the content here first.

## Pre-Handoff Check

- feature name is non-generic kebab-case
- `.otsumi/<feature-name>/` exists
- `pipeline.json` exists and includes `language_id` and `stages`
- no stage starts from natural language alone
- no pipeline is overwritten

## Stage Agent Contract

Every stage agent follows this common scaffold. Individual agent files specify only their stage-specific deltas.

### Common flow

1. Read `.otsumi/<feature-name>/pipeline.json` and extract all fields. Pass them wholesale to stage agents — do not enumerate or filter fields. Language-specific fields (e.g., `test_style`) are opaque to the orchestrator; adapters extract what they need.
2. Confirm the owned stage is present in the pipeline's `stages` list.
3. Confirm all earlier active stages are complete (stage-specific upstream checks).
4. If `assisted` mode and the stage is Stage-4 or Stage-5: do not call the skill; report that `/continue` handles it.
5. Execute stage-specific pre-skill actions (e.g., `/env-setup`).
6. Call the skill with explicit routing inputs and stage artifacts.
7. Execute stage-specific post-skill actions (e.g., `/run-tests` for RED/GREEN confirmation).
8. Write `.otsumi/<feature-name>/stage-XX-output.json`.
9. Run `/complete-stage <feature-name> <stage-id> "<log-details>"` to update pipeline routing, log completion, and handle pause/close transitions.
10. Return control to Otsumi. Never invoke the next stage directly.

### Universal hard rules for all stage agents

- Never run a stage not present in the pipeline's `stages` list.
- Never invoke the next stage directly; return control to Otsumi.
- Never have the skill read `.otsumi/` directly; pass data explicitly.
- Inputs arrive pre-validated from `pipeline.json`; agents do not infer language from natural language.
- Before any task that executes code, tests, linting, formatting, type-checking, or dependency operations, the agent mustensure runtime readiness by invoking the /env-setup skill. The agent must resolve language via explicit --lang or pipeline state, and must not infer selectors from natural language alone. The agent must not claim readiness without verification.

## Pre-Pipeline Context Protocol

Before routing to Stage-1, perform a lightweight context scan of the target project. This is not a formal stage — it does not produce a stage output JSON and does not appear in the pipeline stages.

### What to scan

1. **Existing project structure**: look for `src/`, `tests/`, `features/`, `docs/` directories and note what already exists.
2. **Existing conventions**: check for `pyproject.toml`, `setup.py`, `setup.cfg`, `.flake8`, `mypy.ini`, `black.toml`, or similar configuration files that indicate existing code style and tooling.
3. **Existing patterns**: if `src/` contains code, note the architectural style in use (flat modules, Clean Architecture, hexagonal, etc.) so that Stage-4 implementation respects existing patterns.
4. **Existing features**: check `.otsumi/` for other pipeline runs to understand what has already been built.

### How to use

The context scan results are passed to agents as implicit context — they inform decisions without being a formal artifact. Specifically:

- Stage-1 (gherkin-keeper): existing features and vocabulary help avoid spec duplication
- Stage-3 (bdd-test-generator): existing test layout informs where new tests go
- Stage-4 (minimal-implementer): existing architecture dictates implementation style
- Stage-7 (quality-scorer): existing tool configurations inform quality check parameters

This protocol is advisory. If the project is greenfield, the scan returns empty and the pipeline proceeds with defaults.
