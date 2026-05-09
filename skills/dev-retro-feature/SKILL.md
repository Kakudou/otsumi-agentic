---
name: dev-retro-feature
description: "Reverse-engineer existing code or behavior into feature candidates, gap reports, and Gherkin-ready specs without mutating source."
---

# Dev Retro Feature

Recover feature specifications from existing implementation in **read-only mode**.

## Usage

`/dev-retro-feature {path-or-feature-area}`

## Mission

Produce source-grounded reverse-engineering artifacts:
- `feature_candidates`
- `gap_report`
- `draft_specs` (Gherkin-ready, confirmed behavior only)

## Hard Rules

- NEVER mutate source files.
- NEVER invent behavior not grounded in code, tests, docs, or runtime evidence.
- NEVER claim full coverage when inspected evidence is partial.
- MUST preserve uncertainty explicitly.

## Execution Steps

1. Inspect provided paths, tests, docs, and project structure.
2. Identify observable behaviors.
3. Group behaviors into feature candidates.
4. Detect gaps:
   - missing tests
   - unclear behavior
   - hidden coupling
   - undocumented decisions
5. Draft Gherkin-ready scenarios for confirmed behavior.
6. Return a source-grounded report.

## 3-Phase Methodology

### Phase 1: Source Scan and Feature Boundary Detection

Cluster by observable capability, never by file layout alone.

- Identify entry points (public APIs, routes, CLI handlers, event consumers).
- For each entry point, capture trigger, inputs, observable outcome, and explicit failure behavior.
- Group entry points into feature candidates when they share one business subject, lifecycle, or external surface.
- Flag entanglement when one entry point mixes unrelated concerns.
- Assign confidence: `high`, `medium`, `low`.

#### Cross-call state analysis (mandatory second pass)

Detect invariants by scanning for four architecture-agnostic signals:

1. **Shared persistent state**: one operation writes state another later reads.
2. **Probabilistic or weighted selection**: random/weighted/scored choice affecting behavior.
3. **Accumulators**: counters, histories, ratios, or matrices that evolve across calls.
4. **Implicit sequencing**: operation B only makes sense after A, even if not explicitly enforced.

When any signal is evidenced, emit an `invariant` feature candidate with:
- participating entry points
- signal evidence (`file:line`)
- plain-language user-visible effect
- confidence and uncertainty notes

### Phase 2: Gap Detection

Detect underspecified behavior before any scenario drafting.

| Gap Type | Detection Signal | Recommendation Strategy |
|---|---|---|
| `undefined_failure` | known failure path exists but handling is absent | `address now` |
| `partial_implementation` | TODO/stub/incomplete branch | `address now` |
| `implicit_precondition` | required precondition assumed but not validated | `defer to trap analysis` unless critical |
| `missing_boundary` | no explicit handling for null/empty/min/max/invalid boundaries | `defer to trap analysis` |
| `entangled_behavior` | mixed concerns prevent stable scenario boundary | `address now` (split/clarify boundary) |
| `side_effect_unspecified` | side effect exists but failure/disposition is unspecified | `address now` or explicit `accept undefined` |
| `unreachable_code` | branch cannot be activated through observable input | `accept undefined` and keep out of scenarios |

For each gap: capture `type`, `description`, `location`, and one recommendation (`address now` / `defer to trap analysis` / `accept undefined`).

### Phase 3: Gherkin Drafting

Draft only confirmed observable behavior using domain language.

#### Domain vocabulary recovery (noun-cluster translation)

Before drafting any step:
1. Extract implementation noun clusters (types, enums, handlers, constants).
2. Map each cluster to business vocabulary used by non-technical stakeholders.
3. Prefer test/doc vocabulary when available.
4. Apply the same mapping consistently across all scenarios in a candidate.

Never draft scenario text directly from implementation identifiers.

## Output Schemas

### `feature_candidates`

```yaml
feature_candidates:
  - name: string
    kebab_name: string
    type: cluster | invariant
    entry_points:
      - file: string
        line: number
        identifier: string
        trigger: string
    behavioral_summary: string
    invariant_signals:                # required when type = invariant
      - signal: shared_state | weighted_selection | accumulator | implicit_sequence
        evidence: string              # file:line
        plain_english: string
    entanglement_warnings:
      - string
    test_coverage: covered | partial | none | unknown
    confidence: high | medium | low
```

### `gap_report`

```yaml
gap_report:
  - feature_candidate: string         # kebab_name
    gaps:
      - type: undefined_failure | partial_implementation | implicit_precondition | missing_boundary | entangled_behavior | side_effect_unspecified | unreachable_code
        description: string
        location: string              # file:line or symbol
        recommendation: address now | defer to trap analysis | accept undefined
```

### `draft_specs`

```yaml
draft_specs:
  - feature_candidate: string
    candidate_type: cluster | invariant
    feature_title: string
    actor: string
    intent: string
    value: string
    background_steps:
      - keyword: Given | And
        text: string
    scenarios:
      - name: string
        type: scenario | scenario_outline | multi_step_invariant
        steps:
          - keyword: Given | When | Then | And | But
            text: string
        examples: string | null
        source_ref: string
        confidence: high | medium | low
```

## Approval Interaction Protocol

Execution ownership:
- Fuhyō executes analysis
- Ōshō handles user-visible interaction
- Kakugyō orchestrates sequencing

### 1) Boundary approval gate

Present candidate boundaries (clusters and invariants separately) and require explicit action:

`approve-all` / `approve <n>` / `reject <n>` / `merge <n> <n>` / `split <n>` / `rename <n> <new-name>`

Silence is not consent. No drafting proceeds without explicit boundary approval.

### 2) Gap report gate (mandatory, non-skippable)

For each approved candidate, present gaps before scenarios:

`acknowledge` / `address <n> now` / `defer-all`

Gap reporting is mandatory and MUST NOT be suppressed.

### 3) Per-scenario approval gate

Present one scenario at a time with `source_ref` and confidence.
Require explicit approval for each scenario.
Silence is not consent.

### 4) Next-step routing

After approval-ready draft specs are returned, Kakugyō may route to `/dev-bdd-gherkin` for refinement or `/flow-start-pipeline` for staged delivery.

## Hard Rules (Expanded)

1. NEVER mutate source files.
2. NEVER invent behavior not grounded in evidence.
3. NEVER hide uncertainty; mark confidence explicitly.
4. EVERY scenario MUST carry `source_ref`.
5. NEVER draft a scenario for gap behavior.
6. MUST build and apply a vocabulary map before drafting any scenario.
7. NEVER suppress the gap report.
8. MUST keep clusters and invariants distinct during boundary approval.
9. MUST include all detected low-confidence candidates; do not silently discard.
10. MUST keep scenario language domain-facing and implementation-agnostic.
11. NEVER claim complete behavioral coverage from partial evidence.
12. NEVER bypass explicit approval gates (boundary, gap disposition, per-scenario).
