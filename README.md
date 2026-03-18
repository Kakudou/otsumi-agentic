# Otsumi

> *"Checkmate isn't a move. It's a verdict."*

**Otsumi** is a jack-of-all-trades AI agent that handles any user inquiry — and when the work is a production feature, she loads the BDD delivery pipeline that turns a description into closed, scored, documented code, without letting the machine make the decisions that matter.

She is the front gate. Every request enters through her. Simple questions get direct answers. Code edits get done. Prompt engineering, research, writing — handled. But when you need a disciplined feature delivery, she optimizes your prompt, loads the pipeline brain, and drives the full BDD workflow with specialist subagents.

It is not a vibe-and-ship wrapper. It is a precision instrument that also happens to be your general-purpose ally.

---

## Why This Exists

Most AI coding tools solve the wrong problem. They make the *writing* of code faster. They do nothing for the discipline around it.

What happens to the spec? Where are the test scenarios? Who decided on this architecture and why? What did the quality check actually find?

Otsumi answers those questions by making them unavoidable.

Every feature runs through a staged pipeline:

- behavior defined as Gherkin before a line of code is written
- adversarial trap analysis to surface edge cases the happy-path spec missed
- RED tests generated against an implementation surface that does not exist yet
- implementation that makes those tests GREEN, nothing more
- atomic refactor with tests enforced at every step
- architectural decisions recorded as ADR/TDR so they survive the next sprint
- 7-dimension quality scoring with mandatory evidence before closure
- human-facing documentation generated from real artifacts, not invented summaries

You choose what you own. You choose what the pipeline owns. The pipeline never pretends a stage was completed when it was not.

---

## Two Modes

### Assisted

You write the code. Otsumi handles everything else.

The pipeline runs the spec, the traps, the RED tests, the decisions, the quality review, and the docs. You implement and refactor at your own pace. When you're done, `/continue` validates your work and advances the pipeline.

```
/start-pipeline --mode assisted --lang python \
  user authentication with email and password

→ [S1] Gherkin spec written, scenarios presented for your approval
→ [S2] Trap analysis run, edge cases surfaced for your approval
→ [S3] RED tests generated, confirmed failing
→ PIPELINE PAUSES, you implement

/continue user-authentication

→ [S6] ADR/TDR recorded
→ PIPELINE PAUSES, you review the scorecard

/continue user-authentication

→ [S7] Quality scored: 7 dimensions, 4/5 minimum each
→ [S8] Documentation generated
→ CLOSED
```

**You own:** the implementation, the refactor, and the pace.  
**Otsumi owns:** spec, traps, tests, decisions, quality review, docs, and the gates between them.

### Vibecoding

The pipeline handles every automated stage. You approve at checkpoints.

```
/start-pipeline --mode vibecoding --lang python \
  cart checkout with coupon validation

→ All 8 stages run end-to-end
→ You approve scenarios, traps, and the final scorecard
→ CLOSED
```

**You own:** approval gates and architecture guidance.  
**Otsumi owns:** everything that can be automated.

---

## Install

Otsumi is tool-agnostic. The pipeline lives in plain Markdown. Clone once, symlink into whatever AI CLI you use.

### 1. Clone

```bash
git clone https://github.com/kakudou/otsumi-agentic.git ~/.otsumi-agentic
```

### 2. Symlink

The most easiest way is to create a simlink in your tool's configuration directory.

#### For Claude Code

```bash
ln -s ~/.otsumi-agentic/persona.md      ~/.claude/CLAUDE.md
ln -s ~/.otsumi-agentic/*               ~/.claude/
```

#### For Copilot CLI

```bash
ln -s ~/.otsumi-agentic/persona.md      ~/.copilot/copilot-instructions.md
ln -s ~/.otsumi-agentic/*    ~/.copilot/
```

#### For OpenCode

```bash
ln -s ~/.otsumi-agentic/persona.md      ~/.config/opencode/AGENTS.md
ln -s ~/.otsumi-agentic/*    ~/.config/opencode/
```

### 3. Customization

