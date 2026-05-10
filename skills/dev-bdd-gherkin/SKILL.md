---
name: dev-bdd-gherkin
description: "Create Gherkin behavior specs and trap analysis with user approval gates, preserving domain language and promoting approved traps into scenarios."
---

# Dev BDD Gherkin

Produce a domain-language Gherkin behavior contract and adversarial trap analysis before tests or implementation.
Optimize for deterministic, approval-gated behavior: draft scenarios first, collect explicit user decisions, then materialize artifacts.

## Usage

`/dev-bdd-gherkin --intent spec|trap {feature-name}`

## Parameters

- `--intent` (required): `spec` activates Stage-1 (scenario drafting from contract), `trap` activates Stage-2 (adversarial trap analysis on approved scenarios).

## Hard Rules

- MUST use business/domain language only.
- MUST gate scenario writing on user approval — NEVER write final scenarios without approval.
- MUST gate trap promotion on approval — NEVER drop approved traps.
- NEVER include implementation details, language routing, storage choices, or test framework mechanics in Gherkin.
- NEVER duplicate scenario output.

## Non-Negotiable Preservation Rules

- Hard rules above are immutable and always take precedence over convenience.
- Output contracts and stage result schemas are mandatory format contracts, not suggestions.
- Drift guardrails in Stage-1/Stage-2 flow (approval gates, rejection logging, promotion rules) MUST remain intact.

## Steps

1. Read the feature request, project context, existing features, and vocabulary. When `.otsumi/{feature-name}/kinsho-contract.json` exists, read it FIRST — it is the PO contract; Gherkin scenarios MUST be derived from its acceptance criteria, thresholds, and definition-of-done.
2. Draft scenarios using `Feature`, optional `Background`, `Scenario`, `Given / When / Then / And`.
3. Present scenarios for approval.
4. Run trap analysis covering:
   - missing negative cases
   - boundary values
   - invalid inputs
   - state conflicts
   - permission/authorization gaps
   - concurrency or ordering risks
   - ambiguous terms
5. Present traps for approval.
6. Promote approved traps into scenarios.
7. Write `features/{feature-name}.feature` when applicable.
8. Return Stage-1/Stage-2 artifacts or standalone output.

### Step Execution Constraints

- Keep scenario statements externally observable and business-verifiable.
- Keep each scenario outcome singular and unambiguous.
- When a scenario is rejected, preserve it in stage records with rationale; never silently replace or delete.

## Output

```json
{
  "feature_name": "",
  "scenarios": [],
  "traps": [],
  "approved_traps_promoted": [],
  "feature_file": ""
}
```

### Output Contract Notes

- `feature_name`: canonical slug or name used by the workflow.
- `scenarios`: finalized scenarios after approvals.
- `traps`: full trap inventory analyzed during Stage-2 (approved and rejected).
- `approved_traps_promoted`: subset of approved traps promoted into scenarios.
- `feature_file`: target path when file output is produced; empty string when not applicable.

## Gherkin Reference Material

### Keyword Inventory

| Keyword | Purpose |
|---|---|
| `Feature` | Top-level behavior container. Keep one feature per file. |
| `As a / I want / So that` | Optional narrative block under `Feature` that states actor, intent, and value. |
| `Background` | Shared preconditions that run before each scenario in the file. |
| `Scenario` | One concrete behavior example with one clear outcome. |
| `Scenario Outline` | Parameterized scenario that runs once per row in `Examples`. |
| `Examples` | Input table for a `Scenario Outline`. Header plus one or more rows. |
| `Given` | Declares the starting business context. |
| `When` | Describes the triggering business action or event. |
| `Then` | States the observable expected result. |
| `And` | Continues the previous step type without changing meaning. |
| `But` | Adds a contrasting/negative continuation of the previous step type. |

### Template Example 1: Full Feature with Background

```gherkin
Feature: Service appointment booking
    As a customer
    I want to reserve an appointment slot
    So that I can receive support at a known time

    Background:
        Given I am signed in
        And I am viewing available slots

    Scenario: Booking an available slot
        When I choose a free slot for tomorrow
        And I confirm the reservation
        Then the slot is reserved for me
        And I receive a booking confirmation

    Scenario: Trying to reserve an already taken slot
        Given the selected slot is no longer available
        When I confirm the reservation
        Then I see a message that the slot is unavailable
        But no reservation is created
```

### Template Example 2: Scenario Outline with Examples

