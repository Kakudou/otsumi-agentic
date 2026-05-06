---
name: retro-feature
description: Legacy feature recovery. Reads existing source code, identifies the distinct behaviors it implements, and produces approved Gherkin specs, so legacy code can feed into the standard pipeline and earn its quality gates retroactively.
---

# Retro Feature

You are receiving a `/retro-feature` command.

Legacy code has behavior that was never spec'd. This command reads source as evidence, surfaces what the code does — not what it was intended to do — and translates that into Gherkin a human can read, challenge, and approve.

The output describes what the code does today, not what it should do. Undefined behavior goes into the gap report, not into a scenario. Entangled concerns are flagged. You decide which behaviors are intentional before anything gets written.

Scope lock: produces `.feature` files and a gap report only. NEVER touches source code, generates tests, or creates pipeline state. Stack-agnostic — no language or pipeline context inference.

## Usage

- `/retro-feature <path>` — scan a specific file or directory
- `/retro-feature <path> --tests <tests-path>` — scan source with supplemental test evidence

## Hard Rules

- NEVER use implementation vocabulary in any scenario step: no `class`, `function`, `endpoint`, `model`, `schema`, `database`, `API`, `object`, `method`, `service`, `handler`, `controller`, `repository`.
- NEVER approve a scenario on behalf of User. Every scenario requires explicit User approval.
- NEVER suppress the gap report. Gaps MUST be surfaced before scenarios are presented.
- NEVER overwrite an existing `features/<feature-name>.feature`.
- NEVER create `.otsumi/` pipeline state. This command ends at approved `.feature` files.
- NEVER proceed past boundary approval without explicit User response on every candidate.
- NEVER claim the recovered Gherkin is complete. It describes observable behavior only — hidden behaviors, emergent interactions, and timing-dependent paths may not be captured.
- NEVER touch the source code.
- NEVER draft a scenario for a gap behavior. Gaps belong in the gap report.
- NEVER invent behaviors not present in the source. If not implemented, it goes in the gap report.
- Every scenario MUST carry a `source_ref`. No anonymous scenarios.
- MUST build the vocabulary map before drafting any scenario.
- Low-confidence candidates MUST be flagged explicitly. Never silently upgrade confidence.

---

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `source_path` | yes | absolute path to source file or directory to analyse |
| `tests_path` | no | path to existing test files, supplemental behavioral evidence, not required |

---

## Outputs

| Output | Description |
|--------|-------------|
| `feature_candidates` | detected feature boundaries with evidence, confidence, and warnings |
| `gap_report` | per-candidate undefined, inconsistent, or missing behaviors |
| `draft_specs` | per-candidate draft Gherkin scenarios with source references |

---

## Phase 1: Source Scan and Feature Boundary Detection

### Goal

Cluster the code by the behaviors it produces for its caller or user, not by the file structure it happens to live in.

Legacy code is organized by technical concern, layer, or accident of growth. The job here is to surface the distinct delivery units hidden inside that structure. Each delivery unit becomes a feature candidate.

### What to read for entry points

Entry points are where behavior can be triggered from outside the module. Depending on the language and style of the code:

- public functions and methods at module level
- class methods that form a public API surface
- route handlers, decorators, or annotations that bind an HTTP path to logic
- CLI argument parsers, subcommands, or entry script entry points
- event listeners, message consumers, queue handlers, signal handlers
- exported constructors and their primary interaction methods

For each entry point, identify:
- what triggers this behavior (what the caller must do)
- what inputs it consumes (parameters, request body, environment)
- what output or side effect it produces on the happy path
- what explicit error or rejection paths exist in the code

### What makes a feature candidate

A feature candidate is a cluster of entry points that collectively deliver one coherent, nameable capability to a user or caller:

- They share a subject domain (e.g., "invoice lifecycle", "user session", "payment processing")
- They form a natural sequence or lifecycle (create → validate → submit → notify)
- They operate on the same data or external surface (same table, same external service, same file)
- Removing one from the cluster would leave the remaining behaviors incomplete or meaningless

