---
name: env-setup
description: Prepare the runtime for the project's language and dependencies.
---

Prepare the local environment for the selected execution stack.

## Hard Rules

- NEVER infer selectors from natural language alone.
- NEVER claim a runtime is ready without verifying it.

## Usage

- `/env-setup --lang <language-id>`
- `/env-setup <feature-name>` to read pipeline state

## Steps

1. Resolve `language_id`: use explicit `--lang` first; otherwise read `.otsumi/<feature-name>/pipeline.json`.
2. If `language_id` is missing: halt and report valid usage.
3. Set up the runtime:
   - Detect the package manager and dependency file (e.g., `pyproject.toml`, `requirements.txt`, `setup.py`).
   - Create an isolated environment if the language supports it (e.g., virtual environment for Python).
   - Install declared project dependencies plus test framework dependencies from the project configuration.
4. Report the prepared runtime, resolved language, and resolved environment.

## Python Implementation

1. Create a Python virtual environment: `python -m venv .venv.otsumi --prompt otsumi`
2. Create `pyproject.toml` configured for `poetry 2`.
3. Using that virtual environment, install project dependencies and test framework dependencies.
   - Test framework MUST include at least: `pytest`, `pytest-bdd`, `mypy`, `flake8`, `black`, `isort`
4. Configure tools in the virtual environment:
   - `flake8 --ignore=W,E --line-length=88`
   - `black --line-length=88`
   - `isort --line-length=88 --profile=black`
