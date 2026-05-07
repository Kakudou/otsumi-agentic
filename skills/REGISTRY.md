# Skills Registry

This registry is the canonical index of active skills.

Rules:

- Skill folder names are lowercase ASCII and prefixed by domain.
- No deprecated, legacy, or compatibility-wrapper skills are listed here.
- Skills must stay portable: do not reference a specific agent persona, project mascot, or private orchestration name. The single explicit exception is `agent-load-persona`, which IS the project voice manual and must be forked or rewritten when adapting Otsumi to a different identity.
- Skills may reference other skills by their public skill name.
- Workflow selection belongs to the caller or host workflow; the skill only defines reusable capability behavior.

## Domains

| Domain | Meaning |
| --- | --- |
| `agent` | Prompt refinement, retrospective prompt repair, and skill-generation support. |
| `core` | Generic repository/workflow utilities such as logs, status, backlog, and command recovery. |
| `flow` | Workflow lifecycle controls such as start, continue, complete, abort, replay, and diff. |
| `dev` | Language-agnostic software delivery, BDD workflow, tests, reviews, and development support. |
| `dev-python` | Python-specific implementation, test generation, and refactoring. |
| `design` | Visual/frontend design guidance and generation. |
| `doc` | Documentation, editorial refactor, visual documentation, and decision records. |
| `git` | Local git hygiene, atomic commits, and commit-message discipline. |
| `kb` | Personal knowledge base / second-brain tooling (Obsidian vaults and similar) that resolves vault identity from `system.md` instead of hardcoding paths. |

## Active Skills

