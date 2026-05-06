---
name: flow-diff-stage
description: "Show the artifacts and file changes associated with one stage output."
---

# Flow Diff Stage

Inspect what changed at a stage.

## Usage

`/flow-diff-stage <feature-name> <stage-id>`

## Purpose

Provide transparent, read-only visibility into stage changes.

## Hard Rules

- NEVER mutate files.
- NEVER infer changed files if the artifact does not record them; report missing metadata.
- NEVER hide missing artifacts.

## Steps

1. Read the stage output JSON.
2. Extract:
   - files written
   - files modified
   - decisions created
   - tests added
   - docs added
   - metrics
3. If git is available, optionally show diff for referenced files.
4. Return a concise report.
