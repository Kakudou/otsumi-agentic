---
name: dev-stage-router
description: "Route language-bound development stages to the correct adapter skill using explicit stage and language selectors."
---

# Dev Stage Router

Route language-bound stages to adapter skills. Preserve adapter routing for Stage-3, Stage-4, and Stage-5 until language adapters are fully manifest-driven.

## Usage

`/dev-stage-router --stage <stage-id> --lang <language_id> <payload>`

## Hard Rules

- NEVER infer language from natural language.
- NEVER route a stage that is not in the active workflow state.
- NEVER call an adapter that does not exist.
- NEVER transform payload semantics — only route and normalize.

## Routing Table

| Language | Stage | Adapter |
|---|---|---|
| `python` | `stage-03` | `/dev-python-test-generator` |
| `python` | `stage-04` | `/dev-python-implementer` |
| `python` | `stage-05` | `/dev-python-refactorer` |

## Steps

1. Validate `stage`, `language_id`, and payload.
2. Check the stage is active when pipeline state is provided.
3. Resolve adapter from the table.
4. Invoke the adapter with the original payload plus normalized routing fields.
5. Return the adapter result unchanged except for a routing envelope.