Change `persona.md` to customize the persona of your primary agent.
Change the model in each agents to match your preferences.

If you need a different language than python you gonna need to:
- Create the adapter skills `<my-language>-implementer`, `<my-language>-refactorer`, `<my-language>-test-generator`
- Create the rules in `env-setup`

### 4. Usage

Open a project in your AI CLI. The pipeline is live. Otsumi answers to her name.
for Copilot CLI:
```bash
copilot --allow-all-tools
```
The option `--allow-all-tools` enables Otsumi to run on any tool that reads Markdown-based agent instructions, this is NECESSARY to run the 'hidden' `tasks` tools used to delegate/spawn subagents.
You can leverage that option, by settings all your tools including `tasks` in your agent frontmatter, but this will break direct portability to other tools.

> **Key insight:** the pipeline is plain Markdown. Any tool that reads `.md` files from a config directory can run Otsumi. No runtime dependency, no SDK, no lock-in.

---

## Routing Model

Every pipeline starts with two explicit selectors. No inference from natural language.

```
--mode <mode>              assisted or vibecoding?
--lang <language_id>       what ecosystem are we in?
```

The pipeline resolves the `language_id` into a `stages` list and discovers the test framework from the project. No additional flags required.

Adapter skills declare what they handle. Adding a new language means creating the adapter skills and updating the stage router. The pipeline architecture does not change.

---

## The Pipeline in Eight Stages

| Stage | Name | What Happens |
|-------|------|--------------|
| **S1** | Gherkin Spec | Feature description becomes an approved `.feature` file. Every scenario presented for your approval, silence is not consent. |
| **S2** | Trap Analysis | Adversarial analysis across 7 trap categories: boundary conditions, auth gaps, concurrency, state machines, data integrity, external dependencies, and missing behaviors. Every trap surfaced for your approval. |
| **S3** | RED Tests | Real failing tests generated against implementation surfaces that do not exist yet. Confirmed RED before advancing. |
| **S4** | Implementation | Minimal code to make the RED tests GREEN. One test at a time. Nothing more. Gold plating surfaced to you before output is finalized. |
| **S5** | Refactor | Behavior-preserving improvements. Tests run after every atomic change. Any change that turns the suite RED is immediately undone. |
| **S6** | Decisions | ADR/TDR evaluation against a trigger checklist. Written when triggered. Explicit `no_record_reasoning` when not. Nothing silently skipped. |
| **S7** | Quality Scoring | 7 dimensions scored with evidence. Minimum 4/5 each. Single dimension at 3/5 blocks closure. Remediation loop with 3-cycle limit before ESCALATED. |
| **S8** | Documentation | Human-facing docs generated from real pipeline artifacts: README sections, API reference, changelog entries, Obsidian notes. No invented behavior. |

---

## The Agent Team

Otsumi coordinates a team of specialist subagents. You never talk to them directly, everything surfaces in the Otsumi thread with full context.

| Agent | Stage | Role |
|-------|-------|------|
| `gherkin-keeper` | S1 + S2 | Spec creation and adversarial trap analysis |
| `bdd-test-generator` | S3 | Language-routed RED test generation |
| `minimal-implementer` | S4 | Minimal implementation, one test at a time |
| `safe-refactorer` | S5 | Atomic refactor with enforced GREEN gate |
| `decision-keeper` | S6 | ADR/TDR authoring |
| `quality-scorer` | S7 | Evidence-based 7-dimension scoring |
| `doc-writer` | S8 | Human-facing documentation |

Each agent validates upstream artifacts before executing, writes its own `stage-XX-output.json` as a handoff contract, and returns control to Otsumi. No agent invokes the next one directly.

Skills sit beneath the agents, stateless, standalone-capable capability units that can also be invoked directly without a running pipeline:

```
# Trap analysis on an existing .feature file, no pipeline needed
Ask Otsumi: run gherkin-keeper on features/my-thing.feature

# Deep expert review on a feature, no pipeline needed
Ask Otsumi: run expert-code-review for my-thing

# Write docs for a closed feature
Ask Otsumi: run doc-writer for my-feature, doc_types: [obsidian, changelog]
```

