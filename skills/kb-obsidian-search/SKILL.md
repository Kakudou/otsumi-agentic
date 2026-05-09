---
name: kb-obsidian-search
description: "Find relevant zettels in a vault by natural language query, tag intersection, temporal range, or title/alias fuzzy match. Read-only retrieval primitive for the kb-obsidian skill family."
interaction_model: single-shot
---

# KB Obsidian Search

Retrieve matching zettels from a vault's `zettel_root` using one or more of four complementary retrieval strategies: semantic body query, tag intersection, temporal date range, or title/alias fuzzy match. This skill is the **retrieval primitive** for the entire kb-obsidian family — it provides the shared lookup capability that kb-obsidian-assemble, kb-obsidian-zettelize, and future skills consume rather than re-implement. It is immutably read-only: no vault file is ever written, updated, or deleted, under any condition or argument combination.

## Usage

- `/kb-obsidian-search "{query}"` — default vault; semantic body search for `{query}`.
- `/kb-obsidian-search {vault-id} "{query}"` — scope to a named vault entry in `system.md`.
- `/kb-obsidian-search {vault-id} "{query}" --tags "tag/a,tag/b"` — restrict to zettels whose `tags:` frontmatter intersects `{tag/a, tag/b}`.
- `/kb-obsidian-search {vault-id} "{query}" --temporal "2024-01-01..2024-12-31"` — restrict to zettels whose `Creation Date` or `Modification Date` falls within the range (inclusive, ISO 8601).
- `/kb-obsidian-search {vault-id} "{query}" --limit N` — cap the result set to the top `N` matches (default: no cap).
- `/kb-obsidian-search {vault-id} "{query}" --lang {EN|FR|...}` — restrict to zettels whose `Lang` frontmatter matches.
- `/kb-obsidian-search {vault-id}` — list all zettels with no ranking applied (no query, no mode flags).

Retrieval modes may be combined: when multiple flags are present, each mode fires independently and results are merged with per-mode reasoning before final ranking.

## Hard Rules

- MUST resolve `zettel_root` and `zettel_template` from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any vault path.
- MUST read the actual `zettel_template` file to learn the canonical frontmatter field names and casing before parsing any zettel. Field names MUST be read from the template, NEVER guessed.
- MUST return a ranked result list. Each entry MUST include the zettel identifier (vault-relative path or `Title`) and explicit match reasoning that names which retrieval mode(s) fired and why.
- MUST increment `total_access` by 1 in the frontmatter of every zettel included in the returned result set. This is the only frontmatter mutation this skill performs, and it is mandatory.
- MUST NEVER increment `use_count`. Search is passive retrieval, not productive use. `use_count` is owned by skills that generate, write, or synthesize.
- MUST NEVER create, modify (beyond `total_access`), rename, move, or delete any vault file. This is an unconditional immutability contract — no flag or argument overrides it.
- MUST NEVER `git add`, `git commit`, or `git push`.
- MUST surface an explicit error if `zettel_root` is missing from `system.md` or does not exist on disk. NEVER silently return an empty result when the vault is misconfigured.
- MUST surface an explicit error and refuse to proceed if `--temporal` range is malformed (not `YYYY-MM-DD..YYYY-MM-DD`) or if start > end.
- MUST surface a clear warning when the query matches zero zettels across all active retrieval modes. NEVER silently return an empty list without a diagnostic.

## Steps

### 1. Resolve target vault

1. Read `system.md` → `## Knowledge Bases` section.
2. If `{vault-id}` was provided, find the matching `### \`{vault-id}\`` subsection. Otherwise pick the entry tagged `(default)`.
3. Extract `zettel_root`. Refuse with an explicit error if missing or if the directory does not exist on disk.
4. Read `zettel_template`. Capture the canonical frontmatter keys and casing (`tags:` lowercase, `Creation Date:`, `Modification Date:`, `Title:`, `Aliases:`, `Lang:`, `total_access:`, `use_count:`).

### 2. List and parse zettel corpus

1. Enumerate all `.md` files in `zettel_root` (non-recursive unless the vault entry specifies nested layout).
2. For each file, parse the YAML frontmatter using the canonical field casing from `zettel_template`. Extract:
   - `Title`
   - `Aliases` (YAML list; treat as empty list if absent)
   - `tags` (YAML list; treat as empty list if absent)
   - `Lang` (string or absent)
   - `Creation Date` and `Modification Date` (parse as `YYYY/MM/DD HH:mm:ss`)
   - `total_access` (integer; default `0` if absent)
   - body text (full content below the closing `---`)
