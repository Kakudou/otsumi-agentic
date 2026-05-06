---
name: dev-delivery-review
description: "Review implementation delivery quality across architecture, maintainability, tests, documentation readiness, and scope discipline."
---

# Dev Delivery Review

Produce a structured delivery review.

## Usage

`/dev-delivery-review <feature-name>`

## Purpose

Evaluate whether the delivered work is coherent, maintainable, scoped, and ready for final scoring.

## Hard Rules

- NEVER treat passing tests as sufficient by itself.
- NEVER invent architecture that is not visible in files.
- NEVER ignore scope creep or gold plating.
- Cite concrete files, artifacts, or observations.

## Review Areas

- Architecture fit
- Scope discipline
- Test quality
- Maintainability
- Error handling
- Documentation readiness
- Operational risks

## Output

```json
{
  "architecture": {"score": 0, "evidence": []},
  "scope_discipline": {"score": 0, "evidence": []},
  "test_quality": {"score": 0, "evidence": []},
  "maintainability": {"score": 0, "evidence": []},
  "risks": [],
  "recommendations": []
}
```
