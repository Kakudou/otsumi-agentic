---
name: core-plan-lint
description: "Statically validate a Kakugyō plan against the atomic-or-swarm mandate, atomicity smell tests, dependency semantics, parallel_group safety, blocker vocabulary, and shogi-roster discipline. Returns pass/fail with structured violations and a suggested decomposition for any malformed step."
---

# Core Plan Lint

Lint a Kakugyō orchestration plan before it is dispatched. This skill is the machine-checkable companion to the Hard Rules in `agents/kakugyo.md` and `agents/fuhyo.md` — every check here corresponds to a written contract clause. A passing lint does NOT certify that a plan is *good*, only that it is *well-formed*. Soundness still belongs to Kakugyō; lint just refuses obvious malformedness before it ships.

## Usage

- `/core-plan-lint --plan {plan.json} --user-intent "{raw user request}"` — full lint.
- `/core-plan-lint --plan {plan.json} --user-intent "{...}" --strict` — fail on warnings as well as errors.
- `/core-plan-lint --plan {plan.json} --user-intent "{...}" --skip {check_id,...}` — explicitly skip named checks (each skip MUST be justified in the calling step's `assumptions[]`).

`--plan` accepts a path to a JSON file or an inline JSON object literal.

## Hard Rules

- MUST treat the lint as **structural**, NEVER **semantic**: this skill does not judge whether a plan correctly solves the user's problem, only whether it conforms to the contract shape.
- MUST emit a deterministic verdict: pass / fail-on-error / fail-on-warning-in-strict-mode. NEVER soften an error into a warning to let a plan through.
- MUST list EVERY violation found, NEVER short-circuit on the first one. Kakugyō needs the full set to replan in one round.
- MUST identify the offending step by `step_id`. NEVER attribute a violation to the plan as a whole when a specific step is responsible.
- MUST suggest a concrete decomposition for `atomic-or-swarm` and `coarse-bundling` violations, sourced from the canonical fan-out recipes in `kakugyo.md`.
- MUST surface unknown / off-board agent names (`general-purpose`, bare `Task`, `claude-code`, `agent`, `subagent`, anything not in the shogi roster) as `error` severity. NEVER as `warning`.
- NEVER mutate the plan. This skill is read-only.
- NEVER call out to external tools or skills. The lint runs against the supplied JSON only.

## Input Contract

```json
{
  "plan": {
    "plan_id": "PLAN-001",
    "response_type": "initial | continue | expand | parallelize | replan | close",
    "summary": "",
    "state_root": null,
    "macro_plan": [],
    "next_steps": [],
    "challenge_policy": {},
    "validation_policy": {}
  },
  "user_intent": "the raw / refined user request, used for the multi-unit-implies-swarm check",
  "options": {
    "strict": false,
    "skip_checks": []
  }
}
```

## Output Contract

```json
{
  "verdict": "pass | fail",
  "errors": [
    {
      "check_id": "FUHYO_ATOMICITY_PROOF_MISSING",
      "step_id": "S6",
      "severity": "error",
      "message": "Step S6 has agent: \"fuhyo\" but atomicity_proof is null.",
      "suggested_fix": "Populate atomicity_proof with 5 statements per agents/fuhyo.md, OR fan out the step into a swarm if multi-unit."
    }
  ],
  "warnings": [],
  "stats": {
    "steps_total": 0,
    "fuhyo_steps": 0,
    "swarms": 0,
    "fanout_groups": {}
  },
  "passed_checks": []
}
```

## Steps

### 1. Parse and validate input

1. Load `plan` (file path or inline JSON).
2. Confirm `user_intent` is a non-empty string.
3. If parsing fails, return `verdict: "fail"` with `errors[]` containing a single `check_id: "INPUT_MALFORMED"`.

### 2. Run the check battery

Run every check below. Collect violations as you go — NEVER short-circuit. Each check has a stable `check_id`, a severity, and a fix recipe.

#### Schema checks

| `check_id` | Severity | Rule |
|---|---|---|
| `PLAN_ID_MISSING` | error | `plan.plan_id` is a non-empty string. |
| `RESPONSE_TYPE_INVALID` | error | `plan.response_type` ∈ {`initial`, `continue`, `expand`, `parallelize`, `replan`, `close`}. |
| `STATE_ROOT_REQUIRED` | error | If any step's `output_contract.write_path` is set, `plan.state_root` is also set. |
| `STEP_ID_DUPLICATE` | error | All `step_id` values across `next_steps[]` and `macro_plan[]` are unique. |
| `STEP_AGENT_MISSING` | error | Every `next_steps[]` entry has a non-empty `agent`. |

#### Roster discipline checks

| `check_id` | Severity | Rule |
|---|---|---|
| `OFFBOARD_AGENT` | error | Every `agent` value ∈ {`kinsho`, `ginsho`, `hisha`, `kyosha`, `fuhyo`, `keima`, `kakugyo`}. Reject `general-purpose`, `Task`, `agent`, `subagent`, `claude-code`, or any other string. |
| `KAKUGYO_AS_EXECUTOR` | error | `agent: "kakugyo"` only appears in plan continuations from Kakugyō itself, never as an executable step in `next_steps[]`. |

#### Fuhyō atomicity checks

For every step where `agent == "fuhyo"`:

| `check_id` | Severity | Rule |
|---|---|---|
| `FUHYO_ATOMICITY_PROOF_MISSING` | error | `atomicity_proof` exists, is an array, and has exactly 5 entries. |
| `FUHYO_ATOMICITY_PROOF_EMPTY_ENTRY` | error | No entry in `atomicity_proof` is empty or whitespace-only. |
| `FUHYO_ATOMICITY_PROOF_GENERIC` | error | Each `atomicity_proof[i]` references a concrete subject of THIS step (the file path, module name, behavior id, or scenario name from `atomic_task` / `input_material`). Across `parallel_group` siblings, the proofs MUST differ — copy-pasted proofs across N siblings is a malformedness signal. |
| `FUHYO_AUTHORIZED_SKILLS_MISSING` | error | `authorized_skills` field MUST be present on every Fuhyō step (may be empty `[]` if the work is a bounded transformation that invokes no skills, but the field MUST exist). |
| `FUHYO_SKILL_IN_TASK_NOT_IN_AUTH` | error | If `atomic_task` (case-insensitive) references a skill name — detected by patterns: `invoke {skill}`, `run {skill}`, `use {skill}`, `skill: {skill}`, `authorized_skills:.*{skill}` in the surrounding plan text, OR any string matching a known skill name pattern (`kb-*`, `dev-*`, `doc-*`, `flow-*`, `core-*`, `git-*`, `design-*`) — that skill name MUST appear in the step's `authorized_skills` array. A skill referenced in the task description but absent from `authorized_skills` will cause a runtime refusal. |

#### Atomic-or-Swarm Mandate (linguistic smell tests on `atomic_task`)

For every step where `agent == "fuhyo"`, scan `atomic_task` (case-insensitive) for the patterns below. Any hit is an `error`.

| `check_id` | Pattern | Suggested fix |
|---|---|---|
| `SMELL_AND_JOINED_VERBS` | Two distinct imperative verbs joined by `and` (e.g. `"write tests and run the suite"`, `"copy fixtures and generate scaffolding"`). | Split into two steps. The verifier action gets its own step with `depends_on_data` on the generator(s). |
| `SMELL_COUNT_PHRASE` | Phrases like `all N`, `each of the N`, `every {plural}`, `the {plural}`, `5 files`, `9 modules`. | Replace the single step with N siblings under one `parallel_group` + a sequential verifier. Use the canonical `multi-file content generation` recipe. |
| `SMELL_GLOB_OUTPUT` | A glob in `output_contract.write_path` (`*`, `**`, `?`, `[`, `]`). | Resolve the glob to N concrete paths up front. Emit N siblings, each with a literal `write_path`. |
| `SMELL_MULTI_FILE_VERB` | Verbs that imply multi-file scope: `scaffold`, `generate the suite`, `implement the layer`, `refactor the module set`, `bootstrap the package`. | Decompose by file / module / behavior. One target per step. |
| `SMELL_GENERATOR_PLUS_VERIFIER` | Same step that produces an artifact AND runs a verifier (test command, lint, format) over it. | Generator(s) and verifier are separate steps. Verifier `depends_on_data` on the generator output(s). |

When ANY smell triggers, also emit a synthesized `SUGGESTED_DECOMPOSITION` violation containing the canonical recipe, the inferred fan-out cardinality (best-effort from the smell match), and a stub `next_steps[]` array the planner can use as a starting point.

#### Multi-unit-implies-swarm check

| `check_id` | Severity | Rule |
|---|---|---|
| `MULTI_UNIT_NO_SWARM` | error | If `user_intent` contains count words (`all`, `each`, `every`, numerals followed by a noun, plurals tied to a deliverable verb) AND `next_steps[]` contains zero `parallel_group` swarms AND zero `flow-start-pipeline` steps, the plan has not decomposed multi-unit work. |
| `SWARM_OF_ONE` | warning | A `parallel_group` with only one member is not a swarm — either remove the group label or add siblings. |

#### Dependency semantics checks

| `check_id` | Severity | Rule |
|---|---|---|
| `DEPENDS_ON_DATA_DANGLING` | error | Every entry in `depends_on_data` is a `step_id` that exists AND whose `output_contract` actually produces consumable data. |
| `BLOCKS_VS_DATA_CONFUSION` | warning | If a step's `depends_on_data` lists a predecessor whose `output_contract` is empty / `null`, the relationship may be `blocks_on_completion` instead. Surface for review. |
| `PARALLEL_GROUP_WRITE_CONFLICT` | error | Within a `parallel_group`, no two steps write to the same `output_contract.write_path` (exact match OR ancestor-of relationship). |
| `PARALLEL_GROUP_DATA_DEPENDENCY` | error | Within a `parallel_group`, no step's `depends_on_data` lists another step in the SAME group. Co-grouped steps must be data-independent. |

#### Plan shape checks

| `check_id` | Severity | Rule |
|---|---|---|
| `NEXT_STEPS_EMPTY_NON_CLOSE` | error | `next_steps[]` is non-empty unless `close_signal: true` or `response_type == "close"`. |
| `BATCH_OVER_SIZED` | warning | `next_steps[]` has more than 20 entries. Recommend escalating to a managed workflow. |
| `TERMINAL_STEP_MATCHES_DELIVERABLE` | warning | At least one terminal step (no successor in `depends_on_data` of others within the plan) plausibly produces a deliverable named in `user_intent`. Heuristic — emits warning only. |

#### Policy checks

| `check_id` | Severity | Rule |
|---|---|---|
| `CHALLENGE_POLICY_MISSING` | warning | `plan.challenge_policy` is present and has `enabled`, `max_rounds`, `absolute_max_rounds`. |
| `VALIDATION_POLICY_MISSING` | warning | `plan.validation_policy` is present when the work has acceptance criteria from Kinshō. |

### 3. Compute the verdict

- `verdict = "fail"` if any `error` violation is recorded.
- In `--strict` mode, `verdict = "fail"` if any `warning` violation is recorded.
- Otherwise `verdict = "pass"`.

### 4. Build the report

Emit the Output Contract JSON. Populate:

- `errors[]`: every error violation with `check_id`, `step_id`, `message`, `suggested_fix`.
- `warnings[]`: same shape, lower severity.
- `stats.steps_total`, `stats.fuhyo_steps`, `stats.swarms` (count of distinct non-null `parallel_group` values), `stats.fanout_groups` (map of `parallel_group → member count`).
- `passed_checks[]`: list of `check_id` strings that ran and produced no violation. Useful for the caller to confirm coverage.

### 5. Suggested-decomposition payload (smell-driven)

For every smell-triggered violation, emit a sibling `SUGGESTED_DECOMPOSITION` entry with shape:

```json
{
  "check_id": "SUGGESTED_DECOMPOSITION",
  "source_step_id": "S6",
  "applied_recipe": "multi-file content generation",
  "fanout_cardinality_estimate": 5,
  "stub_next_steps": [
    {
      "step_id": "S6.fan.1",
      "agent": "fuhyo",
      "parallel_group": "GEN_S6",
      "atomic_task_template": "{single behavior or file from the original step}",
      "atomicity_proof_template": [
        "goal: {single named action for behavior i}",
        "input: {bounded inputs for behavior i}",
        "output: {explicit format/path for behavior i}",
        "success: {checkable for behavior i}",
        "no_strategy: {only one way for behavior i}"
      ]
    },
    {
      "step_id": "S6.verify",
      "agent": "fuhyo",
      "parallel_group": null,
      "depends_on_data": ["S6.fan.1", "S6.fan.2", "..."],
      "atomic_task_template": "run the project verifier (test command, lint, ...) and report counts"
    }
  ]
}
```

This payload is a *starting point*, not a finished plan — Kakugyō finalizes the per-sibling specifics. The lint never claims to know the user's domain.

## Integration Patterns

### A. Kakugyō self-lint via a Fuhyō pre-flight step

Kakugyō currently lacks `skill: core-plan-lint: allow` in `agents/kakugyo.md`. The non-permission-changing integration is:

1. After Kakugyō builds the plan, it emits a Fuhyō step `S0.lint` with `agent: "fuhyo"`, `authorized_skills: ["core-plan-lint"]`, `input_material: {plan: <the rest of the plan>, user_intent: <refined>}`.
2. Ōshō dispatches `S0.lint` first.
3. If `verdict == "pass"`, Ōshō proceeds with the rest of `next_steps[]`.
4. If `verdict == "fail"`, Ōshō routes to Kakugyō with `mode: "replan_on_blocker"` and the `errors[]` payload as `last_step_blocker.detail`.

### B. Kakugyō direct invocation (requires permission grant)

If `agents/kakugyo.md` is amended to allow `core-plan-lint`, Kakugyō can invoke the skill itself during plan finalization and never return a malformed plan to Ōshō. This is the cleaner shape long-term but requires an explicit permission change.

### C. Ad-hoc lint by the user

`/core-plan-lint --plan path/to/plan.json --user-intent "..."` — for debugging plan structure outside the orchestration loop.

## Validation

Before claiming success:

- [ ] Every check_id listed in this SKILL.md was executed against the input plan.
- [ ] Every violation cites the offending `step_id` (or `INPUT_MALFORMED` for parse failures).
- [ ] No semantic judgment was made — only structural conformance.
- [ ] `errors[]` is the union of all error-severity violations, not the first one.
- [ ] In `--strict`, warnings escalate to errors; otherwise they remain in `warnings[]`.
- [ ] `SUGGESTED_DECOMPOSITION` payloads were emitted for every smell-triggered violation.
- [ ] No mutation of the input plan occurred.
