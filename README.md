# Otsumi

## TL;DR

- Multi-agent harness for Markdown-driven AI CLIs. Native on Claude Code; small adapter shim for Copilot CLI and OpenCode. No SDK, no runtime, no lock-in.
- Architecture is shogi: eight role-bound agents, the King talks to you, the Bishop plans, six specialists execute.
- Each specialist owns exactly one concern: requirements, validation, writing, evidence, atomic execution, challenge.
- 45 stateless skills, domain-prefixed (`agent-`, `core-`, `flow-`, `dev-`, `dev-python-`, `design-`, `doc-`, `git-`, `kb-`); invocable standalone or chained inside a workflow.
- BDD delivery workflow loads on demand. Assisted mode hands off implementation; vibecoding mode automates everything except approval gates.
- Every closed feature leaves Gherkin specs, RED tests, ADRs, scorecards, and append-only event logs in `.otsumi/{feature-name}/`.
- Drift guardrails baked into every agent file. When behavior leaves scope, it routes out instead of bleeding.
- Plain Markdown all the way down. Clone, symlink `system.md` + `agents/` + `skills/`, run. The voice manual ships as the `agent-load-persona` skill.

> *"Checkmate isn't a move. It's a verdict."*

A multi-agent harness that turns any Markdown-driven AI CLI into a disciplined, role-separated execution board.
The metaphor is **shogi** (Japanese chess). Each piece has one job. The King talks to you, the Bishop plans,
the pieces around them execute. Nothing pretends to be something it isn't.

Most AI coding stacks collapse everything into one agent that does everything badly. Otsumi splits work
along role boundaries that don't blur: one agent talks to you, one plans, and six specialists each own
exactly one concern. When you need a casual answer, you get one. When you need disciplined feature delivery,
the BDD workflow loads and the board organizes around it. When you need research, prompt refinement, skill
creation, or code review, there's a workflow or a skill for it.

Native fit for **Claude Code**. Adapted for **Copilot CLI** and **OpenCode** via small instruction shims in
`instructions/`. No SDK, no runtime, plain Markdown all the way down.

---

## Getting Started

Clone once, symlink into your tool's config directory, and run.

### 1. Clone

```bash
git clone https://github.com/Kakudou/otsumi-agentic.git ~/.otsumi
```

### 2. Symlink

Two surfaces do separate jobs. `system.md` is the short, role-agnostic context loaded for every agent
invocation; symlink it as your tool's global instruction file. `skills/agent-load-persona/SKILL.md` is the
full Ōshō voice manual, packaged as a skill and loaded by Ōshō on session start via `/agent-load-persona`.

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

The `--allow-all-tools` flag enables the `task` tool used to spawn subagents. Required for the orchestration
loop. If you'd rather pin the toolset per-agent in frontmatter, you can, but you lose plug-and-play
portability across tools.

> **Key insight:** the entire system is plain Markdown. No SDK, no runtime, no lock-in. Claude Code runs it
> natively; other AI CLIs need a small adapter file from `instructions/`.

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

Traffic always flows the same way:

```
user → Ōshō → prompt-master refinement → Kakugyō plan → Ōshō invokes specialists → Ōshō synthesizes → user
```

Every actionable request goes through Kakugyō first. No specialist talks to the user. No specialist invokes
another specialist. The board stays clean.

---

## Delivering a Feature

When the request is "deliver this feature properly," Kakugyō selects the BDD delivery workflow. Eight stages,
no silent skips.

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

## Skills

Skills are stateless capability units invoked as `/skill-name`. They live in `skills/{skill-name}/SKILL.md`.
Names are domain-prefixed for grep-ability. Skills run standalone or chain inside a workflow. None of them
know about the agent layer.

Full index: [`skills/REGISTRY.md`](skills/REGISTRY.md).

### agent- Skills

