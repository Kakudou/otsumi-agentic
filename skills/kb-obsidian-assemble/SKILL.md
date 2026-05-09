---
name: kb-obsidian-assemble
description: "Synthesize a resource note in the vault's resources_root by linking and embedding existing zettels on a given topic. NEVER creates new zettel-grade content — only curates, links, and embeds. Surfaces gaps to the user instead of inventing."
interaction_model: multi-turn
---

# KB Obsidian Assemble

Curate existing zettels into a topical resource note in a vault's `resources_root`.

This skill is a **librarian, not a writer**:
- it links, embeds, and structures existing zettels,
- it never invents zettel-grade content,
- it surfaces gaps and stops when coverage is insufficient.

## Operating Intent

- Primary objective: assemble a resource note from existing zettels only.
- Zero invention policy: no new factual prose outside source zettels.
- User-governed gaps: non-empty gap reports require explicit user decision (unless `--strict`).

## Usage

- `/kb-obsidian-assemble "{topic}"` — default vault. Topic is the subject of the resource note (e.g. `"Indian Attack chess opening"`).
- `/kb-obsidian-assemble {vault-id} "{topic}"` — scope to a specific vault entry.
- `/kb-obsidian-assemble {vault-id} "{topic}" --tags "tag/a,tag/b"` — restrict the zettel search to specific tags.
- `/kb-obsidian-assemble {vault-id} "{topic}" --lang {EN|FR|...}` — restrict to zettels matching a `Lang` value.
- `/kb-obsidian-assemble {vault-id} "{topic}" --strict` — only proceed if the gap report is empty; abort otherwise instead of asking.
- `/kb-obsidian-assemble {vault-id} "{topic}" --dry-run` — produce the assembly proposal without writing.

## Hard Rules

- MUST resolve `zettel_root`, `resources_root`, `zettel_template`, and `resource_template` from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any of those paths.
- MUST treat existing zettels as the **only** source of truth. The body of the resource note is built from wikilinks (`[[zettel]]`), embeds (`![[zettel]]` or `![[zettel#section]]`), and short connective glue ONLY.
- MUST NEVER write factual / claim-bearing prose that does not exist in some zettel. Connective glue is permitted ONLY for: section headings, transition sentences that name what the next zettel covers, table-of-contents framing, and citation pointers. If you cannot write a sentence without inventing a fact, do not write it.
- MUST NEVER create new zettels from this skill, even when gaps exist. Filling gaps is the user's call (typically by running `kb-obsidian-zettelize` separately on a fresh source).
- MUST search `zettel_root` by:
  - `Title` and `Aliases` substring / fuzzy match against the topic
  - tag intersection with topic-derived tag candidates and any `--tags` filter
  - body content overlap with the topic
- MUST present the discovered zettel set BEFORE writing, with per-zettel reasoning for inclusion.
- MUST produce a **gap report** identifying what subtopics the resource note's natural structure would cover but no zettel addresses. The report MUST list each gap explicitly.
- MUST require explicit user decision when the gap report is non-empty:
  - `proceed` — assemble with current zettels; gaps surface as `> [!TODO]` callouts in the resource note.
  - `abort` — write nothing.
  - `zettelize first` — return early with a recommendation to run `kb-obsidian-zettelize` against a specified source.
  - `--strict` mode SHORT-CIRCUITS this prompt: any gap → abort.
- MUST honor `resource_template`. If it is `null`, derive frontmatter from `zettel_template` with `Template: Resource`, AND surface the missing-template gap to the user.
- MUST honor every safety caveat declared on the vault entry. If the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally.
- MUST NEVER touch `Creation Date`, `Title`, `Author`, `Template`, or `Lang` of an existing resource note unless the user explicitly approves the change.
- MUST NEVER remove an existing section that the user wrote between runs.
- MUST render unanswered gaps as Obsidian `> [!TODO]` callout blocks. NEVER render gaps as plain prose sentences.
- MUST ensure the `## Sources` footer and the `Links:` frontmatter list reference the same zettel set — no extras in either direction.
- NEVER mutate any zettel. This skill is read-only against `zettel_root`.
- NEVER overwrite an existing resource note without an explicit diff approval.
- NEVER `git add`, `git commit`, or `git push`.
- NEVER pad the body with paraphrased zettel content. If a zettel's content is needed inline, EMBED it (`![[zettel]]`), do not retype it.

## Steps

### 1. Resolve target vault

1. Read `system.md` → `## Knowledge Bases` → vault entry (by id or default).
2. Extract `zettel_root`, `resources_root`, `zettel_template`, `resource_template`, and any safety caveats.
3. If `resource_template` is `null`, set a flag `derived_resource_frontmatter = true` and plan to surface this in the final report.
4. Read `zettel_template` (and `resource_template` if non-null) to capture the canonical frontmatter shape.

### 2. Discover candidate zettels

1. List `zettel_root`.
2. Derive topic-candidate tags from the `{topic}` (lowercase, hierarchical where natural). Combine with any `--tags` filter.
3. Score every zettel against:
   - Title / Aliases match
   - tag intersection
   - body overlap with the topic
   - `Lang` filter (if `--lang`)
