---
name: sk-stage-router
description: Generic routing facade for language-bound pipeline stages. Dispatches Stage-3, Stage-4, and Stage-5 to the correct language adapter.
---

# stage-router

One job: dispatch a pipeline stage to the correct adapter without pretending all stacks behave the same.

## Inputs

- `stage` - required, one of `stage-03`, `stage-04`, `stage-05`
- `feature_name` - required
- `language_id` - required
- `feature_file` - required when the pipeline includes a feature file (stage-03)
- `test_file` - optional depending on stage (stage-04, stage-05)
- `steps_file` - optional depending on stage (stage-04, stage-05)
- `implementation_files` - required for stage-05
- `support_files` - optional
- `constraints` - optional (stage-04)
- `correction_brief` - optional, forwarded from the agent on re-invocation (stage-04)

## Routing Table

| Stage | Language | Adapter |
|-------|----------|---------|
| stage-03 | python | python-test-generator |
| stage-04 | python | python-implementer |
| stage-05 | python | python-refactorer |

## Activation Steps

1. Confirm `stage`, `feature_name`, `language_id` are present.
2. Confirm the stage is in the pipeline's stages list.
3. Resolve the adapter from `stage + language_id`.
4. If no adapter exists: halt and report the missing combination explicitly.
5. Invoke the adapter with the relevant stage artifacts.
6. Return the adapter result unchanged to the caller.

## Hard Rules

- Never execute stage logic directly in this facade.
- Never silently fall back to another adapter.
- Never route to an adapter for a different language.
- Never bypass stage gating.
