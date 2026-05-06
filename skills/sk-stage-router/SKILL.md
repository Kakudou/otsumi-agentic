---
name: sk-stage-router
description: Generic routing facade for language-bound pipeline stages. Dispatches Stage-3, Stage-4, and Stage-5 to the correct language adapter.
---

# stage-router

Dispatch a pipeline stage to the correct language adapter. NEVER execute stage logic directly.

## Hard Rules

- NEVER execute stage logic directly in this facade.
- NEVER silently fall back to another adapter.
- NEVER route to an adapter for a different language.
- NEVER bypass stage gating.
- NEVER proceed if `stage`, `feature_name`, or `language_id` are missing — halt immediately and report what is absent.

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
- `pipeline_fields` - optional passthrough: any extra fields from `pipeline.json` are forwarded to the adapter unchanged

## Routing Table

| Stage | Language | Adapter |
|-------|----------|---------|
| stage-03 | python | python-test-generator |
| stage-04 | python | python-implementer |
| stage-05 | python | python-refactorer |

## Activation Steps

1. VERIFY `stage`, `feature_name`, `language_id` are present. Halt with explicit error if any are missing.
2. VERIFY the stage is in the pipeline's stages list. Halt if not found.
3. Resolve the adapter from `stage + language_id`.
4. If no adapter exists: halt and report the missing combination explicitly. NEVER guess or substitute.
5. Invoke the adapter with the relevant stage artifacts plus all `pipeline_fields` passed through unchanged.
6. Return the adapter result unchanged to the caller.
