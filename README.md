# Otsumi

## TL;DR

- Multi-agent harness for Markdown-driven AI CLIs. Native on Claude Code; small adapter shim for Copilot CLI and OpenCode. No SDK, no runtime, no lock-in.
- Architecture is shogi: eight role-bound agents, the King talks to you, the Bishop plans, six specialists execute.
- Each specialist owns exactly one concern: requirements, validation, writing, evidence, atomic execution, challenge.
- 38 stateless skills, domain-prefixed (`agent-`, `core-`, `flow-`, `dev-`, `dev-python-`, `design-`, `doc-`, `git-`, `kb-`); invocable standalone or chained inside a workflow.
- BDD delivery workflow loads on demand. Assisted mode hands off implementation; vibecoding mode automates everything except approval gates.
- Every closed feature leaves Gherkin specs, RED tests, ADRs, scorecards, and append-only event logs in `.otsumi/{feature-name}/`.
- Drift guardrails baked into every agent file. When behavior leaves scope, it routes out instead of bleeding.
- Plain Markdown all the way down. Clone, symlink `system.md` + `agents/` + `skills/`, run. The voice manual ships as the `agent-load-persona` skill.

> *"Checkmate isn't a move. It's a verdict."*

A multi-agent harness that turns any Markdown-driven AI CLI into a disciplined, role-separated execution board. The metaphor is **shogi** (Japanese chess). Each piece has one job. The King talks to you, the Bishop plans, the pieces around them execute. Nothing pretends to be something it isn't.

Native fit for **Claude Code**. Adapted for **Copilot CLI** and **OpenCode** via small instruction shims (see `instructions/`). No SDK, no runtime, plain Markdown all the way down.

---

## What This Is

Most AI coding stacks collapse everything into one agent that does everything badly. Otsumi splits the work along role boundaries that don't blur:

- **One agent talks to the user.** Only one. The rest are hidden.
- **One agent plans the work.** It returns a plan and shuts up.
- **Specialists do the work.** Each owns exactly one concern: requirements, evidence, writing, validation, atomic execution, challenge.

When you need a casual answer, you get one. When you need disciplined feature delivery, the BDD workflow loads and the board organizes around it. When you need anything else (research, prompt refinement, skill creation, code review), there's a workflow or a skill for it.

It's not a vibe-and-ship wrapper. It's a precision instrument that happens to be your general-purpose ally.

---

## The Board

Eight shogi pieces. Each is an agent file in `agents/`. Each owns one role and refuses everything else.

| Piece | Agent | Role |
|---|---|---|
| 王将 King | **Ōshō** (`osho`) | The only user-facing agent. Owns voice, refines prompts, invokes specialists, synthesizes the final answer. The mouth, the mask, the hand on the board; never the board itself. |
| 角行 Bishop | **Kakugyō** (`kakugyo`) | Hidden orchestrator. Takes a refined request, decomposes it, picks agents/skills/workflows, returns an invocation plan. Owns the flow, never the work. |
| 金将 Gold General | **Kinshō** (`kinsho`) | Requirements and acceptance owner. Defines goals, non-goals, output contracts, quality thresholds, definition of done. Decides what "good enough" means before anyone pretends the job is done. |
| 銀将 Silver General | **Ginshō** (`ginsho`) | Quality validator. Checks produced work against Kinshō's contract. Pass / fail / partial / blocked, with reasons sharp enough to act on. Validates only; never repairs. |
| 飛車 Rook | **Hisha** (`hisha`) | Structured writing. Notes, docs, summaries, schemas, decision narratives, polished prose. Shapes thought into readable structure; does not own truth or acceptance. |
| 香車 Lance | **Kyōsha** (`kyosha`) | External evidence. Web search, scraping, source inspection, citations with metadata. Brings back the signal; never crowns the meaning. |
| 歩兵 Pawn | **Fuhyō** (`fuhyo`) | Atomic execution. One bounded task per invocation, explicit input, explicit output. Refuses anything that requires strategy. |
| 桂馬 Knight | **Keima** (`keima`) | Constructive challenger. Finite critique loops on plans and outputs: blind spots, alternatives, simplifications. Caps itself; never spirals into infinite debate. |

The traffic always flows the same way:

```
user → Ōshō → prompt-master refinement → Kakugyō plan → Ōshō invokes specialists → Ōshō synthesizes → user
```

Every actionable request goes through Kakugyō first. No specialist talks to the user. No specialist invokes another specialist. The board stays clean.

---

## Skills

Skills are stateless capability units invoked as `/skill-name`. They live in `skills/{skill-name}/SKILL.md`. Names are domain-prefixed for grep-ability.

| Domain | Examples |
|---|---|
| `agent-` | Prompt refinement, retrospective analysis, skill generation |
| `core-` | Workflow utilities: atomic logs, status, backlog, command recovery |
| `flow-` | Pipeline lifecycle: start, continue, complete, abort, replay, diff |
| `dev-` | Software delivery: BDD workflow, env setup, test runs, quality checks, reviews |
| `dev-python-` | Python adapters: test generator, implementer, refactorer |
| `design-` | Visual/frontend design direction |
| `doc-` | Documentation, editorial refactor, decision records, Excalidraw |
| `git-` | Local git hygiene |
| `kb-` | Knowledge base / second-brain: vault capture, zettelization, assembly, graph config |