The name of the candidate MUST be derivable in plain business English from what the cluster does, not from what files it lives in.

### Entanglement detection

Flag a candidate as entangled when a single entry point mixes two or more distinct behavioral concerns that do not naturally belong together:

- a function that handles both authentication and authorization decisions
- a method that does validation, persistence, and notification in one pass
- a route handler that mixes business logic with caching and logging

Entanglement warnings do not block the candidate from being proposed. They inform the User that the boundary may need to be split before Gherkin can be written cleanly.

Rate candidate confidence:

- `high`, clear entry points, coherent subject domain, minimal entanglement, behavioral summary is unambiguous
- `medium`, some entanglement or vocabulary ambiguity, summary required reasonable inference
- `low`, significant entanglement, multiple plausible interpretations, strongly recommend split or manual boundary redefinition before drafting

### Cross-call state analysis

After the initial entry point clustering, perform a second pass to detect behavioral invariants, features that cannot be seen by looking at any single entry point in isolation.

This pass is architecture-agnostic. It looks for four universal signals regardless of directory layout, framework, or naming conventions.

**Signal 1: Shared persistent state**

Identify every piece of state that survives a single call, files read or written, database tables touched, caches updated, in-memory structures passed between calls via a shared store. Then look for overlaps: entry points that write to state that another entry point reads.

An overlap is a coupling. A coupling between two different user-facing operations means the output of one affects the behavior of the other over time. That is a behavioral invariant candidate, name it from the user's perspective, not from the state object's name.

Ask for each persistent object found: *who writes it, who reads it, and does reading it change what that reader produces?* If yes, that is a feature. If the answer requires tracing through multiple calls, that is an even stronger signal.

**Signal 2: Probabilistic or weighted selection**

Locate any call to random selection, weighted choice, scoring functions, or probability distributions anywhere in the call chain of any entry point, not just at the top level. These are never present in pure CRUD code. Their presence means the output of some operation varies based on something other than the immediate inputs alone.

Ask: *what does this selection depend on? Is that dependency updated by another entry point?* If yes, the selection and the updater together form one behavioral invariant: the system learns or adapts.

**Signal 3: Accumulators**

Locate any value that is incremented, appended to, or updated (not replaced) across calls, counters, running totals, transition matrices, ratio trackers, history logs, decay values. An accumulator that influences downstream behavior is a learning or adaptation mechanism. Name the mechanism from the user's perspective: *"plans improve as more work is logged"*, not *"transitions dict is updated"*.

**Signal 4: Implicit sequencing**

Look for operations where the documented or intended user workflow requires a specific ordering of entry points, not enforced by the code, but implied by what the code produces. If entry point B only produces meaningful output after entry point A has been called at least once, that sequencing is a behavioral contract that belongs in Gherkin as a `Background` or `Given` precondition.

### Behavioral invariant candidates

Each signal that fires produces a new candidate of type `invariant`, added to `feature_candidates` alongside the entry-point clusters. Invariant candidates:

- have no single entry point, their `entry_points` field lists all the operations involved
- have a `type: invariant` field
- must include a plain-English description of the behavior observable to the user over multiple interactions
- must cite the specific code location (file and line) of each signal that produced them, the state object, the weighted selection call, the accumulator, or the implicit sequence dependency

NEVER invent invariants. Every invariant candidate MUST be grounded in code evidence cited by file and line. If the signal is present but the downstream effect cannot be determined from static analysis alone, flag it as `confidence: low` with an explicit note about what cannot be determined.

### Supplemental test signal

If `tests_path` is provided, read the test files as a second signal:

- Test function names, describe blocks, and it-blocks often carry better domain vocabulary than source code
- Test setup reveals real-world preconditions that the source does not explicitly model or validate
- Tested edge cases are strong evidence of intentional behavior
- Entry points with zero test coverage are elevated as higher-confidence gap candidates

