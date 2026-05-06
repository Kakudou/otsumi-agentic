---
name: dev-opencti-create-connector
description: Scaffold a production-grade OpenCTI external-import connector through an interactive wizard, generating typed settings, API client, STIX mapping, state management, tests, docs, and local-only git-ready changes.
---

# dev-opencti-create-connector

Scaffold a complete OpenCTI external-import connector through a step-by-step wizard. Gather every required detail before writing files, generate a full project skeleton, add tests for every generated module, run local quality checks, and optionally prepare local commits through the git discipline skill. NEVER push.

## Usage

```text
/dev-opencti-create-connector --external-import
```

No additional flags required. The skill drives the interactive wizard.

## Required Companion Skills

- `/git-commits` — used ONLY if the user explicitly asks for local commits after generation.
- `/dev-run-tests` or project-equivalent test command — for post-generation test execution when available.
- `/dev-quality-check` or project-equivalent quality command — for lint/format/type checks when available.

If companion skills are unavailable, execute equivalent local commands and report the exact commands used.

## Hard Rules

- NEVER generate code before completing the full wizard.
- NEVER write files before the user confirms the generation summary.
- NEVER bundle wizard questions. Ask one decision point at a time.
- NEVER copy-paste from existing connectors verbatim, except for explicitly approved invariant infrastructure such as the reference `api_engine` package when the project policy permits it.
- NEVER fabricate missing reference files. If the reference connector or `api_engine` package cannot be found, stop and ask for the path or permission to generate an equivalent implementation.
- NEVER skip test generation. Every generated module gets a corresponding test file.
- NEVER mark the connector complete until mocked tests pass or a blocking failure is reported with evidence.
- NEVER commit automatically.
- NEVER push.
- ALWAYS use `connectors_sdk` models, settings, and state manager.
- ALWAYS use the `api_engine` package or its approved equivalent for HTTP calls with retry, circuit breaker, rate limiter, and hooks.
- ALWAYS generate typed Pydantic models for API responses.
- ALWAYS use raw pytest BDD helpers named `_given_*`, `_when_*`, and `_then_*`. Do not generate `.feature` files.
- ALWAYS ask the user to confirm before writing files.

## Stop Conditions

- STOP if the wizard is incomplete.
- STOP if the user chooses to edit a wizard section.
- STOP if the user does not confirm generation.
- STOP if the reference architecture path is required but missing.
- STOP if generated tests fail and the failure cannot be fixed without changing confirmed requirements.
- STOP before committing unless the user explicitly asks for local commits.

## Wizard Protocol

Run the wizard step by step. Ask one question or one tightly scoped confirmation at a time. NEVER ask a large bundle of unrelated questions.

Persist answers in a `connector_blueprint` object. NEVER write files until the final blueprint is confirmed.

### Step 1 — Connector Identity

Collect:

- Connector name, human-readable, for example `Acme Threat Intel`.
- Connector slug, kebab-case, for example `acme-threat-intel`. Suggest one from the name and let the user override it.
- Python package name, snake_case, for example `acme_threat_intel`. Derive from the slug and let the user confirm.
- Author organization used for `OrganizationAuthor`.
- Default TLP level: `clear`, `green`, `amber`, `amber+strict`, or `red`; default `amber`.
- Default `duration_period`, ISO-8601; default `PT1H`.

### Step 2 — API Endpoint

Collect:

- Base URL, for example `https://api.acme.com`.
- Authentication method: API key header, bearer token, OAuth2 service account, basic auth, or none.
- Relevant credential fields based on auth method.
- Primary endpoint path, for example `/v2/indicators`.
- HTTP method; default `GET`.
- Pagination strategy: offset-based, cursor-based, timestamp-sliding-window, or none.
- Known rate limit, optional.

### Step 3 — API Response Model

Ask the user for either:

- A sample JSON response, pasted as raw JSON.
- A description of the response shape if raw JSON is unavailable.

Then draft Pydantic v2 models:

- Use `ConfigDict(populate_by_name=True)`.
- Use camelCase aliases with `Field(alias="camelName")` where needed.
- Create top-level and nested models.
- Prefer explicit field types over `Any`.

Present the generated model outline for confirmation before proceeding.

### Step 4 — STIX Mapping

Collect:

