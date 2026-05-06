---
name: dev-env-setup
description: "Prepare and verify the development runtime for a selected language and project without silently rewriting project architecture."
---

# Dev Env Setup

Verify runtime readiness before any implementation, tests, linting, formatting, typing, or dependency operation.

## Usage

`/dev-env-setup --lang <language_id> [--feature <feature-name>]`

## Hard Rules

- NEVER infer language from natural language alone.
- NEVER rewrite project configuration without explicit approval.
- NEVER claim readiness without verification.
- MUST prefer existing project config over creating new config.
- MUST report missing tools precisely.

## Steps

1. Resolve `language_id` from explicit flag or persisted workflow state.
2. Inspect project configuration:
   - Python: `pyproject.toml`, `setup.py`, `setup.cfg`, `.flake8`, `mypy.ini`, lockfiles, virtualenvs
3. Detect test/quality tools.
4. Verify command availability.
5. If setup changes are needed, propose them BEFORE applying.
6. Return:

```json
{
  "language_id": "",
  "runtime_ready": true,
  "commands": {},
  "warnings": [],
  "required_user_action": []
}
```
