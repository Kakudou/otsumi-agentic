---
name: core-backlog
description: "List planned feature or task records produced by workflow escalation, without starting them automatically."
---

# Core Backlog

List planned work records.

## Usage

- `/core-backlog`
- `/core-backlog <scope-name>`

## Purpose

Show planned items recorded by workflows, especially items intentionally suppressed from current scope and queued for later.

## Hard Rules

- NEVER auto-start planned work.
- NEVER convert a planned item into a running workflow without explicit user command.
- Preserve origin and source context.
- Report stale or malformed planned records.

## Steps

1. Search `.otsumi/*-planned.json` and `.otsumi/**/planned*.json` where applicable.
2. Read each planned record.
3. Group by:
   - source workflow/feature
   - status
   - origin
   - priority if present
4. Display a compact backlog with suggested explicit start commands.