Full index: [`skills/REGISTRY.md`](skills/REGISTRY.md).

Skills can run standalone, or be chained inside a workflow. None of them know about the agent layer.

---

## The BDD Workflow

When the request is "deliver this feature properly," Kakugyō selects the BDD delivery workflow. Eight stages. No silent skips.

| Stage | Skill | What Happens |
|---|---|---|
| **S1** | `dev-bdd-gherkin` | Behavior contract written as Gherkin. Every scenario presented for approval. |
| **S2** | `dev-bdd-gherkin` (trap mode) | Adversarial trap analysis: boundaries, auth gaps, concurrency, ambiguous terms. Traps approved before promotion. |
| **S3** | `dev-stage-router` → `dev-{lang}-test-generator` | RED tests generated against surfaces that don't exist yet. RED confirmed before moving on. |
| **S4** | `dev-stage-router` → `dev-{lang}-implementer` | Minimal implementation to GREEN. Nothing more. |
| **S5** | `dev-stage-router` → `dev-{lang}-refactorer` | Atomic refactor. Tests after every change. RED rolls back. |
| **S6** | `doc-decision-record` | ADR/TDR if a real decision was made. Explicit no-record reasoning otherwise. |
| **S7** | `dev-quality-score` | Evidence-backed scoring. Blocking dimensions enforce closure gates. |
| **S8** | `doc-writer` | Human-facing docs from real artifacts only. No invented behavior. |

### Two Modes

**Assisted**: you write the code, the workflow handles the rest.

```
/flow-start-pipeline --mode assisted --lang python "user authentication with email and password"

→ S1 Gherkin spec: approved
→ S2 Trap analysis: approved
→ S3 RED tests confirmed
→ PAUSED: you implement and refactor

/flow-continue user-authentication

→ S6 decisions
→ S7 quality scored
→ S8 docs
→ CLOSED
```

**Vibecoding**: the workflow runs every automatable stage. You approve at the gates.

```
/flow-start-pipeline --mode vibecoding --lang python "cart checkout with coupon validation"
```

You own approvals and architectural intent. Otsumi owns the rest.

---

## Install

Native on Claude Code. Other AI CLIs (Copilot CLI, OpenCode) need a small instruction shim from `instructions/` to handle subagent delegation. Clone once, symlink into your tool's config directory.

Two surfaces do separate jobs:

- `system.md`: short, role-agnostic system context loaded for every agent invocation. Symlink this as your tool's global instruction file.
- `skills/agent-load-persona/SKILL.md`: the full Otsumi voice manual, packaged as a skill. Loaded by Ōshō on session start via `/agent-load-persona`. No path discovery, no `view` — the harness resolves it.

### 1. Clone

```bash
git clone https://github.com/Kakudou/otsumi-agentic.git ~/.otsumi
```

### 2. Symlink

**Claude Code**

```bash
ln -s ~/.otsumi/system.md   ~/.claude/CLAUDE.md
ln -s ~/.otsumi/agents      ~/.claude/agents
ln -s ~/.otsumi/skills      ~/.claude/skills
```

**Copilot CLI** (also needs `instructions/copilot-subagent.md` for delegation)

```bash
ln -s ~/.otsumi/system.md   ~/.copilot/copilot-instructions.md
ln -s ~/.otsumi/agents      ~/.copilot/agents
ln -s ~/.otsumi/skills      ~/.copilot/skills
ln -s ~/.otsumi/instructions/copilot-subagent.md  ~/.copilot/copilot-subagent.md
```

**OpenCode**

```bash
ln -s ~/.otsumi/system.md   ~/.config/opencode/AGENTS.md
ln -s ~/.otsumi/agents      ~/.config/opencode/agents
ln -s ~/.otsumi/skills      ~/.config/opencode/skills
```

### 3. Run

Open a project in your AI CLI. Ōshō is live. She answers to "Otsumi."

For Copilot CLI specifically:

```bash
copilot --allow-all-tools
```

The `--allow-all-tools` flag enables the `task` tool used to spawn subagents. Required for the orchestration loop. If you'd rather pin the toolset per-agent in frontmatter, you can, but you lose plug-and-play portability across tools.

> **Key insight:** the entire system is plain Markdown. No SDK, no runtime, no lock-in. Claude Code runs it natively; other AI CLIs need a small adapter file from `instructions/`.

---

## Customization

