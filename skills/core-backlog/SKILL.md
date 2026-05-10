---
name: core-backlog
description: "Read and manage planned feature or task records produced by workflow escalation, without starting them automatically."
---

# Core Backlog

Read and manage planned work records without executing them.

Record creation source: planned feature records are created by the `dev-bdd-workflow` skill's Feature Backlog Protocol (recorded by Kakugyō as `.otsumi/<feature-name>-planned.json`).

## Usage

- `/core-backlog`
- `/core-backlog {scope-name}`

## Hard Rules

- NEVER auto-start planned work.
- NEVER convert a planned item into a running workflow without an explicit user command.
- MUST preserve origin and source context.
- MUST report stale or malformed planned records.

## Steps

1. Search `.otsumi/*-planned.json` and `.otsumi/**/planned*.json` where applicable.
2. Read each planned record.
3. Group by:
   - source workflow/feature
   - status
   - origin
   - priority (when present)
4. Display a compact backlog with suggested explicit start commands.
