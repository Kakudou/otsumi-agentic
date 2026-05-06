---
name: sk-quality-scorer
description: Scores a feature across the pipeline quality dimensions using language-aware evidence when Stage-7 is in the pipeline's stages.
---

# quality-scorer

One job: produce a real verdict from current tool output and review evidence.

## Hard Rules

- NEVER bake one language's tools into generic logic.
- NEVER score without fresh evidence.
- NEVER score without the expert review and its scorecard.
- NEVER treat final Stage-8 docs as part of the Stage-7 closure gate.
- NEVER invent a passing verdict.

## Inputs

- `feature_name` - required
- `mode` - required
- `language_id` - required
- `quality_report` - required
- `code_review` - required
- `expert_review` - required
- `stage_results` - optional map
- `remediation_cycle` - optional

## Scored Dimensions

- `linting`
- `formatting`
- `import_hygiene`
- `type_safety`
- `architecture`
- `test_quality`
- `docs_quality`

## Scoring Rubric

Every dimension is scored 1–5. The rubric is consistent across all dimensions:

| Score | Meaning | Criteria |
|-------|---------|----------|
| **5** | Exemplary | No issues found. Meets or exceeds all standards. Production-ready without caveats. |
| **4** | Good | Minor issues only. Nothing that blocks delivery or creates maintenance risk. Acceptable for production. |
| **3** | Needs Work | Moderate issues present. At least one finding that creates maintenance risk or violates a convention. **Blocks closure.** |
| **2** | Poor | Significant issues. Multiple findings that create risk, indicate missing discipline, or violate hard rules. Requires focused remediation. |
| **1** | Critical | Fundamental problems. The dimension is functionally unmet. Major rework needed. |

The minimum threshold for closure is **4/5 on every dimension**. A single dimension at 3/5 blocks closure regardless of overall average.

## Meaning of `docs_quality`

In Stage-7, `docs_quality` means:

- decision-record quality
- user/operator-facing clarity in pre-close artifacts
- readiness for Stage-8 documentation

It does **not** mean final Stage-8 docs. Stage-8 happens after a CLOSED verdict.

## Evidence Rules

1. Tool-backed dimensions MUST come only from `/quality-check`.
2. Review-backed dimensions MUST come from `/delivery-review`, `expert_review`, plus the files.
3. Every note MUST cite a tool output line, file, or review observation — no unsupported claims.
4. MUST confirm Stage-7 is in the pipeline's stages before scoring; STOP if it is not.
5. MUST use `expert_review` as additional high-signal evidence — NEVER as a replacement for tool-backed checks.

## Language Tool Defaults

- `python`
  - linting: flake8
  - formatting: black
  - import_hygiene: isort
  - type_safety: mypy

## Verdict Logic

- `CLOSED` when every dimension is >= 4
- `REMEDIATION REQUIRED` when any dimension is < 4 and `remediation_cycle < 3`
- `ESCALATED` when any dimension is < 4 and `remediation_cycle >= 3`

## Remediation Routing (canonical source - bdd-orchestrator skill references this table)

- `linting`, `formatting`, `import_hygiene`, `type_safety`, `architecture`
  - vibecoding → `safe-refactorer` agent
  - assisted → user
- `test_quality`
  - pipelines including Stage-1 and Stage-3 → `bdd-test-generator` agent
  - pipelines without Stage-3 → the pipeline's Stage-3 owner when defined, otherwise user
- `docs_quality` → `decision-keeper` agent

Note: routing targets are **agent names**, not skill names. The Otsumi dispatches the agent, and the agent resolves the correct skill via the stage-router or direct skill loading.

## Result Shape

Return `stage-07-result` with:

- `pipeline_mode`
- `language_id`
- `scorecard`
- `quality_tools`
- `review_assessments`
- `expert_review_summary`
- `expert_scorecard`
- `overall`
- `threshold_met`
- `remediation_cycle`
- `remediation`
- `verdict`
