---
name: dev-expert-code-review
description: "Perform a deep expert code review with concrete findings, risk levels, and scorecard evidence."
---

# Dev Expert Code Review

Find subtle problems beyond mechanical quality checks.

## Usage

`/dev-expert-code-review {feature-name}`

## Hard Rules

- MUST back every finding with concrete evidence — NEVER vague criticism.
- MUST separate blockers from non-blocking improvements.
- NEVER invent code behavior.
- NEVER rewrite the code directly unless the caller requested an edit task.

## Review Dimensions

- correctness
- architecture
- maintainability
- security
- performance
- tests
- operational safety

## Output

```json
{
  "blocking_findings": [],
  "non_blocking_findings": [],
  "scorecard": {},
  "recommended_actions": []
}
```
