---
name: quality-score
description: "Score final output using evidence and thresholds, producing CLOSED, REMEDIATION REQUIRED, or ESCALATED verdicts."
---

# Quality Score

Produce an evidence-backed quality verdict.

## Usage

`/quality-score <feature-name>`

## Purpose

Aggregate quality evidence into a final verdict without vibes.

## Hard Rules

- NEVER score without evidence.
- NEVER pass if any blocking dimension is below threshold.
- NEVER average away a critical failure.
- ALWAYS list failing dimensions and remediation guidance.
- Stop after the configured maximum remediation cycles.

## Default Dimensions

- linting
- formatting
- import_hygiene
- type_safety
- architecture
- test_quality
- docs_quality

Default threshold: every dimension must be at least 4/5 unless the workflow defines otherwise.

## Verdicts

| Verdict | Meaning |
|---|---|
| `CLOSED` | all blocking dimensions pass |
| `REMEDIATION REQUIRED` | one or more dimensions failed and cycles remain |
| `ESCALATED` | remediation cycles exhausted or manual judgment required |

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