| Skill | Domain | Status | Purpose | Companion skills |
| --- | --- | --- | --- | --- |
| `agent-create-skill` | agent | active | Create or improve a skill from grounded friction by first running retrospective prompt analysis, then prompt optimization, then producing a portable `SKILL.md`. | `agent-retro-prompt`, `prompt-master` |
| `agent-load-persona` | agent | active (project-specific) | Load the Ōshō voice manual into context on session start. Project-specific by design — fork or rewrite when adapting Otsumi to a different identity. | none |
| `prompt-master` | agent | active | Refine raw user intent into a clearer, tool-targeted, execution-ready prompt while preserving the original request. | none |
| `agent-retro-prompt` | agent | active | Analyze a failed, awkward, or inefficient interaction and produce a better prompt grounded in what actually happened. | none |
| `core-atomic-log` | core | active | Append structured events to an append-only workflow log without corrupting existing history. | none |
| `core-backlog` | core | active | Read and manage planned feature or task records without executing them. | none |
| `core-fix-command` | core | active | Recover or map command/skill names when a user or workflow invokes an outdated or incorrect command. | none |
| `core-plan-lint` | core | active | Statically validate a Kakugyō orchestration plan against the atomic-or-swarm mandate, atomicity smell tests, dependency semantics, parallel_group safety, blocker vocabulary, and shogi-roster discipline. Returns pass/fail with structured violations and a suggested decomposition. | none |
| `core-status` | core | active | Report current workflow or repository state from available state artifacts without mutating anything. | none |
| `design-frontend` | design | active | Produce distinctive frontend design direction and implementation guidance while avoiding generic AI-looking layouts, palettes, and typography. | none |
| `dev-bdd-gherkin` | dev | active | Produce Gherkin-style behavior specifications and trap analysis while keeping domain behavior separate from implementation details. | `prompt-master` |
| `dev-bdd-workflow` | dev | active | Define the reusable BDD delivery workflow: feature scope, constraints/traps, RED tests, minimal implementation, GREEN verification, refactor, decisions, quality, and documentation. | `flow-start-pipeline`, `dev-bdd-gherkin`, `dev-run-tests`, `dev-quality-check`, `dev-quality-score` |
| `dev-delivery-review` | dev | active | Review a completed development delivery for architecture, tests, maintainability, docs readiness, and risk before final acceptance. | `dev-quality-check`, `dev-expert-code-review`, `dev-quality-score` |
| `dev-env-setup` | dev | active | Detect and prepare the development environment while preserving existing project tooling and requiring approval for invasive config changes. | none |
| `dev-expert-code-review` | dev | active | Perform a deep code review focused on correctness, architecture, maintainability, edge cases, and production risk. | `dev-quality-check` |
| `dev-github-issue-safe-fix` | dev | active | Resolve a GitHub issue by ID through a mandatory BDD-safe flow: issue verification, test-suite scan, prompt refinement, user-selected pipeline mode, stage 3/4/5 minimum, and no commit or push. | `prompt-master`, `flow-start-pipeline`, `dev-bdd-workflow`, `dev-run-tests` |
| `dev-opencti-create-connector` | dev | active | Scaffold a production-grade OpenCTI external-import connector through an interactive wizard covering API details, Pydantic models, STIX mapping, settings, state, tests, docs, validation, and optional local commits. | `git-commits`, `dev-run-tests`, `dev-quality-check` |
| `dev-quality-check` | dev | active | Run or describe development quality checks such as linting, formatting, import hygiene, typing, and static quality gates. | `dev-env-setup` |
| `dev-quality-score` | dev | active | Score outputs against explicit criteria and thresholds, identify blockers, and return a pass/fail/remediation verdict. | none |
| `dev-retro-feature` | dev | active | Reverse-engineer existing code behavior into feature candidates, gap reports, and specification drafts without mutating source code. | `dev-bdd-gherkin` |
| `dev-run-tests` | dev | active | Execute or define the project test command, report collected/passed/failed/error counts, and classify RED/GREEN status. | `dev-env-setup` |
| `dev-stage-router` | dev | active | Route development workflow stages to language-specific implementation, test, or refactor skills when a host workflow still requires stage routing. | `dev-python-test-generator`, `dev-python-implementer`, `dev-python-refactorer` |
| `dev-python-implementer` | dev-python | active | Produce minimal Python implementation changes required to satisfy existing failing tests while preserving project conventions. | `dev-run-tests` |
| `dev-python-refactorer` | dev-python | active | Refactor Python code in small behavior-preserving steps, with tests remaining GREEN after each accepted change. | `dev-run-tests`, `dev-quality-check` |
| `dev-python-test-generator` | dev-python | active | Generate Python tests matching the existing test-suite convention, including pytest-bdd or raw `_given_/_when_/_then_` pytest patterns. | `dev-run-tests` |
| `doc-decision-record` | doc | active | Create ADR/TDR-style decision records when meaningful technical or design decisions were made, and explicitly record when no decision is needed. | none |
| `doc-editorial-refactor` | doc | active | Refactor Markdown or prose documents for sharper structure, TL;DR clarity, flow, and voice while preserving protected blocks and factual meaning. | none |
| `doc-stylish-excalidraw` | doc | active | Create stylish visual documentation with Excalidraw-oriented layout logic, diagram classification, bound-arrow semantics, and validation. | `doc-writer` |
| `doc-writer` | doc | active | Generate user-facing documentation from verified artifacts, implementation behavior, or structured source material without inventing unsupported behavior. | `doc-decision-record` |
| `flow-abort` | flow | active | Abort an active workflow safely while preserving artifacts and recording the reason. | `core-atomic-log` |
| `flow-complete-stage` | flow | active | Finalize a workflow stage, update state, record outputs, and determine the next stage or pause condition. | `core-atomic-log`, `core-status` |
| `flow-continue` | flow | active | Resume a paused workflow from its recorded state and validate any user-owned work before continuing. | `core-status`, `flow-complete-stage` |
| `flow-diff-stage` | flow | active | Compare stage artifacts or workflow outputs to show what changed between stages or runs. | `core-status` |
| `flow-replay` | flow | active | Replay a closed or aborted workflow from a chosen stage while preserving upstream artifacts and requiring confirmation before overwriting downstream outputs. | `core-status`, `flow-complete-stage` |
| `flow-start-pipeline` | flow | active | Initialize a staged workflow from refined input, selected mode, language/domain hints, and requested stage set. | `prompt-master`, `core-atomic-log`, `core-status` |
| `git-commits` | git | active | Create local-only atomic commits with disciplined gitmoji/conventional messages, no push, no co-author trailer, and fixup/save-commit behavior when appropriate. | none |
| `kb-obsidian-config` | kb | active | Configure an Obsidian vault's graph view (`graph.json`) — color groups, search lens, display fields, and forces — by resolving the vault path from `system.md` → `## Knowledge Bases` instead of hardcoding it. | none |
| `kb-obsidian-remember` | kb | active | Capture raw content into a vault's `raw_root` byte-for-byte unchanged, with an epoch-timestamped filename for uniqueness. No frontmatter, no formatting, no transformation. | `kb-obsidian-zettelize` |
| `kb-obsidian-zettelize` | kb | active | Atomize an arbitrary source (raw note, URL, PDF, markdown, plaintext) into atomic Zettelkasten notes in a vault's `zettel_root`, deduping against existing zettels by title / tags / content and updating instead of duplicating. | `kb-obsidian-remember`, `kb-obsidian-assemble` |
| `kb-obsidian-assemble` | kb | active | Synthesize a topical resource note in a vault's `resources_root` by linking and embedding existing zettels only. Never invents zettel-grade content; surfaces gaps to the user instead. | `kb-obsidian-zettelize` |