3. Apply `--lang` filter immediately: drop zettels whose `Lang` does not match the requested value. If no `--lang` flag, keep all.

### 3. Activate retrieval modes

Run each applicable mode as an independent scoring pass. A mode is active when:

- **Semantic query** — always active when `{query}` is non-empty. Assess conceptual overlap between the query string and each zettel's body text. Score as `high`, `medium`, or `low` based on keyword density, proximity of query terms, and presence of synonymous phrasing. Record the matched terms or passages.
- **Tag intersection** — active when `--tags` is provided. Compute the intersection of the supplied tag set with each zettel's `tags` list. Score proportionally to intersection size (exact tags, prefix matches, and hierarchical sub-tag matches all count; record which tags matched). Zettels with zero intersection are excluded from this mode's results.
- **Temporal range** — active when `--temporal` is provided. Parse the `YYYY-MM-DD..YYYY-MM-DD` range. A zettel matches if `Creation Date` OR `Modification Date` falls within the range (inclusive). Record which date field caused the match.
- **Title/alias fuzzy match** — always active when `{query}` is non-empty. Compute fuzzy similarity between the query string and each zettel's `Title` and each entry in `Aliases`. Score as `strong`, `partial`, or `none`. Record which field (`Title` or specific alias) drove the match.

Modes that produce no matches are reported as inactive in the result summary.

### 4. Merge and rank results

1. Collect the union of all matched zettels across active modes.
2. For each zettel in the union, build a `match_record`:
   ```
   - path: "zettels/Indian Attack opening principles.md"
     title: "Indian Attack opening principles"
     modes_fired:
       tag_intersection: "chess/openings/indian-attack" (1/2 tags matched)
       title_fuzzy: partial — "Indian Attack" appears in Title
       semantic_query: medium — mentions fianchetto and king-side structure
   ```
3. Rank by composite signal: zettels firing more modes rank higher; within same mode count, higher scores rank higher (semantic high > medium > low; tag intersection 3/3 > 2/3 > 1/3; title strong > partial).
4. Apply `--limit N` cap if provided: keep only the top `N` entries after ranking.

### 5. Increment `total_access`

For every zettel included in the final ranked result set (after `--limit` is applied):

1. Read the zettel file.
2. Locate the `total_access:` frontmatter field.
3. Increment the integer value by 1.
4. Write the file back. This is the ONLY write this skill performs.

If `total_access` is absent from a zettel's frontmatter, add it with value `1`. NEVER modify any other field or the body.

### 6. Report

Return, in order:

- **Query summary**: which vault, which modes were active, any applied flags (`--tags`, `--temporal`, `--lang`, `--limit`).
- **Ranked result list**: the `match_record` for each matched zettel (path, title, per-mode reasoning). If `--limit` was applied, note how many were omitted.
- **Zero-match diagnostic**: if no zettel matched any active mode, emit an explicit warning naming the query and the modes that were attempted, and suggest narrowing or broadening the query. NEVER silently return an empty list.
- **`total_access` update summary**: list of zettels whose `total_access` was incremented.
- Explicit reminder that no other vault mutation was performed and no git operation was performed.

> **Architectural note for skill authors**: kb-obsidian-assemble Step 2 (Discover candidate zettels) re-implements scoring logic that is now canonically owned by this skill. Skills that need zettel discovery SHOULD call kb-obsidian-search and consume its ranked result list rather than re-implementing title/tag/body scoring internally. The match_record format is the stable interchange contract.

## Validation

Before claiming success:

- [ ] `zettel_root` and `zettel_template` came from `system.md`; nothing was hardcoded.
- [ ] Frontmatter field casing was read from the actual `zettel_template` file, not assumed.
- [ ] All four retrieval modes are documented; active modes fired; inactive modes are reported as inactive.
- [ ] Every result entry includes explicit per-mode match reasoning (which mode fired and why).
- [ ] `total_access` was incremented by 1 on every zettel in the returned result set.
- [ ] `use_count` was not touched on any zettel.
- [ ] No vault file was created, renamed, moved, or deleted.
- [ ] No zettel body or frontmatter field (other than `total_access`) was modified.
- [ ] `--limit` was honored: result set contains at most `N` entries when the flag was provided.
- [ ] Zero-match case was surfaced with an explicit diagnostic, not a silent empty list.
- [ ] No git operation was performed.