| What | Where | How |
|---|---|---|
| System context (every agent) | `system.md` | Keep lean. Loaded for every invocation. Add hard floors that apply to all roles. |
| Ōshō voice | `skills/agent-load-persona/SKILL.md` | Rewrite to taste. Loaded by Ōshō on session start via `/agent-load-persona`, NOT by specialists. |
| Agent role | `agents/{name}.md` | Tweak input/output contracts, drift guardrails, hard rules. |
| Workflow stages | `skills/dev-bdd-workflow/SKILL.md` | Change the stage table, the routing skills, the remediation policy. |
| New language | Add `dev-{lang}-test-generator`, `dev-{lang}-implementer`, `dev-{lang}-refactorer` skills. Update `dev-stage-router` routing table. Add detection rules in `dev-env-setup`. | The board doesn't change. Only adapters do. |
| New skill | `skills/{name}/SKILL.md` | Follow the canonical layout. Or use `/agent-create-skill` to distill one from completed work. |
| Triage tiers | `agents/osho.md` (Request Triage section) | Adjust what counts as Tier 0/1/2. Default to escalation, never downgrade on doubt. |

---

## What Gets Left Behind

Every closed feature leaves a trail in `.otsumi/{feature-name}/`:

```
pipeline.json          ← routing selectors, mode, status, timestamps
stage-01-output.json   ← approved scenarios
stage-02-output.json   ← trap analysis summary
stage-03-output.json   ← RED test confirmation
stage-04-output.json   ← implementation summary
stage-05-output.json   ← refactor summary
stage-06-output.json   ← decisions or explicit no-record reasoning
stage-07-output.json   ← scorecard, dimensions, remediation history
stage-08-output.json   ← documentation written
events.json            ← append-only event log: every action, timestamped
```

And in the project itself:

```
features/{feature-name}.feature   ← the approved spec
tests/                            ← the test suite
src/                              ← the implementation
docs/decisions/                   ← ADR + TDR records
```

Nothing is invented. Every artifact is earned.

---

## The Prompt Triad

Three skills that chain into a learning loop:

- **`/agent-retro-prompt`** is the **observer**. It looks at what actually happened and extracts grounded friction. No hypothetical advice, only friction paid for in real rework.
- **`/prompt-master`** is the **weaponizer**. It forges a rough idea into a production-ready prompt for any target tool: Claude, GPT, Gemini, Cursor, Copilot, Midjourney, Sora, ElevenLabs, the lot.
- **`/agent-create-skill`** is the **crystallizer**. It chains the two and reshapes the output into a durable `SKILL.md`.

```
retro-prompt   →  plain language friction + improved prompt
     ↓
prompt-master  →  production-ready prompt for the target tool
     ↓
create-skill   →  SKILL.md with usage, hard rules, steps
```

The friction is paid once. The learning is permanent. Knowledge that would die with the conversation gets crystallized into reusable automation.

---

## The Knowledge Metabolism

Four skills that chain into a second brain:

- **`/kb-obsidian-remember`** is the **intake**. Raw capture, byte-for-byte, no opinion. Drops anything into the raw inbox with an epoch filename and gets out of the way.
- **`/kb-obsidian-zettelize`** is the **atomizer**. Decomposes any source into atomic Zettelkasten notes — one idea per file. Deduplicates against what already exists; updates instead of duplicating.
- **`/kb-obsidian-assemble`** is the **curator**. Synthesizes existing zettels into a topical resource by linking and embedding. Librarian, not writer — surfaces gaps instead of filling them.
- **`/kb-obsidian-config`** is the **lens**. Configures the graph view — colors, forces, search — so you can see what the other three built.

```
remember    →  raw content in the inbox, untouched
    ↓
zettelize   →  atomic notes, deduplicated, linked
    ↓
assemble    →  topical resource, synthesized from zettels
    ↓
config      →  graph view tuned to surface the structure
```

The intake is cheap. The knowledge compounds.

---

## Why This Exists

Most AI coding tools solve the wrong problem. They make *writing* code faster. They do nothing for the discipline around it.

Where is the spec, and where are the test scenarios? Who decided on this architecture and why? What did the quality check actually find?

Otsumi answers those questions by making them unavoidable. Every feature delivered through the BDD workflow leaves behind:

- a Gherkin spec that survives the next sprint
- adversarial traps caught before they cost anything
- RED tests proving the bug existed
- minimal implementation that GREEN-fixed it
- ADR/TDR records of every load-bearing decision
- evidence-backed quality scores with blocking thresholds
- human-facing docs generated from artifacts, not summaries

You choose what you own. You choose what the workflow owns. The workflow never pretends a stage was completed when it wasn't.

---

## Design Principles

**Roles do not blur.** Ōshō is the only mouth, Kakugyō plans but never executes, specialists own one concern. The architecture refuses scope creep at the agent level.

**Drift guardrails are explicit.** Every agent file lists what to do when it starts behaving like a different agent: route the work to the agent that actually owns that concern.

**No hidden state.** Every stage writes a JSON artifact, every event appends to an immutable log. Recovery, replay, and audit are first-class.

**Plain Markdown over framework.** A multi-agent system shouldn't require an SDK. It should require a directory of `.md` files and an AI CLI that can read them.

**Skills are weapons. Agents are trigger fingers. Ōshō is fire control.**

---

*Signal over noise. Always.*

**Written by the Hand of Otsumi and the Eyes of Kakudou.**