- Primary STIX entity produced by the connector, selected from SDK models such as Incident, Indicator, Malware, ThreatActorGroup, Vulnerability, Report, ObservedData, Campaign, IntrusionSet, Infrastructure, or another valid SDK model.
- Observable types extracted from each record, such as IPV4Address, IPV6Address, DomainName, URL, File, Hostname, EmailAddress, or UserAccount.
- Relationship type between primary entity and observables; default `RELATED_TO`.
- Field mapping from API response fields to STIX fields, including name, severity, description, first_seen, last_seen, labels, and other relevant fields.
- Observable extraction rules for every selected observable type.

Present a mapping table and ask for confirmation.

### Step 5 — State Management

Collect:

- State fields to persist between runs, such as `last_event_timestamp` or `pagination_cursor`.
- First-run lookback, ISO-8601 duration; default `P1D`.
- Deduplication strategy: timestamp-based, cursor-based, or none.

### Step 6 — Settings Configuration

Draft the full settings model and config field list:

- Base URL.
- Credentials.
- Connector custom parameters.
- Duration period.
- Any pagination, state, or rate-limit fields.

Ask whether additional config fields are needed.

### Step 7 — Final Confirmation

Present a complete summary:

```json
{
  "connector_identity": {},
  "api_details": {},
  "response_models": {},
  "stix_mapping": {},
  "state_strategy": {},
  "settings_fields": {},
  "planned_files": []
}
```

Ask:

```text
Ready to generate? Yes, or edit a section?
```

Only `Yes` authorizes file writes.

## Connector Blueprint Schema

Maintain this blueprint throughout the wizard:

```json
{
  "connector_identity": {
    "name": "",
    "slug": "",
    "package_name": "",
    "class_name": "",
    "author_organization": "",
    "default_tlp": "amber",
    "duration_period": "PT1H"
  },
  "api": {
    "base_url": "",
    "auth_method": "",
    "auth_fields": {},
    "endpoint_path": "",
    "http_method": "GET",
    "pagination_strategy": "none",
    "rate_limit": null
  },
  "response_model": {
    "sample_json_received": false,
    "models": [],
    "aliases": []
  },
  "stix_mapping": {
    "primary_entity": "",
    "observable_types": [],
    "relationship_type": "RELATED_TO",
    "field_mapping": {},
    "observable_extraction_rules": []
  },
  "state": {
    "fields": [],
    "first_run_lookback": "P1D",
    "deduplication_strategy": "timestamp-based"
  },
  "settings": {
    "fields": []
  }
}
```

## Generation Phase

After final confirmation, generate this structure:

```text
external-import/<connector-slug>/
├── .dockerignore
├── .env.sample
├── Dockerfile
├── README.md
├── config.yml.sample
├── docker-compose.yml
├── entrypoint.sh
├── pyproject.toml
├── src/
│   ├── main.py
│   ├── requirements.txt
│   └── <package_name>/
│       ├── __init__.py
│       ├── connector.py
│       ├── converter_to_stix.py
│       ├── client_api.py
│       ├── settings.py
│       ├── state_manager.py
│       ├── models/
│       │   ├── __init__.py
│       │   └── <response_model>.py
│       ├── mappers/
│       │   ├── __init__.py
│       │   ├── _utils.py
│       │   └── <entity>_mapper.py
│       └── utils/
│           ├── __init__.py
│           ├── timestamps.py
│           └── api_engine/
│               ├── <api_engine package files>
│               └── README.md
└── tests/
    ├── __init__.py
    ├── conftest.py
    ├── test_main.py
    ├── test_connector_e2e.py
    ├── tests_connector/
    │   ├── test_connector.py
    │   └── test_state_manager.py
    ├── tests_client/
    │   └── test_client_api.py
    ├── tests_converter_stix/
    │   ├── test_converter.py
    │   └── mappers/
    │       └── test_<entity>_mapper.py
    └── tests_utils/
        └── <api_engine tests>
```

## File Generation Rules

### `main.py`

```python
"""Entrypoint for the <Connector Name> external-import connector."""

import traceback

from pycti import OpenCTIConnectorHelper

from <package_name> import ConnectorSettings, <ConnectorClass>

if __name__ == "__main__":
    try:
        settings = ConnectorSettings()
        helper = OpenCTIConnectorHelper(config=settings.to_helper_config())
        connector = <ConnectorClass>(config=settings, helper=helper)
        connector.run()
    except Exception:
        traceback.print_exc()
        exit(1)
```

