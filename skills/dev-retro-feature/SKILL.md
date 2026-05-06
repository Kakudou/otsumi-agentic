---
name: dev-retro-feature
description: "Reverse-engineer existing code or behavior into feature candidates, gap reports, and Gherkin-ready specs without mutating source."
---

# Dev Retro Feature

Recover feature specs from existing implementation.

## Usage

`/dev-retro-feature <path-or-feature-area>`

## Purpose

Turn legacy or undocumented behavior into explicit feature candidates and testable specs.

## Hard Rules

- NEVER mutate source files.
- NEVER invent behavior not grounded in code, tests, docs, or runtime evidence.
- NEVER claim full coverage when inspected evidence is partial.
- Preserve uncertainty explicitly.

## Steps

1. Inspect provided paths, tests, docs, and project structure.
2. Identify observable behaviors.
3. Group behaviors into feature candidates.
4. Detect gaps:
   - missing tests
   - unclear behavior
   - hidden coupling
   - undocumented decisions
5. Draft Gherkin-ready scenarios for confirmed behavior.
6. Return a source-grounded report.
