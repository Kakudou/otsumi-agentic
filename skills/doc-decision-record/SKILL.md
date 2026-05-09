---
name: doc-decision-record
description: "Create ADR/TDR decision records or explicit no-record reasoning based on actual implementation and workflow evidence."
---

# Doc Decision Record

Record architectural or technical decisions, or document why no record is needed.

## Usage

`/doc-decision-record {feature-name}`

## Hard Rules

- NEVER silently skip decision evaluation.
- NEVER create a fake ADR for a non-decision.
- NEVER invent context not present in artifacts.
- MUST return either records created or no-record reasoning.
- Fuhyō executes this skill atomically; Hisha writes broad narrative docs outside this skill's scope.

## Decision Types

- **ADR**: architecture decision
- **TDR**: technical decision

## Steps

1. Inspect behavior contract, implementation artifacts, tests, and review notes.
2. Detect decisions involving:
   - architecture
   - storage
   - integration
   - dependency
   - API/interface
   - security or operational tradeoff
3. Write records under `docs/decisions/` when required.
4. If no record is needed, return explicit reasoning.

## Trigger Checklist

Create a decision record when this feature **introduced or modified** any of the following:

- [ ] Architecture changed (new layer, boundary, or structural pattern introduced)
- [ ] Interface added or modified (public API, module contract, or event schema)
- [ ] New dependency introduced (library, service, or internal module)
- [ ] Persistence strategy affected (how/where data is stored or fetched)
- [ ] Security model modified (authN/authZ, access rules, or data exposure posture)
- [ ] Performance strategy changed (caching, batching, async orchestration)
- [ ] Cross-cutting convention established (naming, logging, error, model conventions)

Disambiguation:
- **Introduced or modified** means the feature changed the pattern itself, not merely reused an existing one.
- **Established** means this feature sets a convention future features are expected to follow.

If nothing above triggers, set `decision_required: false` and provide non-empty `no_record_reasoning`.

## ADR vs TDR Decision Tree

Use these questions in order:

1. Does the choice alter system structure, boundaries, or long-term architecture? → **ADR**
2. If not, does it define implementation strategy inside existing architecture? → **TDR**
3. If neither, is it a local/no-propagation choice? → no record file, only explicit reasoning

When uncertain between ADR and TDR: default to **TDR**. Promote to ADR only if other features must comply.

### Classification Examples

| Example decision | Classification |
|---|---|
| Introduce a new application layer boundary | ADR |
| Adopt pub-sub between modules instead of direct calls | ADR |
| Define a shared event schema as a contract | ADR |
| Standardize service authentication strategy | ADR |
| Choose database persistence over in-memory storage | ADR |
| Pick one library over another for parsing | TDR |
| Use sync vs async execution within existing architecture | TDR |
| Choose Pydantic model vs dataclass in this implementation | TDR |
| Standardize technical error message formatting | TDR |
| Tune batch size for a processing operation | TDR |

## ADR Template

Path: `docs/decisions/ADR-XXXX-<kebab-title>.md`

```markdown
---
type: adr
status: accepted
date: YYYY-MM-DD
number: XXXX
tags: [decision, architecture, <domain-tag>]
links:
  - "[[<feature-name>]]"
---

# ADR-XXXX: <Title>

**Date**: YYYY-MM-DD
**Status**: Accepted
**Feature**: [[<feature-name>]]

## Context

<Situation that forced this architectural decision: prior state, constraints, and pressure. Keep the decision itself out of this section.>

## Decision

<Concrete architectural commitment. Start with "We will..." or "This system will...">

## Consequences

### Enables
<What this makes easier or possible.>

### Constrains
<What this limits, forbids, or increases in cost.>

### Risks
<Failure modes and assumptions that must continue to hold.>

## Alternatives Considered

| Alternative | Why Rejected |
|---|---|
| Option A | reason |
| Option B | reason |

## Compliance Guidelines

<Norms future features must follow because of this ADR. Use imperative language. If none: "No compliance rules introduced.">

## Related

- Features: [[<feature-name>]]
- Decisions: [[ADR-XXXX]] [[TDR-XXXX]]
```

## TDR Template

Path: `docs/decisions/TDR-XXXX-<kebab-title>.md`

```markdown
---
type: tdr
status: accepted
date: YYYY-MM-DD
number: XXXX
tags: [decision, technical, <domain-tag>]
links:
  - "[[<feature-name>]]"
---

# TDR-XXXX: <Title>

**Date**: YYYY-MM-DD
**Status**: Accepted
**Feature**: [[<feature-name>]]

## Context

<Implementation-level problem and constraints that required a technical choice.>

## Decision

<Exact technical choice and where it applies. Start with "We will use..." or "This feature will...">

## Rationale

<Why this option was selected, preferably with observable or measurable reasoning.>

## Trade-offs

| Accepted | Rejected |
|---|---|
| <what we gain> | <what we give up> |

## Alternatives

| Alternative | Why Rejected |
|---|---|
| Option A | reason |

## Scope

<Feature-local only, or broader applicability?>

## Related

- Features: [[<feature-name>]]
- Decisions: [[ADR-XXXX]] [[TDR-XXXX]]
```

## Supersession Handling

If a new decision replaces an existing record:

1. Update old record frontmatter status to `superseded by [[ADR-XXXX]]` or `superseded by [[TDR-XXXX]]`.
2. Add `supersedes: [[ADR-XXXX]]` or `supersedes: [[TDR-XXXX]]` to the new record frontmatter.
3. State the supersession relationship in the new record's Context section.

## Result Schema

Return this structure to the caller:

```yaml
decision_required: true | false
records_written:
  - type: adr | tdr
    id: ADR-XXXX | TDR-XXXX
    path: docs/decisions/<file>.md
    trigger: <trigger checklist item>
records_updated:
  - id: ADR-XXXX | TDR-XXXX
    change: <what was updated, e.g. superseded status>
no_record_reasoning: "<required non-empty string when decision_required is false>"
```

Rules:
- `records_updated` must be present when existing records are changed (including supersession updates).
- `no_record_reasoning` is mandatory when `decision_required: false`; empty string is invalid.