NEVER treat test descriptions as authoritative spec. Tests can be wrong, incomplete, or implementation-level. Weight them as supporting evidence, not ground truth.

### Output: `feature_candidates`

```
feature_candidates:
  - name: <plain English feature name>
    kebab_name: <kebab-case derived from name>
    type: cluster | invariant
    entry_points:
      - file: <relative path>
        line: <number>
        identifier: <function, method, or route name>
        trigger: <what initiates this behavior>
    behavioral_summary: <one or two sentences: what this feature does for its user or caller>
    invariant_signals:                          # only present when type = invariant
      - signal: shared_state | weighted_selection | accumulator | implicit_sequence
        evidence: <file:line, the specific code location that produced this signal>
        plain_english: <what this signal means in observable user-facing terms>
    entanglement_warnings:
      - <plain English: what concern is mixed in and why it may warrant splitting>
    test_coverage: covered | partial | none | unknown
    confidence: high | medium | low
```

Return all candidates including low-confidence ones. NEVER suppress — flag and let the User decide.

---

## Phase 2: Gap Detection

### Goal

Surface what the code does not make clear before scenarios are drafted. A scenario written over undefined behavior is a false spec.

### Gap types

| Type | What it means |
|------|---------------|
| `undefined_failure` | An entry point has no error handling for a known failure condition, behavior on failure is not implemented and cannot be spec'd |
| `partial_implementation` | A code path is incomplete: stub, TODO, bare pass, or logic that never reaches a conclusion |
| `implicit_precondition` | The code assumes a precondition that is never validated, behavior when that precondition is violated is unknown |
| `missing_boundary` | No handling for edge inputs: null, empty collection, zero, negative, maximum, boundary behavior is unspecified |
| `entangled_behavior` | Entry point mixes concerns and the boundary between them cannot be resolved from source alone |
| `side_effect_unspecified` | Function writes to DB, sends a message, or mutates external state but has no handling for side effect failure |
| `unreachable_code` | A code path exists that cannot be triggered by any observable input, present in source but cannot be spec'd |

For each gap, record:
- `type`, from the table above
- `description`, plain English: what is unknown or missing
- `location`, file path and line number, or function/method name
- `recommendation`:
  - `address now`, critical unknown; scenario cannot be written without clarification
  - `defer to trap analysis`, a boundary or edge case well-suited for Stage-2 trap analysis when the pipeline runs
  - `accept undefined`, behavior is intentionally unspecified (common for low-risk internal paths)

### Output: `gap_report`

```
gap_report:
  - feature_candidate: <kebab-name>
    gaps:
      - type: <gap-type>
        description: <plain English>
        location: <file:line or function name>
        recommendation: address now | defer to trap analysis | accept undefined
```

---

## Phase 3: Gherkin Drafting

### Goal

Translate observable behaviors into domain-language scenarios that a non-technical stakeholder can read, challenge, and approve.

The code uses implementation vocabulary. Gherkin uses domain vocabulary. The translation is not mechanical, it requires reasoning about intent, not just structure.

### Domain vocabulary recovery

Before drafting any step, build a vocabulary map for each feature candidate:

1. Read the code for noun clusters, what are the real-world objects being operated on?
   - `UserRecord` → `registered user`
   - `InvoiceEntity` → `invoice`
   - `process_document()` → `document is submitted for processing`
   - `PaymentStatusCode.DECLINED` → `payment is declined`

2. Read docstrings, inline comments, and any README fragments for vocabulary already written by the original developer

3. If `tests_path` was provided, read test names and fixture names, they often contain the best domain language in the codebase

4. When choosing between vocabulary options: prefer the term a business stakeholder would use in a conversation about this system, not the term the developer used in a variable name

Apply the vocabulary map consistently across all scenarios for the same feature. If two entry points operate on the same domain object, use the same name for it in both scenarios.

### What constitutes a valid scenario from legacy source

A valid scenario must be:

- **Observable**, the outcome can be verified from outside the implementation, without inspecting internal state
- **Intentional**, the behavior is a deliberate capability present in the code, not a side effect of an unrelated structure
- **Domain-language**, all steps use business vocabulary, zero implementation terms
- **Coherent**, tests one path through one entry point, or one complete observable arc of a behavioral invariant

For cluster candidates, draft per entry point:

1. **Happy path**, the primary successful flow with valid inputs and the expected outcome
2. **Explicit failure paths**, rejection or error behaviors that are explicitly implemented in the code (not gaps, only handled paths)
3. **Explicit boundary behaviors**, only when the code explicitly handles them; NEVER invent handling that is not there

For invariant candidates, draft per signal:

1. **The learning or adaptation scenario**, a multi-step scenario that sets up prior state, performs the operation that updates it, then performs the operation that is influenced by it, and asserts the observable difference

   Use this structure:
   ```
   Background:
     Given <prior state that must exist for the invariant to manifest>

   Scenario: <name the observable change in behavior>
     Given <starting condition for this specific path>
     When  <first interaction, the one that updates state>
     And   <second interaction, the one influenced by prior state>
     Then  <the observable difference compared to a system with no history>
   ```

   The `Then` step must describe something the user can see or measure, a different plan output, a different ordering, a preemption that did or did not occur. It must not reference internal state, matrix values, or probability scores.

2. **The boundary condition**, what happens at the edges of the invariant's influence: first interaction ever (no prior state), state that conflicts with current input, reset that clears accumulated state

   These are often `low` confidence and MUST be flagged as such; they require the system to have reached a specific accumulated state before the behavior manifests, which static analysis cannot always fully verify.

NEVER draft scenarios for:
- Gap behaviors (live in the gap report, not in Gherkin)
- Internal implementation details (object instantiation, module loading, internal function calls)
- Behaviors that can only be verified by inspecting internal state
- Behaviors from unreachable code paths
- Invariant effects that cannot be observed without reading the internal state object directly

### Source references

Every drafted scenario carries a `source_ref`, the file path and line number (or function name) in the source that produced it.

Source refs are for the approval conversation. They answer "where did this come from?" when User challenges a scenario.

### Background

If multiple scenarios for the same candidate share identical setup steps (≥ 2 scenarios, identical preconditions), draft a `Background` block. Same rule as greenfield Gherkin: NEVER use `Background` for steps that only one scenario needs.

### Scenario Outlines

If the code handles multiple input variations through the same path, different document types, different status codes, different role levels, use `Scenario Outline` + `Examples`. Extract the actual values from the source: constants, enum members, switch cases, if-elif chains, lookup tables.

### Scenario confidence

Rate each scenario:
- `high`, behavior is fully explicit in source, domain vocabulary is unambiguous, no gaps involved
- `medium`, behavior required minor inference, vocabulary required judgment
- `low`, behavior is partially inferred or relies on implicit preconditions; User MUST scrutinize carefully

### Output: `draft_specs`

```
draft_specs:
  - feature_candidate: <kebab-name>
    candidate_type: cluster | invariant
    feature_title: <human-readable title for the Feature: line>
    actor: <who uses this feature, derived from entry point callers or domain context>
    intent: <what they want to achieve>
    value: <why it matters, inferred from behavioral summary>
    background_steps:
      - keyword: Given | And
        text: <shared precondition step>
    scenarios:
      - name: <scenario name>
        type: scenario | scenario_outline | multi_step_invariant
        steps:
          - keyword: Given | When | Then | And | But
            text: <step in domain language>
        examples: <pipe-delimited table or null>
        source_ref: <file:line, for cluster scenarios> | <signal evidence refs, for invariant scenarios>
        confidence: high | medium | low
```

---

## Activation Steps

### 1. Validate inputs

- `<path>` must exist and must be a readable source file or directory. If it does not exist: halt and report.
- If `--tests <tests-path>` is provided, confirm the path exists. Absence does not block execution, it is supplemental only.
- No other flags. No language or pipeline context inference.

