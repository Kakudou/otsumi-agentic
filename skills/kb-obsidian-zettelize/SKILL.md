---
name: kb-obsidian-zettelize
description: "Atomize a source (raw note, URL, PDF, markdown, plaintext, anything) into atomic Zettelkasten notes in the vault's zettel_root, deduping against existing zettels and updating them when relevant instead of duplicating."
interaction_model: multi-turn
---

# KB Obsidian Zettelize

Take any source and produce a set of atomic zettels in a vault's `zettel_root`, honoring the vault's `zettel_template`, deduplicating against existing zettels by tags and title, and updating instead of duplicating when an overlapping zettel already exists.

## Usage

- `/kb-obsidian-zettelize {source}` — default vault. `{source}` is one of:
  - an absolute path inside the vault (e.g. a file under `raw_root`)
  - an absolute path on disk to a `.md`, `.txt`, or `.pdf` file
  - a URL (skill fetches and extracts the readable content)
  - inline quoted text
- `/kb-obsidian-zettelize {vault-id} {source}` — scope to a specific vault entry.
- `/kb-obsidian-zettelize {vault-id} {source} --lang {EN|FR|...}` — override the `Lang` frontmatter.
- `/kb-obsidian-zettelize {vault-id} {source} --batch-approve` — present the full set of proposed zettels in one batch instead of per-zettel approval.
- `/kb-obsidian-zettelize {vault-id} {source} --dry-run` — produce the proposal set without writing.

## Hard Rules

