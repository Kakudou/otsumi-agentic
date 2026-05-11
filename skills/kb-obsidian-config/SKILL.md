---
name: kb-obsidian-config
description: "Configure an Obsidian vault's graph view (graph.json) — color groups, search lens, display, and forces — by resolving the vault path from system context instead of hardcoding it."
interaction_model: multi-turn
---

# KB Obsidian Config

Configure the Obsidian graph view for a vault registered under `## Knowledge Bases` in `system.md` without hand-editing JSON or leaking vault paths into the skill body.

## Usage

- `/kb-obsidian-config` — interactive: ask which axis to configure (color groups, search lens, display, forces) for the default vault registered in `system.md`.
- `/kb-obsidian-config {vault-id}` — same, scoped to the named vault entry.
- `/kb-obsidian-config {vault-id} colors` — only edit `colorGroups`.
- `/kb-obsidian-config {vault-id} search "{query}"` — set the active discovery search.
- `/kb-obsidian-config {vault-id} reset-search` — clear the active search.
- `/kb-obsidian-config {vault-id} forces` — only tune the physics block.
- `/kb-obsidian-config {vault-id} display` — toggle filter/display fields (`showTags`, `showOrphans`, `textFadeMultiplier`, ...).

## Hard Rules

- MUST resolve the vault root from system context (`## Knowledge Bases` → `{vault-id}` → `Absolute path`, injected as `custom_instruction` at session start). NEVER hardcode a vault path, vault id, or folder layout in this skill or in any generated suggestion.
- MUST refuse with an explicit error if system context has no `## Knowledge Bases` entry matching `{vault-id}`, or — when no id was given — if the section does not contain exactly one default vault marked `(default)`.
- MUST refuse with an explicit error if `<vault-config-root>/graph.json` does not exist on disk. This skill configures existing graph views; it does not bootstrap new ones.
- MUST tell the user to **close the Graph View tab in Obsidian BEFORE** writing `graph.json`. Obsidian rewrites that file continuously while the view is open and silently overwrites edits.
- MUST read the existing `<vault-config-root>/graph.json` before editing. Preserve every field outside the requested edit scope byte-identical to the pre-edit version.
- MUST present the proposed JSON diff and require explicit `approve` / `edit` / `abort` before writing.
- MUST write valid JSON. Reject anything that does not parse.
- MUST instruct the user to reopen the Graph View tab after the write completes.
- MUST honor every safety caveat declared on the vault entry (encryption, submodules, off-limits paths). If the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally.
- NEVER `git add`, `git commit`, or `git push` from this skill. Configuration changes are local-only.
- MUST consult the vault's templates folder (per system context → `Conventions source of truth`) before proposing queries targeting frontmatter fields, tag namespaces, or note types — field names MUST come from real templates, NEVER guessed.
- NEVER scan the vault for color-group or search suggestions without disclosing exactly which folders, templates, tags, or frontmatter fields you actually inspected.
- NEVER invent folders, tags, or frontmatter fields. Mark any speculative suggestion `(speculative)`.
- NEVER touch UI-state fields (`scale`, `close`, `collapse-*`) unless the user explicitly asks.

## Steps

### 1. Resolve target vault

1. Resolve from system context (`## Knowledge Bases` section, already available as `custom_instruction`).
2. If `{vault-id}` was supplied, find the matching `### \`{vault-id}\`` subsection. Otherwise pick the entry tagged `(default)`. If neither resolves cleanly, refuse and ask the user which vault to target.
3. Extract `Absolute path`, `Config root`, `Top-level structure`, and any safety caveats. These become `<vault-root>`, `<vault-config-root>`, and `<vault-structure>` for the rest of the run.
4. Verify `<vault-config-root>/graph.json` exists. If not, refuse — this skill configures, it does not bootstrap.

### 2. Pre-flight

1. Tell the user, in plain words: "Close the Obsidian Graph View tab before I write `graph.json`. It will overwrite my changes the moment it sees them."
2. Wait for explicit confirmation.
3. Read `<vault-config-root>/graph.json` and snapshot its current keys.

### 3. Mode dispatch

Resolve the operation from the invocation:

- `colors` → **Color Groups** flow (4a).
- `search "{query}"` → **Discovery Search** flow (4b).
- `reset-search` → set `search` to `""` and skip to step 5.
- `forces` → **Forces** flow (4c).
- `display` → **Display / Filters** flow (4d).
- no mode → ask which axis (or axes) to change.

### 4a. Color Groups flow

1. If the user has no concrete grouping in mind:
   1. Read the vault's templates folder (at the absolute path declared in system context under `Conventions source of truth`) to learn the canonical frontmatter keys, tag namespaces, and `Template:` values actually in use.
   2. Scan `<vault-structure>` (top-level folders, tag frequency from real frontmatter, common path prefixes, distinctive `Template:` signatures).
   3. Propose ≤ 12 groups grounded **only** in what was actually observed. Disclose every folder, template, tag, and frontmatter field you inspected.
2. For each proposed group, produce:
   - `query` — an Obsidian graph query (`path:`, `file:`, `tag:`, `OR`, `-path:`, `-tag:`).
   - `color` — one of the canonical RGB integers (see **RGB Reference**) chosen by signal:
     - warm (red / orange / gold) → high-signal, actionable, hot.
     - cool (blue / teal / cyan / gray) → reference, archive, high-volume that should not dominate.
     - same hue → conceptually paired clusters.
3. Order matters: earlier entries take precedence on overlap. Order intentionally.
4. Present the full proposed `colorGroups` array as a JSON diff against the current file. Get approval.

