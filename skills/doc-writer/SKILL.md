---
name: doc-writer
description: "Generate human-facing documentation from real artifacts without inventing behavior."
---

# Doc Writer

Generate documentation humans can use — README sections, changelog entries, API docs, runbooks, blog-style explanations, or notes — from verified artifacts.

## Usage

`/doc-writer {feature-name}`

## Hard Rules

- NEVER invent behavior not present in artifacts.
- NEVER document unapproved or suppressed scope as delivered.
- NEVER overwrite existing docs without preserving user content.
- MUST cite source artifacts internally during the work process.

## Steps

1. Read workflow state, specs, tests, decisions, and review outputs.
2. Determine doc targets requested by the caller.
3. Draft docs using only verified behavior.
4. Preserve existing project tone when present.
5. Return files written and source artifacts used.
