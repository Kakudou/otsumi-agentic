---
name: env-setup
description: Prepare the runtime for the project's language and dependencies.
---

Prepare the local environment for the selected execution stack.

## Usage

- `/env-setup --lang <language-id>`
- `/env-setup <feature-name>` to read pipeline state

## Steps

1. Resolve `language_id` from an explicit `--lang` flag first, otherwise from `.otsumi/<feature-name>/pipeline.json`.
2. If `language_id` is missing: stop and report valid usage.
3. Set up the runtime based on the project's language and dependencies:
   - Detect the package manager and dependency file (e.g., `pyproject.toml`, `requirements.txt`, `setup.py`).
   - Create an isolated environment if the language supports it (e.g., virtual environment for Python).
   - Install the project's declared dependencies plus any test framework dependencies detected from the project configuration.
4. Report the prepared runtime, resolved language, and resolved environment.

## Hard Rules

- Never infer selectors from natural language alone.
- Never claim a runtime is ready without checking.

## Python Implementation

1. Create a python virtual environment `python -m venv .venv.otsumi --prompt otsumi`
2. Create the dependency file `pyproject.toml` and set it up for `poetry 2`
2. Using that virtual environment, install the project's dependencies and test framework dependencies. 
  3. Test framework dependencies should contains at least: 
  - pytest
  - pytest-bdd
  - mypy
  - flake8
  - black
  - isort
3. Configure the test framework dependencies in the virtual environment:
  4. flake8 --ignore=W,E --line-length=88
  5. black --line-length=88
  6. isort --line-length=88 --profile=black
