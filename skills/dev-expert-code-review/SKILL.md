---
name: dev-expert-code-review
description: "Perform a deep expert code review with concrete findings, risk levels, and scorecard evidence."
---

# Dev Expert Code Review

Run a deep code review.

## Usage

`/dev-expert-code-review <feature-name>`

## Purpose

Find subtle problems beyond mechanical quality checks.

## Hard Rules

- NEVER provide vague criticism without evidence.
- NEVER invent code behavior.
- NEVER rewrite the code directly unless the caller requested an edit task.
- Separate blockers from non-blocking improvements.

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
