---
name: dev-bdd-gherkin
description: "Create Gherkin behavior specs and trap analysis with user approval gates, preserving domain language and promoting approved traps into scenarios."
---

# Dev BDD Gherkin

Produce a domain-language Gherkin behavior contract and adversarial trap analysis before tests or implementation.

## Usage

`/dev-bdd-gherkin {feature-name}`

## Hard Rules

- MUST use business/domain language only.
- MUST gate scenario writing on user approval — NEVER write final scenarios without approval.
- MUST gate trap promotion on approval — NEVER drop approved traps.
- NEVER include implementation details, language routing, storage choices, or test framework mechanics in Gherkin.
- NEVER duplicate scenario output.

## Steps

1. Read the feature request, project context, existing features, and vocabulary. When `.otsumi/{feature-name}/kinsho-contract.json` exists, read it FIRST — it is the PO contract; Gherkin scenarios MUST be derived from its acceptance criteria, thresholds, and definition-of-done.
2. Draft scenarios using `Feature`, optional `Background`, `Scenario`, `Given / When / Then / And`.
3. Present scenarios for approval.
4. Run trap analysis covering:
   - missing negative cases
   - boundary values
   - invalid inputs
   - state conflicts
   - permission/authorization gaps
   - concurrency or ordering risks
   - ambiguous terms
5. Present traps for approval.
6. Promote approved traps into scenarios.
7. Write `features/{feature-name}.feature` when applicable.
8. Return Stage-1/Stage-2 artifacts or standalone output.

## Output

```json
{
  "feature_name": "",
  "scenarios": [],
  "traps": [],
  "approved_traps_promoted": [],
  "feature_file": ""
}
```