---

## Commands (Stateless skills invoked by users)

| Command | What It Does |
|---------|--------------|
| `/start-pipeline --mode assisted --lang <l> <description>` | Start an assisted pipeline. You implement. |
| `/start-pipeline --mode vibecoding --lang <l> <description>` | Start a vibecoding pipeline. Otsumi implements. |
| `/continue <feature-name>` | Resume an assisted pipeline from its next stage. Guards against closed or running pipelines. |
| `/env-setup --lang <l>` | Prepare the runtime environment. |
| `/run-tests <feature-name>` | Run the test suite. |
| `/quality-check <feature-name>` | Run the four tool-backed quality dimensions (lint, format, imports, types). |
| `/delivery-review <feature-name>` | Run structured review across architecture, test quality, and docs. |
| `/expert-code-review <path>` | Run a deep standalone code review with expert scorecard. |
| `/complete-stage <feature-name> <stage-id> "<log-details>"` | Finalize a stage: update pipeline routing, log completion, handle pause/close. |
| `/atomic-log <feature-name> <event> <summary>` | Append one event to the feature's append-only log. |
| `/retro-prompt <feature-name>` | Analyse a closed pipeline and return a better version of the original prompt. Auto-runs at pipeline close. |
| `/retro-prompt` | Standalone, works on any session without a running pipeline. |
| `/retro-feature <path>` | Read existing source code, identify distinct features, and produce approved Gherkin specs ready to feed into the pipeline. Stack-agnostic, no pipeline context required. |
| `/retro-feature <path> --tests <tests-path>` | Same as above, with supplemental test files as behavioral evidence. |
| `/status <feature-name>` | Show detailed pipeline status for a feature. |
| `/status` | List all pipelines found in `.otsumi/`. |
| `/abort <feature-name>` | Gracefully abort a running or paused pipeline. Preserves all artifacts. |
| `/backlog` | List all planned features from the Feature Backlog Protocol. |
| `/diff-stage <feature-name> <stage-id>` | Show what changed at a specific pipeline stage. |
| `/replay <feature-name> --from <stage-id>` | Re-run a closed or aborted pipeline from a specific stage. |
| `/prompt-master` | Optimize any prompt for any AI tool. Surgical, credit-efficient, zero-reprompt output. Used automatically before every pipeline start. |
| `/create-skill <feature-name>` | Distill a closed pipeline into a reusable skill: runs retro-prompt, then prompt-master, outputs a ready SKILL.md. |
| `/create-skill <name> "<description>"` | Distill the current conversation or task into a reusable skill. |

---

## Quality Gate

A feature is **CLOSED** only when all 7 dimensions score ≥ 4/5.

| Dimension | Source | What It Measures |
|-----------|--------|------------------|
| `linting` | `/quality-check` | Lint status |
| `formatting` | `/quality-check` | Code formatting compliance |
| `import_hygiene` | `/quality-check` | Import ordering and hygiene |
| `type_safety` | `/quality-check` | Type check or equivalent |
| `architecture` | `/delivery-review` | Structure, boundaries, naming, modeling |
| `test_quality` | `/delivery-review` | Behavioral coverage and test reliability |
| `docs_quality` | `/delivery-review` | Decision record completeness |

One dimension at 3/5 blocks closure regardless of overall average. On failure: Otsumi routes the fix to the responsible agent, re-runs the evidence, and re-scores. Maximum 3 remediation cycles. On cycle 4: `ESCALATED`, manual intervention with explicit options.

---

## What Gets Left Behind

Every closed feature leaves a trail in `.otsumi/<feature-name>/`:

```
pipeline.json          ← routing selectors, mode, status, timestamps
stage-01-output.json   ← approved scenario list
stage-02-output.json   ← trap analysis summary
stage-03-output.json   ← test generation summary, RED confirmation
stage-04-output.json   ← implementation summary, test infrastructure changes
stage-05-output.json   ← refactor summary
stage-06-output.json   ← ADR/TDR decisions or explicit no-record reasoning
stage-07-output.json   ← scorecard, dimension scores, remediation history
stage-08-output.json   ← documentation written
atomic-log.json        ← append-only event log: every action, timestamped
```