4. For each zettel read during the discovery scan, increment its `total_access` frontmatter field by 1. This records that the zettel was accessed as part of candidate discovery.
5. Build `candidate_set` ordered by relevance score with explicit per-zettel reasoning:
   ```
   - "[[Indian Attack opening principles]]"
       title_match: strong
       tags_overlap: ["chess/openings/indian-attack"]
       body_overlap: high
   - "[[King-side fianchetto motif]]"
       title_match: partial
       tags_overlap: ["chess/openings/indian-attack", "chess/motifs"]
       body_overlap: medium
   ```

### 3. Plan resource structure

Draft a section outline for the resource note grounded in the topic.
For each planned section, map zero-or-more zettels from `candidate_set` to it.

Sections with zero mapped zettels are **gaps**.

### 4. Gap report

Present:

- the proposed outline
- per-section zettel mapping
- the **gap list** with each gap's intended scope

Then prompt for `proceed` / `abort` / `zettelize first` (or short-circuit per `--strict`).

If `zettelize first`: return a recommendation containing the gap topics and a suggested `kb-obsidian-zettelize` invocation. Stop. Write nothing.

### 5. Materialize the resource note (only on `proceed`)

1. Filename: `{Topic}.md` in `resources_root`. Replace characters Obsidian prohibits in filenames (`/`, `\`, `:`, `|`, `?`, `*`, `<`, `>`, `"`) with a hyphen or safe equivalent. Disclose any substitution to the user. If a same-named resource exists, switch to **update** flow (Step 6) instead of overwriting.
2. Frontmatter:
   - if `resource_template` exists → start from it.
   - else → derive from `zettel_template` with:
     - `Title`: `{Topic}`
     - `Aliases`: optional aliases the user supplied
     - `Author`: vault default
     - `tags`: union of distinguishing tags from the included zettels (deduped, lowercase)
     - `Links`: YAML list of wikilinks to every included zettel
     - `Lang`: `--lang` value or vault default
     - `Template`: `Resource`
     - `Creation Date` / `Modification Date`: now in `YYYY/MM/DD HH:mm:ss`.
     - `total_access`: `0`
     - `use_count`: `0`
3. Body — strict assembly grammar:
    - `# {Topic}` heading
    - optional one-paragraph framing sentence that NAMES what the resource collects (e.g. "Curated zettels covering the Indian Attack opening, its principles, and key motifs.") — no factual claims that are not in zettels.
   - per outline section:
     - `## {Section title}`
     - one optional connective sentence that names what the next zettel block covers (no claims).
     - mapped zettels rendered as either:
       - **link list**: `- [[zettel]] — {short Alias from the zettel itself, NEVER a paraphrase}`
       - **inline embed**: `![[zettel]]` for short zettels whose full body is the section.
       - **section embed**: `![[zettel#Heading]]` to pull a single section.
   - gap sections (if user chose `proceed` despite gaps):
     ```
     > [!TODO] Gap: {gap topic}
     > No zettel covers this yet. Run `/kb-obsidian-zettelize` against a relevant source.
     ```
4. Footer:
   - `## Sources` listing every included zettel as wikilinks (mirrors `Links:` in machine-readable form).
5. For every zettel included in the resource note (linked or embedded), increment its `use_count` frontmatter field by 1. This records productive use: the zettel's knowledge was curated into a resource.

### 6. Update flow (when a same-named resource already exists)

1. Read the existing resource note.
2. Compute the minimal diff:
   - new included zettels → append to `Links:` and to the relevant outline sections.
   - new tags → append.
   - update `Modification Date`.
   - NEVER touch `Creation Date`, `Title`, `Author`, `Template`, or `Lang` unless explicitly approved.
   - NEVER remove an existing section that the user wrote between runs.
3. Show diff. Get explicit approval. Write atomically.

### 7. Approval and write

- Default: present the full proposed resource note for `approve` / `edit` / `abort` BEFORE writing.
- `--dry-run`: stop here; emit the proposal as the return value.

Write atomically (temp + rename in `resources_root`).

### 8. Report

Return:

- resource path written (or proposal in dry-run)
- list of included zettels with role (link / embed / section embed)
- gap list (resolved or surfaced as TODO callouts)
- `derived_resource_frontmatter` warning if `resource_template` was `null`
- if the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally
- reminder that no git operation was performed
- reminder that no zettels were mutated

## Validation

Before claiming success:

- [ ] `zettel_root`, `resources_root`, `zettel_template`, `resource_template` all came from `system.md`.
- [ ] Every body sentence is either: a section heading, a connective sentence that names without claiming, a wikilink, an embed, or a TODO callout.
- [ ] No factual / claim-bearing prose was generated. No paraphrased zettel content was inlined.
- [ ] `Sources` footer matches `Links:` frontmatter (same set, no extras).
- [ ] No zettel was modified.
- [ ] Gap report was presented; user choice (`proceed` / `abort` / `zettelize first`) was honored.
- [ ] If `resource_template` was `null`, the missing-template gap was surfaced.
- [ ] No git operation was performed.
- [ ] Submodule writes were disclosed.
- [ ] For every zettel read during candidate discovery (Step 2), `total_access` was incremented.
- [ ] For every zettel included in the resource note (Step 5), `use_count` was incremented.
