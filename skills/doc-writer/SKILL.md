---
name: doc-writer
description: "Generate human-facing documentation from real artifacts without inventing behavior."
---

# Doc Writer

Write documentation from evidence.

## Usage

`/doc-writer <feature-name>`

## Purpose

Generate documentation humans can use: README sections, changelog entries, API docs, runbooks, blog-style explanations, or notes, depending on requested output.

## Hard Rules

- NEVER invent behavior not present in artifacts.
- NEVER document unapproved or suppressed scope as delivered.
- NEVER overwrite existing docs without preserving user content.
- Cite source artifacts internally in the work process.

## Steps

1. Read workflow state, specs, tests, decisions, and review outputs.
2. Determine doc targets requested by caller.
3. Draft docs using only verified behavior.
4. Preserve existing project tone when present.
5. Return files written and source artifacts used.