And in the project:

```
features/<feature-name>.feature     ← the approved spec
tests/                              ← the test suite
src/                                ← the implementation
docs/decisions/                     ← ADR + TDR records
```

Nothing is invented. Every artifact is earned.

---

## Design

**Skills are weapons. Agents are trigger fingers. Otsumi is the fire control system.**

The three-layer design keeps the system extensible without breaking what's already working:

- **Skills** do one thing well and have no pipeline awareness. They can run standalone.
- **Agents** own the gates: validate state, resolve inputs, invoke the skill, write the handoff JSON, log the event, return to Otsumi.
- **Otsumi** owns the flow: initialize, route, remediate, close. She is also the front gate for *any* user request, not just pipelines.

### The Front Gate Architecture

Otsumi is a jack-of-all-trades primary agent. She handles any inquiry directly — code, writing, research, analysis, prompt engineering. When the request involves a BDD pipeline, she loads the `bdd-orchestrator` skill, which gives her the pipeline orchestration brain. She then spawns stage subagents directly and drives the workflow.

This means there is no separate orchestrator agent. The orchestration logic lives as a skill that Otsumi loads on demand. Why? Because subagents cannot spawn other subagents — if the orchestrator were a subagent, it could not delegate to gherkin-keeper, minimal-implementer, etc. Making it a skill solves this: the logic runs inside Otsumi's context, and Otsumi retains her ability to spawn subagents.

### The Prompt Triad: retro-prompt, prompt-master, create-skill

Three skills, each with a clean single responsibility, that chain into a learning loop.

**`/retro-prompt`** is the **observer**. It looks backward at what actually happened — which traps were missed, which constraints caused rework, which assumptions turned out wrong — and extracts grounded lessons. No hypothetical advice, only friction that was paid for in real rework. Its output is plain natural language, tool-agnostic. That's its power and its limit: it produces insight, not automation.

**`/prompt-master`** is the **weaponizer**. It takes any rough idea and forges it into a production-ready prompt optimized for a specific tool. It doesn't care where the input comes from — a user typing, a retro-prompt output, a napkin sketch. It handles any AI tool: Claude, ChatGPT, Gemini, Cursor, Copilot, Midjourney, DALL-E, Stable Diffusion, ComfyUI, Sora, ElevenLabs, Zapier, and more. Its job is format, structure, and precision.

**`/create-skill`** is the **crystallizer**. It chains the two in sequence — observer feeds weaponizer — and adds a third step: reshaping the output into a durable `SKILL.md` artifact. Without it, you'd manually run retro-prompt, copy the output, run prompt-master, then format the result as a skill file. `create-skill` makes that pipeline atomic.

```
retro-prompt  →  plain natural language + friction points + scores
     ↓
prompt-master →  production-ready prompt for target tool
     ↓
create-skill  →  SKILL.md file with Usage, Steps, Hard Rules
```

No skill knows about the others' internals. `retro-prompt` doesn't know it's feeding `prompt-master`. `prompt-master` doesn't know its input came from a retrospective. `create-skill` is just the orchestration that chains them and adds the file-writing ceremony.

The result: knowledge that would die with the conversation gets crystallized into reusable automation. The friction was paid once. The learning is permanent.

```
# After closing the invoice-pdf-export pipeline:
/create-skill invoice-pdf-export

→ Phase 1: retro-prompt analyses the pipeline, finds 3 friction points
  (timezone-naive date filtering, silent unicode drops, missing locale)
→ Phase 2: prompt-master converts the improved prompt into a SKILL.md
→ Phase 3: you review and approve

→ skills/pdf-export/SKILL.md is written
→ /pdf-export is now a command
→ Next PDF export starts with those 3 traps pre-embedded as constraints
```

### Extensibility

Adding a new language means creating the adapter skills and updating the stage router. The pipeline architecture does not change.

---

*Signal over noise. Always.*

**Written by the Hand of Otsumi and the Eyes of Kakudou.**
