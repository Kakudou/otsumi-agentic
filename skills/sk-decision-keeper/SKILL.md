---
name: sk-decision-keeper
description: Evaluates whether a feature introduced decisions requiring an ADR or TDR. Standalone, no pipeline state required. Use with a feature name, the feature file, and optionally src/ files to analyse. Explicitly states when no record is needed.
---

# decision-keeper

One job: ensure no architectural or technical decision goes unrecorded.

Silent decisions are technical debt with a fuse.
Evaluate what was built. Write the record, or explicitly state why none is needed.
Never silently skip.

---

## Inputs

Provided by whoever calls this skill (agent or direct invocation):

| Input | Required | Description |
|-------|----------|-------------|
| `feature_name` | yes | kebab-case name of the feature |
| `feature_file` | yes | path to the `.feature` file |
| `language_id` | no | language family context (e.g. `python`), passed by the agent when in pipeline context, omit for standalone use |
| `implementation_files` | no | list of `src/` files built for this feature (provide when available) |
| `step_files` | no | list of step definition files (provide when available) |
| `existing_decisions_dir` | no | path to `docs/decisions/` for conflict/supersession check (defaults to `docs/decisions/`) |

---

## Outputs

| Output | Description |
|--------|-------------|
| `docs/decisions/ADR-XXXX-<title>.md` and/or `TDR-XXXX-<title>.md` | written if triggered |
| `stage-06-result` | decision summary, returned to caller, not written to disk by this skill |

The caller (agent or Otsumi) is responsible for writing stage output JSONs and logging.

---

## Activation Steps

1. Confirm `feature_name` and `feature_file` are provided by the caller.

2. Read `<feature_file>`: understand what behavior was specified.

3. Read `<existing_decisions_dir>` (default: `docs/decisions/`): check existing records for conflicts, supersession, and next available number.

4. If `implementation_files` were provided: read them to understand what was actually built.

5. If `step_files` were provided: read them for additional implementation context.

6. Run the Trigger Checklist (see below) against all available context.

7. If one or more triggers fire: write ADR/TDR records (see Templates below).

8. If no trigger fires: document the reasoning explicitly: do not leave `no_record_reasoning` empty.

9. Return `stage-06-result` to the caller.

The caller (agent or Otsumi) is responsible for writing `stage-06-output.json`, running the commands `/atomic-log`, and managing pipeline state.

---

## Trigger Checklist

A record is required if the feature **introduced or modified** any of these:

- [ ] Architecture changed: new layer, boundary, or structural pattern introduced (not just used)
- [ ] Interface added or modified: public API, module contract, event schema
- [ ] New dependency introduced: library, external service, or internal module
- [ ] Persistence strategy affected: how or where data is stored or retrieved
- [ ] Security model modified: auth, access control, data exposure rules
- [ ] Performance strategy changed: caching, batching, async patterns
- [ ] Cross-cutting convention established: naming, error handling, logging, model type

**"Introduced or modified"**: using an existing pattern without changing it does NOT trigger a record.
**"Established"**: if this feature sets a convention others will follow, it triggers a record even if the change seems small.

If none apply: set `decision_required: false` and write the reasoning explicitly in `no_record_reasoning`.

---

## ADR vs TDR - Classification

Three questions in order:

1. Does it affect system structure, boundaries, or long-term design? → **ADR**
2. Does it affect how something is implemented within existing architecture? → **TDR**
3. Is it a local inline choice with no cross-feature impact? → document in `no_record_reasoning` only, no file needed

| ADR (architecture) | TDR (technical) |
|--------------------|-----------------|
| New layer or module boundary | Choice of library or data structure |
| Pub/sub vs direct call pattern | Sync vs async within existing pattern |
| Event schema contract | Pydantic vs dataclass for a model |
| Auth strategy | Error message format |
| Database vs in-memory persistence | Batch size for a processing loop |

When uncertain between ADR and TDR: default to TDR. Promote to ADR only if other features must comply.

---

## ADR Template

`docs/decisions/ADR-XXXX-<kebab-title>.md`

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

<What situation forced this decision. The problem, the constraint, the pressure.
Include: what existed before, what changed, what trade-off had to be made.
Do NOT include the decision itself here.>

## Decision

<Exactly what was decided. Specific. No vague language.
Start with: "We will..." or "This project will...">

## Consequences

### Enables
<What this decision makes possible or easier.>

### Constrains
<What this decision makes harder, more expensive, or forbidden.>

### Risks
<What could go wrong. What assumptions must hold for this to remain valid.>

## Alternatives Considered

| Alternative | Why Rejected |
|-------------|--------------|
| Option A    | reason       |
| Option B    | reason       |

## Compliance Guidelines

<Rules this creates for future features. Stated as imperatives.
Example: "All domain models MUST use Pydantic BaseModel. @dataclass is forbidden."
If no compliance rules: write "No compliance rules introduced.">

## Related

- Features: [[<feature-name>]]
- Decisions: [[ADR-XXXX]] [[TDR-XXXX]]
```

---

## TDR Template

`docs/decisions/TDR-XXXX-<kebab-title>.md`

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

<What implementation problem required a technical choice.
Include: what options were visible, what constraints existed.>

## Decision

<Exactly what was chosen and how it is applied.
Start with: "We will use..." or "This feature will...">

## Rationale

<Why this option over the alternatives. Measurable or observable reasoning preferred.>

## Trade-offs

| Accepted | Rejected |
|----------|----------|
| <what we gain> | <what we give up> |

## Alternatives Considered

| Alternative | Why Rejected |
|-------------|--------------|
| Option A    | reason       |

## Scope

<Is this decision scoped to this feature, or does it apply more broadly?>

## Related

- Features: [[<feature-name>]]
- Decisions: [[ADR-XXXX]] [[TDR-XXXX]]
```

---

## Supersession Handling

If this decision replaces an existing ADR/TDR:

1. Update the old record's frontmatter: `status: superseded by [[ADR-XXXX]]`
2. Add `supersedes: [[ADR-XXXX]]` to the new record's frontmatter
3. Reference the supersession in the new record's Context section

---

## Result Schema

Return `stage-06-result` to the caller:
```
decision_required: true | false
records_written: [{ type, id, path, trigger }]
records_updated: [{ id, change }]
no_record_reasoning: "<required and non-empty if decision_required: false>"
```

`no_record_reasoning` is **required** when `decision_required: false`. An empty string is not acceptable.

The caller writes the stage output JSON and runs `/atomic-log` if in pipeline context.

---

## Handoff

Return `stage-06-result` to the caller.

Report:
> Stage-6 complete for `<feature-name>`.
> Decision required: `<true|false>`. Records written: `<n>`.
