---
name: sk-gherkin-keeper
description: Transforms a feature description into an approved Gherkin spec, then performs adversarial trap and constraint analysis. Standalone, no pipeline state required. Use with a feature name and description, or a path to an existing .feature file for trap analysis only.
---

# gherkin-keeper

Two stages. One continuous act of definition.
Stage-1: write the spec.
Stage-2: break it.
The feature is not defined until both are done and User has signed off on both.

---

## Inputs

Provided by whoever calls this skill (agent or direct invocation):

| Input | Required | Description |
|-------|----------|-------------|
| `feature_name` | yes | kebab-case name of the feature |
| `feature_description` | yes for Stage-1 | raw description of the feature to specify |
| `feature_file` | yes for Stage-2 standalone | path to an existing `.feature` file |

If `feature_file` is provided but `feature_description` is not: skip Stage-1, run Stage-2 only.
If `feature_description` is provided: run both stages sequentially.

> **Note:** This skill is intentionally stack-agnostic. `language_id`, are not inputs, Gherkin uses domain language only and does not vary by stack. The agent layer owns routing context; this skill ignores it.

---

## Outputs

| Output | Description |
|--------|-------------|
| `features/<feature-name>.feature` | written and approved Gherkin spec with trap scenarios |
| `stage-01-result` | scenario summary, returned to caller, not written to disk by this skill |
| `stage-02-result` | trap summary, returned to caller, not written to disk by this skill |

The caller (agent or Otsumi) is responsible for writing stage output JSONs and logging.

---

## Gherkin Templates Reference

The canonical keyword vocabulary and structure for `.feature` files produced by this skill.
Every template below is a valid, self-contained example. Use them as the structural blueprint when drafting scenarios.

### Keyword Inventory

| Keyword | Purpose |
|---------|---------|
| `Feature` | Top-level container. One per file. Names the capability under spec. |
| `As a / I want / So that` | Optional description block directly under `Feature`. States the actor, intent, and value. Not parsed as steps, pure human context. |
| `Background` | Steps that run before every scenario in the file. Shared preconditions only. Never put assertions here. |
| `Scenario` | A single concrete example of behavior. One path, one outcome. |
| `Scenario Outline` | A parameterized scenario. Runs once per row in `Examples`. Use angle brackets `<param>` for placeholders. |
| `Examples` | Data table bound to a `Scenario Outline`. Pipe-delimited columns. One header row, N data rows. |
| `Given` | Establish the world state before the action. |
| `When` | The action or event under test. |
| `Then` | The expected observable outcome. |
| `And` | Continues the previous `Given`, `When`, or `Then`. |
| `But` | Negating continuation of the previous step type. Reads as "and NOT". |

### Template: Full-Form Feature with Description and Background

```gherkin
Feature: Article publishing
    As a blog author
    I want to publish my articles to the site
    So that readers can access my content

    Background:
        Given I am logged in as an author
        And I have a draft article

    Scenario: Successfully publishing an article
        When I go to the article page
        And I press the publish button
        Then the article should be published
        And I should see a confirmation message

    Scenario: Attempting to publish without a title
        Given the article has no title
        When I press the publish button
        Then I should see the error "Title is required"
        But the article should not be published
```

### Template: Scenario Outline with Examples

```gherkin
Feature: Inventory management
    As a warehouse manager
    I want to track cucumber stock levels
    So that I can ensure accurate inventory counts

    Scenario Outline: Eating cucumbers reduces stock
        Given there are <start> cucumbers
        When I eat <eat> cucumbers
        Then I should have <left> cucumbers

        Examples:
            | start | eat | left |
            |    12 |   5 |    7 |
            |    20 |   5 |   15 |
            |     1 |   1 |    0 |
```

### Template: Scenario with And / But Step Chains

```gherkin
Feature: User registration
    As a new visitor
    I want to create an account
    So that I can access member features

    Scenario: Registering with valid information
        Given I am on the registration page
        And I have not registered before
        When I fill in my name, email, and password
        And I accept the terms of service
        And I submit the registration form
        Then my account should be created
        And I should receive a welcome email
        But I should not be subscribed to the newsletter by default
```

### Usage Notes