- MUST resolve `zettel_root`, `raw_root`, and `zettel_template` from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any of those paths.
- MUST read the actual `zettel_template` file before generating any zettel. Frontmatter keys, casing (`tags:` lowercase), `Author`, `Lang`, `Template: Zettel`, and date format (`YYYY/MM/DD HH:mm:ss`) MUST come from the template, NEVER from a guess.
- MUST also read `definition_template` when extracted concepts include term definitions — emit those as `#definition` notes per that template's shape rather than as plain zettels.
- MUST honor zettel **atomicity**: one idea per file. If a candidate note covers two distinct ideas, split it. NEVER produce a multi-idea zettel.
- MUST search `zettel_root` BEFORE creating any new zettel:
  - by **title** (fuzzy / alias overlap)
  - by **tags** (intersection of proposed tags and existing-zettel tags)
  - by **content** (semantic overlap on the candidate's core claim)
  - present every match with the overlap reasoning and let the user choose: `merge into existing` / `update existing` / `create new` / `skip`.
- MUST NEVER fabricate facts not present in the source. If the source does not state a claim, the claim does not enter a zettel. If summarization risks distortion, quote.
- MUST cite the origin in every zettel:
  - source is a vault note → wikilink in `Links:`
  - source is a local file → relative or absolute path in `Links:` and a body footer line
  - source is a URL → URL in `Links:` and a body footer line with the fetch date
- MUST present every proposed zettel (or the full batch in `--batch-approve` mode) for explicit `approve` / `edit` / `skip` / `abort` BEFORE writing.
- MUST honor every safety caveat declared on the vault entry. If the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally.
- NEVER squash multiple ideas into one zettel.
- NEVER modify the source. Raw notes stay raw. Even if the source lives in `raw_root`, this skill only reads it.
- NEVER `git add`, `git commit`, or `git push`.
- NEVER mutate an existing zettel without surfacing a diff and getting approval.
- NEVER overwrite a zettel's `Creation Date`. On update, only `Modification Date` and the touched fields/sections may change.

## Steps

### 1. Resolve target vault

1. Read `system.md` → `## Knowledge Bases` → vault entry (by id or default).
2. Extract `zettel_root`, `raw_root`, `zettel_template`, `definition_template`, and any safety caveats.
3. Read `zettel_template` and `definition_template`. Capture the canonical frontmatter keys and casing.

### 2. Ingest the source

Resolve `{source}` by shape:

- **Path inside vault** → read with the file tool; capture provenance as a vault wikilink.
- **Local file path** (`.md`, `.txt`, `.pdf`, `.html`, ...) → read; for PDFs, extract text via the available tool; capture provenance as the absolute path.
- **URL** → fetch readable content; capture provenance as the URL plus fetch ISO timestamp.
- **Inline text** → use as-is; capture provenance as `inline ({epoch})`.

If extraction yields no usable text, refuse and report.

### 3. Atomic decomposition

1. Read the source end-to-end.
2. Identify atomic claims: one self-contained idea each. Discard filler, hedging, and conversational scaffolding.
3. For each atomic claim, draft a candidate zettel:
   - `Title` — a precise, unambiguous noun phrase or claim.
   - `tags` — lowercase, hierarchical when natural (`chess/openings/indian-attack`). Reuse existing vault tags whenever possible (Step 4 will check).
   - `Aliases` — alternate phrasings the user might search for.
   - body — the claim, supporting context, quoted source where precision matters, no embellishment.
4. If a candidate looks like a *concept definition* (a term + a single sentence of meaning), route it through `definition_template` instead of `zettel_template` and tag it `definition`.

### 4. Dedup pass against `zettel_root`

For each candidate:

1. List existing files in `zettel_root`.
2. Match by:
   - filename / Title fuzzy similarity
   - Aliases overlap
   - tag intersection (heuristic: Jaccard ≥ 0.5 signals strong overlap, but lower values with title/content match still qualify)
   - body claim overlap (semantic similarity on the core claim)
3. Build an `overlap_report` per candidate:
   ```
   candidate: "Indian Attack — King-side fianchetto motif"
   matches:
     - existing: "Indian Attack opening principles"
       title_similarity: 0.62
       tag_overlap: ["chess/openings/indian-attack"]
       claim_overlap: high
       recommendation: update existing (add fianchetto motif as new section)
   ```
4. Present matches; user chooses per-candidate:
   - `merge` — fold candidate's information into the matched zettel (one idea per zettel still applies; if the matched zettel already covers a *different* atomic idea, do NOT merge — create a new linked zettel instead).
   - `update` — append a new section / link / tag to the matched zettel.
   - `create new` — proceed to step 5.
   - `skip` — drop this candidate.

### 5. Materialize zettels

For each `create new` candidate:

1. Copy `zettel_template`'s frontmatter skeleton verbatim.
2. Fill fields:
   - `Title`: candidate Title.
   - `Aliases`: list of candidate Aliases (YAML list).
   - `Author`: from vault entry's `Author / Lang defaults` (default `カクドウ ~ Kakudou`).
   - `tags`: lowercase YAML list.
   - `Links`: YAML list of:
     - vault wikilinks to existing zettels referenced
     - source provenance (wikilink / path / URL)
   - `Lang`: if `--lang` was supplied, use it instead of the vault default; otherwise use source language detection or vault default (`EN`).
   - `Template`: `Zettel` (or `Definition` for definition-shaped notes).
   - `Creation Date` and `Modification Date`: current time in `YYYY/MM/DD HH:mm:ss`.
3. Body:
   - `# {Title}`
   - the atomic claim with supporting context
   - quoted source excerpts where precision matters
   - footer line: `Source: {provenance}` and, for URL sources, `Fetched: {iso8601}`.
4. Filename: `{Title}.md` in `zettel_root`. If the title contains characters Obsidian prohibits in filenames (`/`, `\`, `:`, `|`, `?`, `*`, `<`, `>`, `"`), replace with safe equivalents and disclose the substitution. If the resulting filename collides, refuse and re-route through dedup — this indicates a dedup miss in Step 4.

### 6. Materialize updates (for `update` / `merge` candidates)

For each candidate routed to an existing zettel:

1. Read the existing zettel.
2. Compute the minimal diff:
   - new tags → append to existing `tags` list (no duplicates).
   - new Aliases → append.
   - new Links → append (no duplicates).
   - new body section → append under a clearly headed section, NEVER mid-document without context.
   - update `Modification Date` to now.
   - NEVER touch `Creation Date`, `Author`, `Title`, `Template`, or `Lang` unless the user explicitly approved.
3. Show the diff. Get explicit approval per file.
4. Apply atomically (temp file + rename in the same directory).

### 7. Approval gate and write

- Default mode: per-candidate approval before each write.
- `--batch-approve`: present the full set as one approval prompt, then write all approved.
- `--dry-run`: stop here; emit the proposal set as the skill's return value.

For each approved write:
- write atomically (temp + rename).
- verify the file parses as valid YAML frontmatter + markdown body.

### 8. Cross-link pass

After all writes are committed:

1. For every newly created zettel, scan its `Links:` for wikilinks pointing to existing zettels.
2. Offer the user the option to add reciprocal links into those targets' `Links:` lists. Default is **off** (do not silently mutate other zettels). Apply only on explicit approval.

### 9. Report

Return:

- created zettel paths
- updated zettel paths with diff summaries
- skipped candidates with reasons
- any source claims that did NOT make it into a zettel and why (e.g. duplicate of existing, too vague, source-only opinion)
- if the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally
- explicit reminder that no git operation was performed
- if the source was a raw note: a suggestion to consider archiving / deleting the raw once the user is satisfied (skill does NOT delete it)

## Validation

Before claiming success:

- [ ] `zettel_root`, `zettel_template`, and (when used) `definition_template` came from `system.md`.
- [ ] The actual template files were read; frontmatter casing matches the template (lowercase `tags:`, etc.).
- [ ] Every zettel covers exactly one atomic idea.
- [ ] Every zettel cites its source in `Links:` and / or a body footer.
- [ ] Dedup pass ran against `zettel_root` for every candidate; overlap reports were shown to the user.
- [ ] No source claim was hallucinated; every body statement traces back to the source.
- [ ] No existing zettel was mutated without an explicit per-file diff approval.
- [ ] `Creation Date` of every existing zettel is unchanged.
- [ ] No git operation was performed.
- [ ] Submodule writes were disclosed.
- [ ] If --lang was supplied, the Lang field matches the --lang override.
- [ ] If --dry-run: no file was written to disk; proposal returned as output only.