### 2. Execute Phases 1–3

Run Phase 1 (Source Scan and Feature Boundary Detection), Phase 2 (Gap Detection), and Phase 3 (Gherkin Drafting) as defined above.

### 3. Feature boundary approval

Present the `feature_candidates` list to User. Display cluster candidates and invariant candidates in separate groups:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Feature · Candidate Boundaries · <path>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Direct features (<n>):

  <n>. <feature-candidate-name>
       Evidence: <files or entry points>
       Confidence: high | medium | low
       Concerns: <entanglement warnings, scope notes, or "none">

Behavioral invariants (<n>):
  These are cross-cutting behaviors observable only across multiple interactions.
  Each was detected from code signals, not entry points.

  <n>. <feature-candidate-name>
       Signals: <plain-English summary of what was detected>
       Evidence: <file:line citations>
       Confidence: high | medium | low
       Concerns: <notes or "none">

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: approve-all / approve <n> / reject <n> / merge <n> <n> / split <n> / rename <n> <new-name>
```

Wait for explicit User response. Silence is not consent.

- `approve-all`, accept all candidates as proposed
- `approve <n>`, accept candidate n
- `reject <n>`, discard candidate n, no Gherkin written for it
- `merge <n> <n>`, combine two candidates into one feature
- `split <n>`, flag a candidate for boundary redefinition; wait for User's revised description, then re-enter approval
- `rename <n> <new-name>`, override the proposed feature name

Only approved, non-split candidates proceed to Gherkin drafting. Rejected candidates are noted in the completion report.

### 4. Present gap report before scenario approval

Before presenting any draft Gherkin, surface the gap report for each approved candidate:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Feature · Gap Report · <feature-candidate-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Gaps found: <n>

  • [<gap-type>] <what is undefined or missing>
    Location: <file:line or function name>
    Recommendation: address now / defer to trap analysis / accept undefined

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: acknowledge / address <n> now / defer-all
```

- `acknowledge`, noted, proceed to scenario drafting with gaps flagged
- `address <n> now`, User clarifies behavior for gap n; wait, incorporate, then continue
- `defer-all`, proceed; gaps will reappear naturally during trap analysis when the pipeline runs Stage-2

MANDATORY: NEVER skip this step. Suppressing the gap report produces false confidence.

### 5. Per-feature Gherkin approval and `.feature` file creation

For each approved feature candidate, in order:

**a.** Derive a kebab-case feature name from the approved candidate name.

**b.** Check for an existing `.feature` file. If `features/<feature-name>.feature` already exists: halt for this feature and report. NEVER overwrite. User must resolve the conflict explicitly.

**c.** Present draft Gherkin scenarios one at a time:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Feature · Scenario <n>/<total> · <feature-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<scenario name>

  Given <precondition>
  When  <action>
  Then  <outcome>

Source: <file:line or function that produced this scenario>
Confidence: high | medium | low
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Actions: approve / reject / edit / split / merge
```

Wait for explicit approval on each scenario. Silence is not consent.

**d.** After all scenarios are approved, present the full draft spec for final confirmation before writing.

**e.** Write `features/<feature-name>.feature`.
**f.** Keep the `Source: <file:line or function that produced this scenario>` comment in the file, with the associated scenario, this will help trace back the source of each scenario.

### 6. Report completion

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Feature · Complete · <path>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Features recovered: <n>
  • <feature-name>, <n> scenarios, <n> gaps (<disposition summary>)
  ...

Features skipped: <n>
  • <feature-name>, reason: rejected / conflict / split-pending

Ready to pipeline:
  For each recovered feature, start a pipeline with:
  /start-pipeline --mode assisted --lang <your-lang> <feature description>

  Or run trap analysis on the spec first:
  Ask the Otsumi to run gherkin-keeper on features/<feature-name>.feature

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

No pipeline state is created by this command. The `.feature` files are the output. The developer chooses the language and starts the pipeline explicitly.
