---
name: delivery-review
description: Perform a language-aware delivery review for a feature or the full project.
---

Perform a structural review across architecture, test quality, and docs quality.

## Hard Rules

- NEVER invent files, checks, or evidence.
- NEVER treat post-close Stage-8 docs as a prerequisite for Stage-7 closure.
- NEVER skip a dimension silently.
- ALWAYS review from the pipeline's language and project conventions, not from stale assumptions.

## Usage

- `/delivery-review <feature-name>`
- `/delivery-review --lang <language-id>`

## Purpose

Produce review-backed evidence for Stage-7 and for human engineering judgment.

## Steps

1. Resolve the review scope:
   - Feature-scoped: read `.otsumi/<feature-name>/pipeline.json`, extract `language_id` and `stages`.
   - Full-project mode: require explicit `--lang`; discover project conventions from the repository structure.
   - If neither is available: halt and report valid usage.
2. Build the review file list from the pipeline's language and project configuration:
   - feature file when the project uses one
   - implementation files matching the project's source layout
   - test binding file when present
   - step file when present
   - support files when present
   - relevant decision records when present
   - documentation-readiness artifacts when present
3. Read every file in scope.
4. Evaluate three dimensions: `architecture`, `test_quality`, `docs_quality`.
5. `docs_quality` covers decision-record quality, operator-facing clarity, and readiness for Stage-8 documentation — NOT final Stage-8 docs, which are written only after a CLOSED verdict.
6. Cite every issue with a concrete file reference and a suggested fix.
7. If feature-scoped, log the review run through `/atomic-log`.

## Review Focus

ALWAYS adapt review criteria to the project's architecture and conventions:
- Assess business logic placement and module boundaries.
- Evaluate test coverage quality relative to the test framework in use.
- Check for typed interfaces and domain models where the language supports them.
- Verify separation of concerns and minimal module complexity.

## Output Shape

Return a structured review with three sections: `architecture`, `test_quality`, `docs_quality`.

Each section MUST include:
- score or assessment label
- findings
- evidence citations
- concrete remediation guidance