---
name: dev-quality-score
description: "Score final output using evidence and thresholds, producing CLOSED, REMEDIATION REQUIRED, or ESCALATED verdicts."
---

# Dev Quality Score

Aggregate quality evidence into an evidence-backed final verdict — no vibes.

## Usage

`/dev-quality-score {feature-name}`

## Hard Rules

- NEVER score without evidence.
- NEVER pass if any blocking dimension is below threshold.
- NEVER average away a critical failure.
- MUST list failing dimensions and remediation guidance.
- MUST stop after the configured maximum remediation cycles.

## Default Dimensions

- linting
- formatting
- import_hygiene
- type_safety
- architecture
- test_quality
- docs_quality

Default threshold: every dimension MUST be at least 4/5 unless the workflow defines otherwise.

## Scoring Rubric (1-5 Named Scale)

Every dimension is scored 1–5 using the same named rubric:

| Score | Name | Criteria |
|---|---|---|
| **5** | Exemplary | No issues found. Meets or exceeds standards. Production-ready without caveats. |
| **4** | Good | Minor issues only. Acceptable for production. |
| **3** | Needs Work | Moderate issues present. Blocks closure. |
| **2** | Poor | Significant issues. Requires remediation. |
| **1** | Critical | Fundamental problems. Major rework needed. |

The minimum threshold for closure is **4/5 on every dimension**. A single dimension at **3/5 blocks closure regardless of overall average**.

## Meaning of `docs_quality` (Stage-7)

`docs_quality` means:

- decision-record quality
- operator-facing clarity
- readiness for Stage-8 documentation

It does **NOT** mean final Stage-8 docs. Stage-8 happens after a CLOSED verdict.

## Evidence Rules

1. Tool-backed dimensions come from `dev-quality-check` output.
2. Review-backed dimensions come from `dev-delivery-review`, `dev-expert-code-review`, and files changed.
3. Every score note must cite a tool output line, file path, or review observation.
4. Confirm Stage-7 exists in pipeline stages before scoring.
5. `dev-expert-code-review` is additional high-signal evidence, never a replacement for tool-backed checks.

## Language Tool Defaults

- `python`
  - linting: flake8
  - formatting: black
  - import_hygiene: isort
  - type_safety: mypy

## Verdicts

| Verdict | Meaning |
|---|---|
| `CLOSED` | all blocking dimensions pass |
| `REMEDIATION REQUIRED` | one or more dimensions failed and cycles remain |
| `ESCALATED` | remediation cycles exhausted or manual judgment required |

## Remediation Routing (Ginshō -> Fuhyō)

| Failing dimension(s) | Routing |
|---|---|
| `linting`, `formatting`, `import_hygiene`, `type_safety`, `architecture` | Fuhyō executes via `dev-python-refactorer` |
| `test_quality` | Fuhyō executes via `dev-python-test-generator` and re-runs Stage-3 |
| `docs_quality` | Fuhyō executes via `doc-decision-record` |

## Output

```json
{
  "verdict": "",
  "scores": {},
  "thresholds_checked": [],
  "blocking_failures": [],
  "non_blocking_warnings": [],
  "remediation_cycle": 0,
  "remediation_guidance": []
}
```

## Stage-07 Result Schema (canonical)

Return a result object containing:

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

## Additional Hard Rule

- Never treat post-close Stage-8 docs as a Stage-7 closure prerequisite.
