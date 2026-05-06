---
name: core-status
description: "Display current status for one workflow/pipeline or summarize all workflow state roots."
---

# Core Status

Read-only view of workflow or pipeline status from persisted state.

## Usage

- `/core-status`
- `/core-status <scope-name>`

## Hard Rules

- NEVER mutate state.
- NEVER infer completion when required artifacts are missing.
- NEVER execute workflow stages.
- MUST report missing or malformed state explicitly.

## Steps

1. If `<scope-name>` is provided, read `.otsumi/<scope-name>/pipeline.json` or equivalent state root.
2. If no scope is provided, list `.otsumi/*/pipeline.json`.
3. For each state root, report:
   - name
   - status
   - mode (when present)
   - current stage
   - last completed stage
   - next stage
   - started/resumed timestamps
   - missing or malformed artifacts
4. Return a compact table plus warnings.