### `settings.py`

- Inherit `BaseConnectorSettings` from `connectors_sdk`.
- Override `BaseExternalImportConnectorConfig` for connector-level config such as name and duration period.
- Create a custom `<Name>Config(BaseConfigModel)` for API URL, credentials, pagination, and connector-specific settings.
- Use `pydantic.Field` with descriptions and defaults.

### `state_manager.py`

- Inherit `ConnectorStateManager` from `connectors_sdk`.
- Add typed fields such as `last_event_timestamp: datetime | None`.
- Override `save()` to dump declared fields with `exclude_none=True` and merge `model_extra`.

### `client_api.py`

- Use `api_engine.ApiClient` with `RetryRequestStrategy`.
- Implement authentication with a `BaseRequestHook` subclass when auth is required.
- Expose an async generator for paginated fetching.
- Expose `close()` that delegates to `self._api_client.close()`.

### `connector.py`

Use single-responsibility methods:

- `_resolve_time_window`
- `_log_run_started`
- `_convert_batch`
- `_send_bundle`
- `_finalize_run`

Required behavior:

- `process_message()` is the entry point for scheduled processing.
- `_collect_intelligence()` wraps `asyncio.run(self._async_process_message(state))`.
- Structured logging uses a `[CONNECTOR]` prefix.
- Logs include batch counts, type summaries, state changes, and dedup indicators.

### `converter_to_stix.py`

- Create `OrganizationAuthor` and `TLPMarking` in `__init__`.
- Provide `convert_<entity>()` methods that call individual mappers.
- Return a flat list of STIX objects.

### `mappers/`

- One file per STIX entity type.
- Pure mapping functions shaped like `map_<entity>(data, *, author, tlp_marking) -> <SDKModel>`.
- Use `_utils.py` for shared extraction helpers.

### `models/`

- Pydantic v2 models.
- Use `ConfigDict(populate_by_name=True)`.
- Use aliases for camelCase API fields.
- Include top-level response model and nested models.

### `utils/timestamps.py`

```python
"""Shared timestamp parsing utilities."""

from datetime import datetime


def parse_ts(ts: str) -> datetime:
    """Parse an ISO-8601 timestamp to a timezone-aware datetime."""
    return datetime.fromisoformat(ts.replace("Z", "+00:00"))
```

### `utils/api_engine/`

Copy the full `api_engine` package from the approved reference connector only if it is available and policy permits reuse. This is treated as invariant infrastructure.

Include all interfaces, exceptions, implementations, tests, and README from the approved reference package. If direct copy is not permitted, generate an equivalent package with the same public behavior and tests.

## Test Generation Rules

- Use raw pytest with `_given_*`, `_when_*`, and `_then_*` helper functions.
- Use `polyfactory` for Pydantic model factories.
- Mock external HTTP calls with `unittest.mock.patch` on the strategy `execute` method.
- Use `MagicMock(spec=OpenCTIConnectorHelper)` for the helper.
- Add an autouse fixture to clear `RateLimiterRegistry` between tests when the rate limiter is present.
- Cover every mapper, client pagination, state transitions, and one end-to-end connector test.
- Never use `pass`, `assert True`, or `NotImplementedError`.
- Tests must perform real assertions.

### Raw Pytest BDD Pattern

```python
"""Unit tests for <Module> using _given_/_when_/_then_ BDD helpers."""

from unittest.mock import MagicMock

# ===========================================================================
# Helpers
# ===========================================================================


def _given_<setup>(args) -> <Type>:
    """Set up precondition."""
    ...


def _when_<action>(subject, args) -> <Result>:
    """Execute the action under test."""
    ...


def _then_<assertion>(result, expected) -> None:
    """Assert the expected outcome."""
    assert result == expected

# ===========================================================================
# Test: <Scenario Name>
# ===========================================================================


def test_<scenario_snake_case>() -> None:
    """<One-line description of the scenario>."""
    # _given_ <precondition>
    thing = _given_<setup>(...)

    # _when_ <action>
    result = _when_<action>(thing, ...)

    # _then_ <expected outcome>
    _then_<assertion>(result, expected_value)
```