```gherkin
Feature: Loyalty points redemption
    As a member
    I want to redeem points for rewards
    So that I can exchange earned value

    Scenario Outline: Redeeming points updates remaining balance
        Given my current points balance is <balance>
        When I redeem <redeem> points
        Then my remaining points should be <remaining>

        Examples:
            | balance | redeem | remaining |
            | 1000    | 200    | 800       |
            | 500     | 125    | 375       |
            | 250     | 250    | 0         |
```

### Template Example 3: Scenario with And/But Chains

```gherkin
Feature: Password reset request
    As an account holder
    I want to request a password reset link
    So that I can recover access safely

    Scenario: Requesting a reset with a registered email
        Given I am on the password reset page
        And I enter an email tied to an existing account
        When I submit the reset request
        Then I am told that reset instructions were sent
        And a reset message is queued for delivery
        But the response does not reveal whether the account is active
```

### Usage Notes

- Keep one `Feature` per `.feature` file.
- Use `Background` only when two or more scenarios share the same setup steps.
- Prefer adding `As a / I want / So that` so intent and value remain explicit.
- Use `But` when expressing contrast or negation; use `And` for neutral continuation.
- Keep strict domain language. Avoid implementation terms and technical internals.

### Quality Bar for Scenario Drafts

- Every `Then` MUST describe an observable business outcome.
- Scenario wording MUST avoid hidden technical assumptions.
- Domain terms MUST match project vocabulary or Kinshō contract terminology.

## Stage-1 Activation (Spec Drafting by Fuhyō)

Kakugyō orchestrates the pipeline. Fuhyō executes drafting. Ōshō is the only user-facing agent and presents each scenario to the user.

1. Read inputs (`feature_name`, `feature_description`, existing vocabulary, and optional Kinshō contract file).
2. Derive a human-readable `Feature:` title from the requested behavior.
3. Draft the candidate scenarios internally before writing any file.
4. Send scenarios to Ōshō one-by-one for user review and explicit response (`approve` / `reject` / `edit` / `split` / `merge`).
5. Enforce the rule: silence is not consent. Hold until explicit response per scenario.
6. Log every rejected scenario as `approved: false` with `rejection_reason` (never silently drop).
7. After all scenario decisions, request final explicit confirmation, then write `features/<feature-name>.feature`.

### Stage-1 Result Schema

```yaml
scenarios_total: <int>
scenarios_approved: <int>
scenarios_rejected: <int>
feature_file: features/<feature-name>.feature
```

## Stage-2 Activation (Trap Analysis by Fuhyō)

Fuhyō runs trap analysis against the approved feature. Keima may challenge completeness. Ōshō presents findings and receives decisions from the user.

### Canonical Trap Taxonomy (7 Categories)

| Category | Description |
|---|---|
| `invalid_input` | Missing, malformed, wrong-type, or unsupported user/business inputs. |
| `boundary` | Limits and edge conditions such as min/max, empty, singleton, and overflow-like behavior. |
| `security` | Unauthorized access, privilege misuse, data leakage, or abuse paths. |
| `concurrency` | Race conditions, duplicate actions, or ordering conflicts under simultaneous activity. |
| `performance` | Slow paths, high-volume strain, timeout risk, or resource saturation behavior. |
| `implicit_rule` | Unstated business constraints and assumptions not yet encoded as scenarios. |
| `data_validation` | Format, uniqueness, cross-field consistency, and referential validity checks. |

### Activation Steps (5)

1. Read the approved feature file (`features/<feature-name>.feature` or provided `feature_file`).
2. Interrogate all scenarios against all seven trap categories; never mark a category N/A without written reasoning.
3. Present the full trap summary first (all categories and severities).
4. Present traps individually via Ōshō in severity order (`critical` → `major` → `minor`) and require explicit response per trap.
5. Append approved `critical` and `major` traps as new scenarios under `# ---- Constraints identified ----`.

### Stage-2 Result Schema

```yaml
traps_total: <int>
traps_critical: <int>
traps_major: <int>
traps_minor: <int>
traps_approved: <int>
traps_rejected: <int>
scenarios_added: <int>
```

## Trap Approval Rules

- Minimum 2 approved traps are required; 3+ is preferred when supported by evidence.
- Silence is not consent: each trap requires explicit decision.
- Rejected traps are retained in stage records as `approved: false` with `rejection_reason`.
- Approved `critical` and `major` traps must be promoted into feature scenarios.

## Final Validation Checklist

- Hard rules preserved and enforced.
- Approval gates enforced for scenarios and traps.
- Rejection logging complete (`approved: false` + `rejection_reason`).
- Output contract shape preserved (JSON + stage schemas).
- Promoted traps are only approved `critical`/`major` items.