### 4b. Discovery Search flow

1. Treat `search` as a lens, not a permanent filter. Pick queries that answer questions a single file cannot — bridges, sources, relationships, performance, gaps.
2. If no concrete query was supplied:
   1. Read the templates folder first (at the absolute path declared in system context) to learn the frontmatter axes that real notes actually expose (e.g. `Template`, `Lang`, `Tags`, `#definition`).
   2. Scan `<vault-structure>` for folders / tag frequencies / template signatures.
   3. Propose 2–4 candidate lenses grounded in what you actually found, with each lens annotated by which template or frontmatter field it leans on.
3. Confirm a single chosen query and write it to `search`.
4. After the user has explored, offer to clear (`reset-search`) or rotate to another lens.

### 4c. Forces flow

Tunable fields:

- `centerStrength` (0–1) — pull toward center.
- `repelStrength` (1 → ~15+, can spike higher for very dense graphs) — node repulsion.
- `linkStrength` (0–1) — pull between linked nodes.
- `linkDistance` (px) — target link length.

Heuristics:

- spread-out, clearly separated clusters → high `repelStrength` + high `linkDistance`.
- organic, off-center layout → low `centerStrength`.
- tight clusters (with overlap risk) → high `linkStrength`.

Propose a delta diff, not a full rewrite. Get approval.

### 4d. Display / Filters flow

| Block | Field | Type | Effect |
|---|---|---|---|
| Filters | `search` | string | Active query (same syntax as color group queries). |
| Filters | `showTags` | bool | Render tag nodes. |
| Filters | `showAttachments` | bool | Render attachment nodes. |
| Filters | `hideUnresolved` | bool | Hide links to non-existent pages. |
| Filters | `showOrphans` | bool | Render disconnected nodes. |
| Display | `showArrow` | bool | Directional arrows on links. |
| Display | `textFadeMultiplier` | number | `0` = always show labels; higher = fade faster on zoom-out. |
| Display | `nodeSizeMultiplier` | number | Node size scale (`1` = default). |
| Display | `lineSizeMultiplier` | number | Link thickness scale (`1` = default). |

Leave UI-state fields (`collapse-filter`, `collapse-color-groups`, `collapse-display`, `collapse-forces`, `scale`, `close`) untouched unless explicitly requested.

### 5. Write and verify

1. Merge edits into the in-memory `graph.json`. Preserve every untouched key.
2. Validate as JSON. Reject if it does not parse.
3. Show the final diff. Re-confirm with the user.
4. Write `<vault-config-root>/graph.json` atomically: write to a sibling temp file in the same directory, then rename over the original.
5. Remind the user to reopen the Graph View tab.
6. Report which fields were changed, the absolute path written, and the fact that no git operation was performed.

## Reference: Query Syntax

- `path:folder/` — files in a folder (include the trailing slash).
- `file:name` — filename substring match.
- `tag:#tagname` — frontmatter or inline tag match.
- `OR` — combine queries: `tag:#hype/fire OR tag:#hype/hot`.
- `-path:folder/` — exclude a folder.
- `-tag:#noise` — exclude a tag.

## Reference: RGB

Obsidian encodes a color as a single integer: `(r << 16) | (g << 8) | b`.

| Color | Integer | Hex |
|---|---|---|
| Red | `16065571` | `0xF54C43` |
| Orange | `15962679` | `0xF39237` |
| Gold | `16104769` | `0xF5BD41` |
| Green | `4769678` | `0x48C78E` |
| Teal | `4243643` | `0x40C0BB` |
| Cyan | `6607590` | `0x64D2E6` |
| Blue | `7042559` | `0x6B7BFF` |
| Purple | `10711254` | `0xA370D6` |
| Pink | `15494055` | `0xEC6BA7` |
| Gray | `10395294` | `0x9E9E9E` |

Color group entry shape:

```json
{
  "query": "<obsidian search query>",
  "color": { "a": 1, "rgb": 7042559 }
}
```

## Design Principles for Color Grouping

- Group by the vault's **primary structural axis** — folder, type, or tag hierarchy — not by individual files.
- Warm colors → high-signal / actionable. Cool colors → reference / high-volume that should not dominate.
- Same hue for conceptually paired categories (e.g., two ingestion sources that share meaning).
- Cap at ~12 groups. Past that, color stops carrying signal.
- Earlier groups win over later ones on overlap. Order intentionally.
- Leave unmatched nodes default gray — they become whitespace that makes the colored clusters pop.

## Search Design Heuristics

A good `search` answers a question the user cannot answer by reading single files. Think in terms of:

- **Bridges** — what connects two otherwise separate clusters?
- **Sources** — where did this knowledge originate?
- **Relationships** — who or what connects to who or what?
- **Performance** — what is active vs. stale?
- **Gaps** — what is orphaned or under-connected that should not be?

Layer multiple axes with `OR`. Strip noise with `-path:` or `-tag:`.

## Validation

Before claiming success:

- [ ] System context resolved a vault entry; nothing was hardcoded in this skill or in the diff.
- [ ] If suggestions referenced frontmatter fields, tag namespaces, or note types, the relevant template(s) were actually read — disclosed in the proposal.
- [ ] Graph View was confirmed closed before writing.
- [ ] Existing `graph.json` keys outside the edit scope are byte-identical to the pre-edit snapshot.
- [ ] Final file is valid JSON.
- [ ] No git operation was performed.
- [ ] User received the "reopen Graph View" reminder.
