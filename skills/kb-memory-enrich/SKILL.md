---
name: kb-memory-enrich
description: "Create or update dedicated Memory notes that wrap canonical zettels with operational recall metadata (tier, scope, agents, projects, recall conditions). Librarian, not writer — never invents claims, never modifies zettel bodies. Enforces tier-promotion eligibility per system.md Memory Notes Mutability Policy."
interaction_model: multi-turn
---

# KB Memory Enrich

## Mission

Librarian for memory wrappers. Create or update dedicated Memory notes that link canonical zettels with operational recall metadata — tier, scope, agents, projects, recall conditions, and confidence signals. Never invents claims; never modifies zettel bodies; surfaces gaps. Enforces tier-promotion eligibility per the Memory Notes Mutability Policy. Modeled on `kb-obsidian-assemble`'s wrapper-note pattern.

## Usage

| Command | Purpose |
|---|---|
| `/kb-memory-enrich --new "{title}"` | Create a new memory note (requires `--agent` since default scope is `agent`). |
| `/kb-memory-enrich {vault-id} --new "{title}" --scope agent --agent hisha --tier mtm` | Create agent-scoped memory at mtm tier. |
| `/kb-memory-enrich {vault-id} --new "{title}" --scope shared --tier ltm` | Create shared memory at ltm tier. |
| `/kb-memory-enrich {vault-id} --new "{title}" --scope project --project otsumi-agentic` | Create project-scoped memory. |
| `/kb-memory-enrich {vault-id} --target {memory-note-path}` | Update an existing memory note. |
| `/kb-memory-enrich {vault-id} --target {memory-note-path} --dry-run` | Preview changes without writing. |
| `/kb-memory-enrich {vault-id} --target {memory-note-path} --batch-approve` | Batch-approve all changes in one confirmation. |
| `/kb-memory-enrich {vault-id} --target {memory-note-path} --force-promote` | Bypass tier-promotion eligibility (requires justification). |
| `/kb-memory-enrich {vault-id} --target {memory-note-path} --remove-canonical-source "[[wikilink]]"` | Remove an append-only canonical source (requires justification). |

## Hard Rules

1. MUST resolve `memory_root`, `memory_template`, `zettel_root`, `zettel_template` from system context (`## Knowledge Bases` section, injected as `custom_instruction` at session start). NEVER hardcode.
2. MUST read `memory_template` (at the absolute path declared in system context under Template registry) before generating any memory frontmatter.
3. MUST verify every wikilink in `Canonical Sources` resolves to an existing zettel. Refuse to create a memory note whose canonical source does not exist. (Update flows MAY accept newly-archived zettels but MUST surface the lifecycle gap.)
4. MUST place new notes by `Scope`:
   - `Scope: shared` → `memory_root/shared/`
   - `Scope: agent` → `memory_root/agents/{agent-id}/` (single-agent only; `Agents[]` MUST contain exactly that agent-id)
   - `Scope: project` → `memory_root/projects/{project-id}/` (single-project only; `Projects[]` MUST contain exactly that project-id)
5. MUST refuse ambiguous scope: `--scope agent` without `--agent`, `--scope project` without `--project`, multiple agents in `Agents[]` with `Scope: agent`, multiple projects in `Projects[]` with `Scope: project`.
6. MUST auto-elevate scope: if `Projects: [a, b]` (multiple projects), `Scope` MUST be `shared` and folder placement MUST be `memory_root/shared/`. Surface the auto-elevation in the diff.
7. MUST default `Scope` to `agent` on creation when omitted, requiring `--agent`.
8. MUST dedup pass against existing memory notes BEFORE creating: search by title fuzzy, tags, recall_summary semantic overlap. Present overlap report; user chooses `merge into existing` / `update existing` / `create new` / `skip`.
9. MUST validate enums: `Status ∈ {active, stale, superseded, archived}`, `Tier ∈ {stm, mtm, ltm}`, `Scope ∈ {shared, agent, project}`, `Stability ∈ {volatile, active, stable}`, `Source Quality ∈ {direct_user_decision, project_artifact, derived_summary, weak_signal}`, `Confidence ∈ [0.0, 1.0]`.
10. MUST enforce tier-promotion eligibility on update flows. If user proposes `stm → mtm` without `Recall Count ≥ 5` AND `Confidence ≥ 0.7`: refuse with `validation.enum_errors` AND `promotion_eligibility.rules_failed[]`. If user proposes `mtm → ltm` without `Recall Count ≥ 10` AND `Confidence ≥ 0.85` AND `Source Quality ∈ {direct_user_decision, project_artifact}`: refuse.
11. `--force-promote` MAY bypass eligibility but MUST require a one-line justification (read interactively from the user); the justification is appended as an audit footer in the memory note's body:
    ```
    > [!NOTE] Force promotion {YYYY/MM/DD HH:mm:ss} by user
    > Justification: {one-line text}
    ```