The agent- family covers prompt refinement, retrospective analysis, and skill crystallization. Three of the
four skills chain into the Prompt Triad: retro finds friction, prompt-master forges it, create-skill saves
it. The fourth loads the Ōshō persona on session start.

| Skill | What it does |
|---|---|
| `/agent-retro-prompt` | Reviews a completed pipeline, task, or conversation. Extracts grounded friction and returns a better starting prompt. No hypothetical advice: only friction paid for in real rework. |
| `/prompt-master` | Forges a rough idea into a production-ready prompt for any target tool: Claude, GPT, Gemini, Cursor, Copilot, Midjourney, Sora, ElevenLabs. Activates only on explicit prompt-engineering requests. |
| `/agent-create-skill` | Chains agent-retro-prompt then prompt-master, then crystallizes the output into a durable SKILL.md. One invocation, one new reusable skill. |
| `/agent-load-persona` | Loads the project-specific Ōshō voice manual on session start. Not portable across projects without a rewrite. |

See [The Prompt Triad](#the-prompt-triad) for the full chain workflow.

### core- Skills

The core- family provides infrastructure primitives: append-only log writes, status reads, plan validation,
backlog listing, and command recovery. Flow- and dev- workflows depend on these.

| Skill | What it does |
|---|---|
| `/core-atomic-log` | Appends a single event to an append-only JSON log without corrupting history. The write primitive all workflow logging depends on. |
| `/core-backlog` | Lists planned feature or task records produced by workflow escalation. Read-only; never starts anything automatically. |
| `/core-fix-command` | Resolves command or skill lookup mismatches. Returns the likely intended command with a safe correction. |
| `/core-plan-lint` | Validates a Kakugyō plan against atomicity, dependency semantics, parallel group safety, blocker vocabulary, and shogi-roster discipline. Returns pass/fail with violations and decomposition suggestions. |
| `/core-status` | Displays current status for one workflow/pipeline, or summarizes all workflow state roots. Run this first when resuming interrupted work. |

### flow- Skills

The flow- family manages pipeline lifecycle. Every BDD pipeline runs on these primitives. Use them to start,
advance, pause, resume, replay, or inspect any workflow stage.

| Skill | What it does |
|---|---|
| `/flow-start-pipeline` | Starts one or more explicit workflow pipelines. Accepts mode, language, test style, stages, state root, and routing selectors. |
| `/flow-continue` | Resumes a paused pipeline. Validates any user-owned implementation or refactor work before advancing. |
| `/flow-complete-stage` | Finalizes a workflow stage: updates state, logs completion, resolves the next active stage, and applies pause/close behavior. |
| `/flow-abort` | Aborts a running or paused pipeline safely. Preserves artifacts and records the reason. |
| `/flow-replay` | Replays a closed or aborted pipeline from a selected stage. Preserves upstream artifacts. Requires overwrite confirmation. |
| `/flow-diff-stage` | Shows the artifacts and file changes associated with one stage output. |

**Common workflows**

```
# Start a new pipeline
/flow-start-pipeline --mode vibecoding --lang python "feature name"

# Check pipeline state
/core-status

# Resume after a pause gate
/flow-continue feature-name

# Inspect what a stage produced
/flow-diff-stage feature-name --stage 3

# Abort safely
/flow-abort feature-name
```

### dev- Skills

The dev- family covers the full software delivery lifecycle: behavior specs, environment setup, test
execution, quality checks, code review, and documentation. Twelve skills total. [Delivering a Feature](#delivering-a-feature)
documents how they chain in stage order. Use them standalone for targeted review and inspection outside a
full pipeline.

| Skill | What it does |
|---|---|
| `/dev-bdd-gherkin` | Writes Gherkin behavior specs with user approval gates. Second pass runs adversarial trap analysis: boundaries, auth gaps, concurrency, ambiguous terms. Traps promoted to scenarios only after approval. |
| `/dev-bdd-workflow` | The full BDD delivery workflow doctrine: specification, traps, RED tests, minimal implementation, refactor, decisions, quality score, docs. Eight stages, no silent skips. |
| `/dev-stage-router` | Routes language-bound delivery stages (S3 through S5) to the correct language adapter skill. |
| `/dev-env-setup` | Prepares and verifies the development runtime for a selected language and project. Does not silently rewrite project architecture. |
| `/dev-run-tests` | Runs the project test suite or a selected subset. Reports RED/GREEN state with counts and failures. |
| `/dev-quality-check` | Tool-backed quality checks: linting, formatting, import hygiene, type safety. |
| `/dev-quality-score` | Scores output using evidence and thresholds. Produces CLOSED, REMEDIATION REQUIRED, or ESCALATED verdicts. |
| `/dev-delivery-review` | Reviews delivery quality across architecture, maintainability, tests, documentation readiness, and scope discipline. |
| `/dev-expert-code-review` | Deep expert code review with concrete findings, risk levels, and scorecard evidence. |
| `/dev-retro-feature` | Reverse-engineers existing code or behavior into feature candidates, gap reports, and Gherkin-ready specs. Never mutates source. |
| `/dev-github-issue-safe-fix` | Handles a GitHub issue by ID through the BDD workflow. Includes issue verification, test-suite convention scanning, prompt refinement, and user-selected pipeline mode. No commit or push. |
| `/dev-opencti-create-connector` | Scaffolds a production-grade OpenCTI external-import connector: typed settings, API client, STIX mapping, state management, tests, docs. |

### dev-python- Skills

Python-specific language adapters for BDD stages S3 through S5. Routed automatically by `/dev-stage-router`
when `--lang python` is specified. Use them standalone to run a single stage without a full pipeline.

| Skill | What it does |
|---|---|
| `/dev-python-test-generator` | Generates RED tests from approved behavior using pytest-bdd or plain pytest helper style depending on project conventions. |
| `/dev-python-implementer` | Implements the smallest Python code needed to make approved tests pass. No gold plating. |
| `/dev-python-refactorer` | Performs safe, atomic, behavior-preserving Python refactors. Runs tests after every change. RED rolls back. |

### design- Skills

| Skill | What it does |
|---|---|
| `/design-frontend` | Creates production-grade frontend interfaces with strong design direction. Anti-slop constraints built in. |

### doc- Skills

The doc- family covers the documentation lifecycle: generate from artifacts, record decisions, refactor prose,
and produce visual diagrams. All four write from real evidence; none invent behavior.

| Skill | What it does |
|---|---|
| `/doc-writer` | Generates human-facing documentation from real artifacts only. Never invents behavior or summarizes what didn't happen. |
| `/doc-decision-record` | Creates ADR/TDR records or explicit no-record reasoning, grounded in actual implementation and workflow evidence. |
| `/doc-editorial-refactor` | Rewrites Markdown with sharp editorial voice, TL;DR, clean typography, and zero em dashes. Preserves all technical blocks unchanged. |
| `/doc-stylish-excalidraw` | Creates Excalidraw diagrams and document-layout boards with cyberpunk-terminal styling and real bound-arrow semantics. |

### git- Skills

| Skill | What it does |
|---|---|
| `/git-commits` | Creates safe local atomic commits. Focused diffs, short one-line messages, gitmoji discipline. No pushes. |

### kb- Skills

Nine skills for personal knowledge management in Obsidian vaults. Vault paths resolve from `system.md`;
nothing is hardcoded. See [The kb-obsidian Skill Ecosystem](#the-kb-obsidian-skill-ecosystem) for the full
pipeline, support layer, usage score tracking, common workflows, and family reference table.

---

## The Prompt Triad

Three skills that chain into a learning loop:

- **`/agent-retro-prompt`** is the **observer**. It looks at what actually happened and extracts grounded
  friction. No hypothetical advice, only friction paid for in real rework.
- **`/prompt-master`** is the **weaponizer**. It forges a rough idea into a production-ready prompt for any
  target tool: Claude, GPT, Gemini, Cursor, Copilot, Midjourney, Sora, ElevenLabs, the lot.
- **`/agent-create-skill`** is the **crystallizer**. It chains the two and reshapes the output into a
  durable `SKILL.md`.

```
retro-prompt   →  plain language friction + improved prompt
     ↓
prompt-master  →  production-ready prompt for the target tool
     ↓
create-skill   →  SKILL.md with usage, hard rules, steps
```

The friction is paid once. The learning is permanent. Knowledge that would die with the conversation gets
crystallized into reusable automation.

---

## The kb-obsidian Skill Ecosystem

Nine skills for managing personal knowledge in Obsidian vaults. Four form the main pipeline: `remember` is
the intake (raw capture, byte-for-byte, no opinion), `zettelize` is the atomizer (one idea per file,
deduplicated), `assemble` is the curator (topical synthesis, not generation), and `write` turns vault
knowledge into polished prose. Five more handle retrieval, health, logging, archival, and graph
configuration. All paths resolve from `system.md`; nothing is hardcoded. All refuse git operations. All
write operations require human approval before committing anything to the vault.

The intake is cheap. The knowledge compounds.

### The Knowledge Pipeline

```
Raw content → remember → zettelize → assemble → write → doc-writer
                            ↑                      ↑
                         (atomize)          (optional bypass)
```

| Skill | Role |
|---|---|
| **`/kb-obsidian-remember`** | Byte-for-byte raw capture into `raw_root`. Epoch-timestamped filename. No formatting, no opinion. Gets out of the way. |
| **`/kb-obsidian-zettelize`** | Atomizes any source into one-idea-per-file zettels in `zettel_root`. Deduplicates against existing zettels; updates instead of duplicating. |
| **`/kb-obsidian-assemble`** | Curates existing zettels into topical resource notes. Librarian, not writer: links and embeds only. Surfaces gaps rather than filling them. |
| **`/kb-obsidian-write`** | Generates polished prose from vault knowledge. Mandatory zettel citations. Accepts an optional template (blog post, briefing, essay) or freestyle for open-ended prose. Can bypass `assemble` and take zettels directly. |

`doc-writer` (downstream, optional) can refactor `write` output into personal voice or house style.

### The Support Layer

Four primitives that the pipeline skills depend on; call them standalone too.

| Skill | Role |
|---|---|
| **`/kb-obsidian-search`** | Retrieval primitive. Find zettels by semantic query, tag intersection, temporal range, or title/alias match. Used standalone or as infrastructure by other skills. |
| **`/kb-obsidian-lint`** | Vault health checks. Five enumerated checks: `orphan-zettel`, `unassembled-cluster`, `missing-crosslink`, `stale-candidate`, `candidate-contradiction`. Read-only. Never recommends content generation. |
| **`/kb-obsidian-log`** | Operations logging. Append-only event log via `core-atomic-log`. Query interface for operation history. Silent degradation on failure. |
| **`/kb-obsidian-archive`** | Lifecycle management. Marks notes as archived, superseded, or deprecated. Never deletes. The remediation path for `lint`'s `stale-candidate` findings. |
| **`/kb-obsidian-config`** | Graph view configuration: color groups, search lens, display, forces. Makes the structure visible. |

### Usage Score Tracking

Every zettel carries two frontmatter fields that every skill respects:

- `total_access`: incremented by any skill that reads the zettel
- `use_count`: incremented only on productive use (citation, merge, assembly inclusion)

This creates a usage-weighted knowledge graph. High-score zettels are foundational knowledge. Low-score
zettels with high age are candidates for `lint`'s `stale-candidate` check and eventual `archive`.

### Common Workflows

**Capture and process**
```
/kb-obsidian-remember    # drop raw content into inbox
/kb-obsidian-zettelize   # atomize into zettel_root
```

**Build a topic overview**
```
/kb-obsidian-search      # find relevant zettels
/kb-obsidian-assemble    # synthesize into resource note
/kb-obsidian-write       # (optional) generate prose from it
```

**Health check and maintain**
```
/kb-obsidian-lint        # identify orphans, clusters, stale candidates
/kb-obsidian-archive     # remediate stale-candidate findings
```

**Write from knowledge**
```
/kb-obsidian-search      # locate the relevant zettels
/kb-obsidian-write --template "blog post"   # generate structured prose
/doc-writer              # (optional) refactor into personal voice
```

### Skill Family Reference

| Skill | One-liner | Read-only? | Usage score |
|---|---|---|---|
| `kb-obsidian-remember` | Raw intake | No | No effect |
| `kb-obsidian-zettelize` | Atomize sources into zettels | No | Increments `total_access` on dedup scan + `use_count` on merge/update |
| `kb-obsidian-assemble` | Curate zettels into resource notes | No | Increments `total_access` + `use_count` on inclusion |
| `kb-obsidian-write` | Generate prose from vault knowledge | No | Increments `total_access` + `use_count` on citation |
| `kb-obsidian-search` | Retrieve zettels by query | Yes | Increments `total_access` |
| `kb-obsidian-lint` | Vault health checks (5 checks) | Yes | Increments `total_access` |
| `kb-obsidian-log` | Append-only operations log | No (appends only) | No effect |
| `kb-obsidian-archive` | Mark notes archived/superseded/deprecated | No | Increments `total_access` |
| `kb-obsidian-config` | Graph view configuration | No | No effect |

Vault path resolution is always sourced from `system.md`. See individual `skills/kb-obsidian-{name}/SKILL.md`
files for full input contracts, step-by-step behavior, and hard rules.

---

## Memory System

Context that compounds. Knowledge wrappers that survive sessions.

The memory system is a dedicated layer on top of the Obsidian vault. It does
not replace zettels — it wraps them. A memory note is a lightweight operational
record that links one or more canonical zettels with recall metadata: when to
surface it, who it belongs to, how confident the system is in it, and how many
times it has been recalled. The canonical facts live in the zettels. The memory
note is the address book entry that makes those facts findable at the right
moment.

### Architecture

Two paths. One automatic, one explicit.

**Read is autonomous.** When Kakugyō detects project-continuity or
memory-aware signals in a request, it may emit an `S0.recall` step as the first
entry in the plan. Fuhyō executes it via `kb-memory-recall`, and the returned
`context_packet` rides back to Kakugyō through Ōshō's relay on the next
continuation call. No user action required.

**Write is gated.** Memory notes are created by explicit user triggers only.
The canonical trigger is "remember this." Kakugyō decomposes it into the
crystallization recipe (see Orchestration Integration below) and every write
requires per-file approval before the note is committed.

Memory notes are **one-way wrappers**: they link canonical zettels; zettels
never link back to memory notes. Canonical truth stays in the zettel layer.
Memory notes carry operational context, not new facts.

```
memory note → [[canonical zettel]]   ← one-way only
```

Folder placement follows scope:

```
memory_root/
  shared/                  ← Scope: shared (any agent, any project)
  agents/{agent-id}/       ← Scope: agent (single agent only)
  projects/{project-id}/   ← Scope: project (single project only)
```

### The Three Skills

| Skill | One-liner | Read-only? |
|---|---|---|
| `kb-memory-recall` | Retrieve memory notes by agent/project/intent, hydrate linked zettels, return a bounded `context_packet` | Yes (counters only) |
| `kb-memory-enrich` | Create or update memory wrappers with tier, scope, recall conditions, and confidence metadata | No |
| `kb-memory-decay` | Read-only health scan: 7 canonical checks for expired, stale, low-confidence, overdue, and orphaned memory notes | Yes (read-only) |

`kb-memory-recall` is the read primitive. It scores candidates against the
query, partitions results into `must_load`, `maybe_load`, and
`stale_or_risky`, hydrates their canonical zettels, and falls back to
`kb-obsidian-search` if no memory wrapper exists for the domain.

`kb-memory-enrich` is the write primitive. It enforces the tier-promotion
eligibility rules, runs a dedup pass before creation, places files by scope,
and requires explicit user approval before every write. Librarian, not writer:
it never invents facts; every claim in a memory note must trace back to a
linked zettel.

`kb-memory-decay` never mutates anything. It surfaces what has gone stale,
what is ready for promotion, and what canonical sources are no longer active —
then stops. All remediation decisions belong to the user.

### Orchestration Integration

S0.recall may fire when Kakugyō detects project-continuity signals
(references to an existing project, "same pattern as before," "as we decided,"
vault/zettel/memory references). It is the first step in the plan, always
delegated to Fuhyō:

```
S0.recall   fuhyo   null   "kb-memory-recall vault-id '{task}' --agent {a} --project {p}"   []
```

The `context_packet` returned is advisory. It biases downstream planning; it
does not override agent contracts or system hard rules.

Memory creation follows the **crystallization recipe** — a Kakugyō canonical
plan pattern triggered when the user says "remember this":

```
S-pre   fuhyo   null   "kb-obsidian-remember on raw source / inline content"          []
S-zet   fuhyo   null   "kb-obsidian-zettelize on the raw note path"                   [S-pre]
S-mem   fuhyo   null   "kb-memory-enrich --new wrapping the produced zettels"         [S-zet]
```

Each step carries its own approval gate. If the source is already a zettel,
drop S-pre and S-zet. Memory rides the existing obsidian pipeline — it does
not add a parallel track, it extends the end of the same one.

Ōshō relays `context_packet` to Kakugyō via `last_step_inline_result` on the
next continuation call. She does not interpret or filter it. If
`promotion_candidates[]` is non-empty, she surfaces them as a side note.
Promotion is never planned autonomously.

### Practical Usage

**Create a memory note**

Say "remember this" — with inline content, a zettel path, or after a session
decision. Kakugyō decomposes it into the crystallization recipe above.

```
User: "remember this decision about the retry strategy"
→ Kakugyō emits S-pre / S-zet / S-mem crystallization plan
→ Fuhyō executes each step with approval gates
→ Memory note created, linked to the produced zettel
```

**Recall happens automatically**

For project-aware requests, S0.recall fires before planning. No invocation
needed. The relevant memory notes surface as part of Kakugyō's context.

**Recall manually for a specific task**

```
/kb-memory-recall "{task}"
/kb-memory-recall {vault-id} "{task}" --agent hisha --project otsumi-agentic --intent documentation
```

**Run a health scan**

```
/kb-memory-decay                              # scan all memory notes
/kb-memory-decay {vault-id} --tier stm        # stm notes only
/kb-memory-decay {vault-id} --agent hisha     # agent-scoped only
```

**Promote a memory note after decay surfaces it as eligible**

```
/kb-memory-enrich {vault-id} --target {path}  # propose tier change in diff; approve
```

### Guardrails

Memory notes have an enforced mutability model. The system encodes it; these
are not guidelines.

- **Do not manually edit memory note frontmatter.** Title, Creation Date,
  Author, Template, and Lang are immutable. Tier, Scope, Status, Confidence, Stability, Source Quality, and Superseded By are
  conditionally mutable — changes require enrich with a diff and explicit
  approval. See system.md § Mutability Policy for the full list.
- **Do not bypass enrich to write memory notes.** Enrich enforces dedup, scope
  placement, enum validation, and tier-promotion eligibility. Bypassing it
  produces notes the system cannot guarantee it can parse.
- **Memory notes are wrappers, not replacements.** Facts belong in zettels.
  A memory note that claims facts not present in its linked canonical sources
  violates the librarian contract. `kb-memory-enrich` refuses to write such
  notes.
- **Do not create a memory note without a canonical zettel source.** Enrich
  verifies that every wikilink in `Canonical Sources` resolves to an existing
  zettel before writing. If the source does not exist yet, run
  `kb-obsidian-zettelize` first.
- **Memory is advisory bias.** `operational_notes` in a memory note cannot
  override agent hard rules, the orchestration chain, or system-level
  constraints. They inform — they do not instruct.

### Tier and Scope Systems

**Tiers** are stored in frontmatter (`Tier: stm | mtm | ltm`), not in folder
structure. They represent confidence in the memory's longevity and reliability.

| Tier | Meaning | Default lifetime |
|---|---|---|
| `stm` | Short-term memory — session or task-specific | Review after 7d; auto-stale at 14d |
| `mtm` | Medium-term memory — project or phase-specific | Review after 30d; auto-stale at 90d |
| `ltm` | Long-term memory — stable, cross-project knowledge | Review after 180d |

**Tier promotion** requires meeting eligibility thresholds (enforced by enrich;
decay surfaces candidates):

- `stm → mtm`: `Recall Count ≥ 5` AND `Confidence ≥ 0.7`
- `mtm → ltm`: `Recall Count ≥ 10` AND `Confidence ≥ 0.85` AND
  `Source Quality ∈ {direct_user_decision, project_artifact}`

All promotions require explicit user approval and a per-file diff. Demotion
is also explicit — never automatic. `--force-promote` bypasses eligibility
but requires a one-line justification written into the note as an audit footer.

**Scope** determines folder placement and recall visibility:

| Scope | Folder | Recalled when |
|---|---|---|
| `shared` | `memory_root/shared/` | Always included in any recall |
| `agent` | `memory_root/agents/{agent-id}/` | Included when `--agent` matches |
| `project` | `memory_root/projects/{project-id}/` | Included when `--project` matches |

Scope defaults to `agent` on creation. Multi-project memories auto-elevate to
`shared`. Scope migration is a physical file move with an audit footer — enrich
handles it.

See `skills/kb-memory-recall/SKILL.md`, `skills/kb-memory-enrich/SKILL.md`,
and `skills/kb-memory-decay/SKILL.md` for full input contracts, scoring
formulas, step-by-step behavior, and hard rules.

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

## Why This Exists

Most AI coding tools solve the wrong problem. They make *writing* code faster. They do nothing for the
discipline around it.

Where is the spec? Where are the test scenarios? Who decided on this architecture, and why? What did the
quality check actually find?

Otsumi answers those questions by making them unavoidable. Every feature delivered through the BDD workflow
leaves behind:

- a Gherkin spec that survives the next sprint
- adversarial traps caught before they cost anything
- RED tests proving the bug existed
- minimal implementation that GREEN-fixed it
- ADR/TDR records of every load-bearing decision
- evidence-backed quality scores with blocking thresholds
- human-facing docs generated from artifacts, not summaries

You choose what you own. You choose what the workflow owns. The workflow never pretends a stage was completed
when it wasn't.

---

## Design Principles

**Roles do not blur.** Ōshō is the only mouth, Kakugyō plans but never executes, specialists own one
concern. The architecture refuses scope creep at the agent level.

**Drift guardrails are explicit.** Every agent file lists what to do when it starts behaving like a different
agent: route the work to the agent that actually owns that concern.

**No hidden state.** Every stage writes a JSON artifact, every event appends to an immutable log. Recovery,
replay, and audit are first-class.

**Plain Markdown over framework.** A multi-agent system shouldn't require an SDK. It should require a
directory of `.md` files and an AI CLI that can read them.

**Skills are weapons. Agents are trigger fingers. Ōshō is fire control.**

---

*Signal over noise. Always.*

**Written by the Hand of Otsumi and the Eyes of Kakudou.**
