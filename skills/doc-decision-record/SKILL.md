---
name: doc-decision-record
description: "Create ADR/TDR decision records or explicit no-record reasoning based on actual implementation and workflow evidence."
---

# Doc Decision Record

Record architectural or technical decisions.

## Usage

`/doc-decision-record <feature-name>`

## Purpose

Ensure meaningful decisions survive the work. If no decision was made, record why no ADR/TDR is required.

## Hard Rules

- NEVER silently skip decision evaluation.
- NEVER create a fake ADR for a non-decision.
- NEVER invent context not present in artifacts.
- ALWAYS return either records created or no-record reasoning.

## Decision Types

- ADR: architecture decision
- TDR: technical decision

## Steps

1. Inspect behavior contract, implementation artifacts, tests, and review notes.
2. Detect decisions involving:
   - architecture
   - storage
   - integration
   - dependency
   - API/interface
   - security or operational tradeoff
3. Write records under `docs/decisions/` when required.
4. If no record is needed, return explicit reasoning.
