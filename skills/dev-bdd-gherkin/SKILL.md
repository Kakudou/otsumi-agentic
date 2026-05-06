---
name: dev-bdd-gherkin
description: "Create Gherkin behavior specs and trap analysis with user approval gates, preserving domain language and promoting approved traps into scenarios."
---

# Dev BDD Gherkin

Create behavior specs and trap analysis.

## Usage

`/dev-bdd-gherkin <feature-name>`

## Purpose

Produce a domain-language Gherkin behavior contract and adversarial trap analysis before tests or implementation.

## Hard Rules

- NEVER include implementation details, language routing, storage choices, or test framework mechanics in Gherkin.
- NEVER write final spec scenarios without approval.
- NEVER drop approved traps.
- NEVER duplicate scenario output.
- Use business/domain language only.

## Steps

1. Read the feature request, project context, existing features, and vocabulary.
2. Draft scenarios using:
   - `Feature`
   - `Background` when useful
   - `Scenario`
   - `Given / When / Then / And`
3. Present scenarios for approval.
4. Run trap analysis:
   - missing negative cases
   - boundary values
   - invalid inputs
   - state conflicts
   - permission/authorization gaps
   - concurrency or ordering risks
   - ambiguous terms
5. Present traps for approval.
6. Promote approved traps into scenarios.
7. Write `features/<feature-name>.feature` when applicable.
8. Return Stage-1/Stage-2 artifacts or standalone output.

## Output

Return:

```json
{
  "feature_name": "",
  "scenarios": [],
  "traps": [],
  "approved_traps_promoted": [],
  "feature_file": ""
}
```
