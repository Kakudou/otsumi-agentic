---
name: delivery-review
description: Perform a language-aware delivery review for a feature or the full project.
---

Perform a structural review across architecture, test quality, and docs quality.

## Usage

- `/delivery-review <feature-name>`
- `/delivery-review --lang <language-id>`

## Purpose

Produce review-backed evidence for Stage-7 and for human engineering judgment.

## Steps

1. Resolve the review scope.
2. If feature-scoped, read `.otsumi/<feature-name>/pipeline.json` and extract:
   - `language_id`
   - `stages`
3. For full-project mode, require explicit `--lang`.
4. Resolve the review scope from `pipeline.json`'s `language_id` and `stages`. For full-project mode, use the explicit flags and discover project conventions from the repository structure.
5. Build the review file list from the pipeline's language and project configuration:
    - feature file when the project uses one
    - implementation files matching the project's source layout
    - test binding file when present
    - step file when present
    - support files when present
    - relevant decision records when present
    - documentation-readiness artifacts when present
6. Read every file in scope.
7. Evaluate three dimensions:
    - `architecture`
    - `test_quality`
    - `docs_quality`
8. In this command, `docs_quality` means decision-record quality, operator-facing clarity, and readiness for Stage-8 documentation. It does not mean final Stage-8 docs, which are written after a CLOSED verdict.
9. Cite every issue with a concrete file reference and a suggested fix.
10. If feature-scoped, log the review run through `/atomic-log`.

## Review Focus

Review criteria are adapted to the project's architecture and conventions. The reviewer should:
- Assess business logic placement and module boundaries
- Evaluate test coverage quality relative to the test framework in use
- Check for typed interfaces and domain models where the language supports them
- Verify separation of concerns and minimal module complexity

## Output Shape

Return a structured review with:

- `architecture`
- `test_quality`
- `docs_quality`

Each section should include:

- score or assessment label
- findings
- evidence citations
- concrete remediation guidance

## Hard Rules

- Review from the pipeline's language and project conventions, not from stale assumptions.
- Never invent files, checks, or evidence.
- Never treat post-close Stage-8 docs as a prerequisite for Stage-7 closure.
- Never skip a dimension silently.