## Infrastructure File Rules

### `pyproject.toml`

Use Poetry build system and include:

- Runtime dependencies: `pycti`, `pydantic`, `aiohttp`, `connectors-sdk`, plus API-specific libraries only when needed.
- Test dependencies: `pytest`, `pytest-asyncio`, `pytest-cov`, `polyfactory`.
- Quality tools: `ruff`, `black`, `isort`, `mypy`, `flake8` when compatible with the host project.
- Tool configuration for ruff, mypy, black, isort, flake8, and pytest.

### `Dockerfile`

- Base image: `python:3.12-alpine` unless the host project requires another supported base.
- Install from `requirements.txt` or Poetry export.
- Entrypoint runs `main.py`.

### `docker-compose.yml`

- Include all environment variables with `ChangeMe` placeholders.
- Comment optional fields.

### `config.yml.sample`

- YAML equivalent of environment variables.

### `.env.sample`

- Flat environment variable list.

### `.dockerignore`

Exclude:

- `__metadata__`
- `__pycache__`
- `.venv*`
- `tests/`
- `config.yml`

### `entrypoint.sh`

```sh
cd /opt/opencti-connector-<slug> && python3 main.py
```

### `README.md`

Include:

- Connector name.
- Description.
- Configuration table.
- Deployment instructions.
- Test instructions.
- Local development commands.

## Error Handling Rules

- Wrap async processing with specific handling for authentication/configuration errors when detectable.
- Log helpful credential/configuration guidance without leaking secrets.
- Log tracebacks for generic exceptions.
- Always close the API client in `finally`.
- Use clean `sys.exit(0)` behavior for `KeyboardInterrupt` and `SystemExit` in `process_message`.

## Observability Rules

Every connector must log:

- `[CONNECTOR] Connector last run` or `[CONNECTOR] Connector has never run...`
- `[CONNECTOR] Run started` with time window and first-run flag.
- `[CONNECTOR] Batch fetched` with counts.
- `[CONNECTOR] Batch converted to STIX` with total and approximate unique STIX counts plus type summary.
- `[CONNECTOR] Bundle sent` with work ID and counts.
- `[CONNECTOR] State updated` with new timestamp or cursor.
- `[CONNECTOR] Run completed` with totals.

Use `self.helper.connector_logger.info(message, attributes_dict)` for structured logging.

## Reference Architecture

Use `external-import/google-secops-siem-incidents/` as the structural reference when available.

It demonstrates:

- Settings through `connectors_sdk`.
- Typed state manager.
- `api_engine` for HTTP.
- Pydantic response models.
- Mapper-per-entity pattern.
- Structured logging with dedup counts.
- Async pagination.
- Comprehensive raw pytest BDD tests.

Do not copy connector-specific business logic from the reference connector.

## Post-Generation Validation

Run, or route to companion skills for, the following checks:

```text
python -m pytest tests/ -q
ruff check src/
black --check src/ tests/
```

If project configuration uses different equivalent commands, use the project commands and report them.

Return:

```json
{
  "generated_project_path": "external-import/<connector-slug>",
  "files_written": [],
  "tests": {
    "command": "",
    "passed": false,
    "summary": ""
  },
  "quality": {
    "commands": [],
    "passed": false,
    "summary": ""
  },
  "blocking_failures": [],
  "commit_offer_required": true
}
```

## Commit Integration

After generation and validation, ask whether the user wants local commits.

If the user says yes:

- Use `/git-commits`.
- Provide the generated project path and a logical commit plan.
- Never push.
- Prefer atomic commits with gitmoji plus Conventional Commit format.

Suggested commit sequence:

```text
🎉 feat(project): scaffold <connector-name> connector structure
📦 chore(project): add tooling and dependency manifests
✨ feat(settings): add connector configuration with pydantic
✨ feat(state): add typed state manager
✨ feat(api-engine): add resilient HTTP client engine
✨ feat(models): add API response pydantic models
✨ feat(client-api): add <API> client with auth and pagination
🧬 feat(mappers): add <entity> and observable mappers
✨ feat(converter): add STIX converter orchestrator
✨ feat(connector): add main connector pipeline
✅ test(all): add BDD tests for all components
📝 docs(readme): add connector README
📝 docs(api-engine): add api_engine README
```