12. MUST handle scope migration (e.g., `agent → shared`) as a physical file move: compute new path; show both folder move AND frontmatter diff; on approve, write new path atomically, delete old path, append audit footer to the body recording the move:
    ```
    > [!NOTE] Scope migration {YYYY/MM/DD HH:mm:ss} by user
    > From: {old_scope} ({old_agents_or_projects}) → {new_scope}
    ```
    Mirror `kb-obsidian-archive --move` semantics.
13. MUST suggest companion field changes on tier transitions per the `Tier ↔ Companion Field Defaults` table. Suggestions are part of the diff; user can edit before approval.
14. `--remove-canonical-source "[[wikilink]]"` is the explicit escape hatch for removing an append-only `Canonical Sources` entry. MUST require a one-line justification (audit footer):
    ```
    > [!NOTE] Canonical source removal {YYYY/MM/DD HH:mm:ss} by user
    > Removed: [[{wikilink}]]
    > Justification: {one-line text}
    ```
    MUST refuse if removal would leave `Canonical Sources` empty (memory wrappers must wrap something).
15. MUST warn (do not block) when `Tier: ltm` AND `Confidence < 0.8`.
16. MUST warn (do not block) when `Status: superseded` AND `Superseded By` is empty.
17. MUST update `Modification Date` on every write.
18. MUST emit submodule disclosure when target path resolves under a vault submodule.
19. MUST increment `total_access` on every linked canonical zettel read during validation.
20. MUST increment `use_count` on every linked canonical zettel (productive use: zettel was wrapped into a memory).
21. MUST present per-file diff and require explicit approval before writing.
22. MUST write atomically (temp + rename in same directory).
23. MUST handle filename collision: `{YYYYMMDD}-{slug}.md` collides → append `-2`, `-3`, ... until free.
24. MUST handle missing `memory_template`: refuse creation with explicit error (`memory_template_missing`); user must create the template first.
25. MUST handle missing `memory_root`: refuse with explicit error.
26. MUST NEVER modify zettel bodies or zettel structural metadata beyond counter increments.
27. MUST NEVER create memory notes containing factual claims absent from linked canonical sources. `Recall Summary` and `Operational Notes` derive from / point at zettel content; never invented.
28. MUST NEVER `git add`, `git commit`, or `git push`.
29. In `--target` update mode, MUST refuse malformed/unparseable YAML frontmatter with `contract_violation: malformed_yaml at {path}`. MUST NOT attempt repair. MUST NOT silently skip the note. MUST surface `{path}` and the parsing error (line if possible) to the caller.
30. When creating a new memory note that wraps a zettel whose frontmatter `Status` is `archived`, MUST default the new memory note to `Status: stale` (not `active`). This marks the canonical source as no longer current. User MAY later promote to `active` via an explicit enrich call if still valid.
31. On write failure (disk full, permission error, or I/O error), MUST keep any created temp file on disk for inspection, MUST NOT leave a partial/corrupt memory note at the target path, MUST surface the error to the caller with the exact target path and OS error, and MUST report `files_modified: 0` in the output contract.
32. Concurrent modification (two parallel enrich calls targeting the same memory note) is a known v1 limitation: no file-level locking mechanism exists; last-write-wins semantics apply and overlapping writes may lose data; surface this as a known limitation in documentation, not as a runtime error.