- **One `Feature` per file.** The feature title is human-readable, derived from the description, not kebab-case.
- **`Background` is optional.** Use it only when ≥ 2 scenarios share identical setup steps. If only one scenario needs it, inline the steps.
- **`As a / I want / So that`** is strongly recommended. It anchors the spec to a real actor and a real reason. Omit only when the context is self-evident from the feature title.
- **`But`** is syntactically identical to `And`. Use it when the step negates or contrasts with the previous expectation, it signals intent to the reader.
- **Domain language only.** No implementation terms (`API`, `database`, `endpoint`, `model`, `schema`, `service`, `class`, `function`, `object`, `method`). Write steps a non-technical stakeholder can read and challenge.

---

## Stage-1: Gherkin Spec

### Rules

- Domain vocabulary only: zero implementation terms like `class`, `function`, `API`, `database`, `endpoint`, `model`, `schema`, `service`, `object`, `method`
- Minimum 1 happy path scenario
- Use `Scenario Outline` + `Examples` for multiple data variations
- Steps must be written in plain business language, readable by a non-technical stakeholder
- Every scenario must be approved individually before the `.feature` file is written
- Silence is not consent, hold for explicit response on each scenario

### Activation Steps

1. Confirm `feature_name` and `feature_description` are provided by the caller.

2. Derive the feature title from the description, human-readable, not kebab-case.

3. Draft all scenarios internally. Do not write the `.feature` file yet.

4. Present each scenario to User one at a time:
   - Show: scenario name, Given/When/Then steps
   - Wait for: `approve` / `reject` / `edit` / `split` / `merge`
   - Incorporate feedback before moving to the next scenario
   - Rejected scenarios are logged as `approved: false` with a `rejection_reason`, not silently dropped

5. After all scenarios are approved, present the full spec for final confirmation.
   Wait for explicit approval before writing.

6. Write `features/<feature-name>.feature`.

7. Return `stage-01-result` to the caller:
```
scenarios_total: <n>
scenarios_approved: <n>
scenarios_rejected: <n>
feature_file: features/<feature-name>.feature
```

The caller writes the stage output JSON and runs the commands `/atomic-log` if in pipeline context.

---

## Stage-2: Trap & Constraint Analysis

### Rules

- Minimum 2 approved traps. 3 preferred.
- Never mark a category N/A without explicit written reasoning
- Every `critical` or `major` trap that is approved → scenario added to `.feature`
- Present the full trap summary first, then each trap individually in severity order: critical → major → minor
- Wait for explicit approval on each trap before writing anything
- Rejected traps are logged with `approved: false` and a `rejection_reason`, not silently dropped

### Trap Categories

Interrogate the feature against all 7. Do not skip any.

| # | Category         | What to look for                                              |
|---|------------------|---------------------------------------------------------------|
| 1 | invalid_input    | malformed, missing, wrong type, unexpected format             |
| 2 | boundary         | limits, min/max, empty, single item, overflow                 |
| 3 | security         | unauthorized access, privilege escalation, injection, exposure|
| 4 | concurrency      | race conditions, duplicate submissions, parallel state        |
| 5 | performance      | large volume, slow path, timeout, resource exhaustion         |
| 6 | implicit_rule    | unstated business rules, assumed preconditions                |
| 7 | data_validation  | format constraints, referential integrity, uniqueness         |

### Activation Steps

1. Read `features/<feature-name>.feature`: either just written in Stage-1 or provided as `feature_file` input.

2. Interrogate the feature against all 7 trap categories. Draft findings internally.

3. Present the full trap summary to User (all traps, all severities at a glance).
   Wait for acknowledgment before proceeding to individual review.

4. Present each trap individually in severity order:
   - Show: category, severity, description, proposed scenario name
   - Wait for: `approve` / `reject` / `edit` / `severity-change` / `demote`
   - Incorporate feedback before moving to the next trap

5. For each approved `critical` or `major` trap: append a scenario to `features/<feature-name>.feature` under a `# ---- Constraints identified ----` comment line.

6. Return `stage-02-result` to the caller:
```
traps_total: <n>
traps_critical: <n>
traps_major: <n>
traps_minor: <n>
traps_approved: <n>
traps_rejected: <n>
scenarios_added: <n>
```

The caller writes the stage output JSON and runs the commands `/atomic-log` if in pipeline context.

---

## Handoff

Return both results to the caller.

Report:
> Stage-1 and Stage-2 complete for `<feature-name>`.
> `.feature` written and approved. Traps documented and approved.
