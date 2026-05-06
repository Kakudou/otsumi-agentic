---
name: dev-delivery-review
description: "Review implementation delivery quality across architecture, maintainability, tests, documentation readiness, and scope discipline."
---

# Dev Delivery Review

Evaluate whether delivered work is coherent, maintainable, scoped, and ready for final scoring.

## Usage

`/dev-delivery-review <feature-name>`

## Hard Rules

- MUST cite concrete files, artifacts, or observations.
- NEVER treat passing tests as sufficient by themselves.
- NEVER invent architecture that is not visible in files.
- NEVER ignore scope creep or gold plating.

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