## Steps

1. Resolve target vault and templates from system context (`## Knowledge Bases` section, already available as `custom_instruction`).
2. Determine mode (`--new`, `--target`, `--remove-canonical-source`).
3. For `--new`:
   a. Validate scope flags. Refuse ambiguous scope.
   b. Verify every `Canonical Sources` wikilink resolves to an existing zettel.
   c. Build candidate frontmatter from `memory_template`. Compute filename: `{YYYYMMDD}-{slug}.md` (handle collision with `-N` suffix).
   d. Compute target folder per `Scope`. Auto-elevate to shared if multi-project.
4. Dedup pass against `memory_root`: search by title fuzzy, tags overlap, recall_summary semantic overlap. Present overlap report.
5. Validate enums and emit warnings (ltm/confidence, superseded/empty `Superseded By`).
6. For `--target`:
   a. Read existing note; capture `Modification Date` and `Recall Count` snapshot.
   b. Compute proposed diff per Mutability Policy (immutable fields blocked, conditional fields require approval, append-only fields receive only additions unless `--remove-canonical-source` is set).
   c. If diff includes tier change: enforce promotion eligibility (or accept `--force-promote` with justification prompt).
   d. If diff includes scope change: compute new folder, propose move with audit footer.
   e. Suggest companion field changes per `Tier ↔ Companion Field Defaults`.
7. Increment `total_access` on every canonical zettel read. Increment `use_count` on every canonical zettel newly wrapped.
8. Show diff (and move plan if applicable). Get approval (per-file or batch when `--batch-approve`).
9. Write atomically (temp + rename in same directory). For scope migration: write to new path, then delete old path.
10. Emit submodule disclosure. Report changed/created paths and audit footers written.

## Output Contract

```yaml
memory_enrichment:
  mode: create|update|skip|move
  target:
  proposed_frontmatter:
  proposed_body:
  diff:
  proposed_move:
    from: null|path
    to: null|path
  validation:
    enum_errors: []
    promotion_eligibility:
      requested:
      eligible:
      rules_failed: []
    warnings: []
    canonical_sources_verified:
  dedup_overlap:
    - existing_path:
      similarity:
      recommendation:
  audit_footer_written:
    - reason:
      content:
  write_result:
    changed:
    path:
    fields_changed: []
  counter_updates:
    zettels_total_access: []
    zettels_use_count: []
  submodule_disclosure:
  requires_approval:
```

## Validation Checklist

- [ ] All paths resolved from system context.
- [ ] `memory_template` read (at absolute path from system context) before frontmatter generation.
- [ ] Every `Canonical Sources` wikilink verified to exist (creation flow).
- [ ] Scope flags validated; folder placement enforced by `Scope`.
- [ ] Multi-project memory auto-elevated to `shared`.
- [ ] Filename collision handled with `-N` suffix.
- [ ] Enum values validated; warnings emitted per ltm/confidence and superseded/empty `Superseded By`.
- [ ] Promotion eligibility enforced (or `--force-promote` justification recorded as audit footer).
- [ ] Companion field changes proposed on tier transitions.
- [ ] Scope migration treated as physical file move with audit footer.
- [ ] `--remove-canonical-source` requires justification and refuses empty `Canonical Sources`.
- [ ] Dedup pass executed; overlap report shown.
- [ ] No factual claim invented beyond what canonical zettels say.
- [ ] Per-file diff presented; explicit user approval received.
- [ ] Write performed atomically (temp + rename).
- [ ] `total_access` incremented for every canonical zettel read.
- [ ] `use_count` incremented for every canonical zettel wrapped.
- [ ] No zettel body or structural metadata modified.
- [ ] Submodule disclosure emitted if applicable.
- [ ] No git operation performed.
- [ ] If `--dry-run`: no file written; proposal returned only.
