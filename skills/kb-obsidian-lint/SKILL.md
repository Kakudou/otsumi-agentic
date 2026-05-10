---
name: kb-obsidian-lint
description: "Run vault-wide health checks against zettel_root and resources_root, surfacing orphans, unassembled clusters, missing crosslinks, stale candidates, and candidate contradictions. Read-only analysis — never mutates the vault, never recommends autonomous content generation."
interaction_model: single-shot
---

# KB Obsidian Lint

Run a vault health scan over `zettel_root` and `resources_root` using exactly five canonical checks.

This skill is a **read-only analysis primitive**:
- It surfaces findings and assigns canonical check IDs.
- It suggests remediation using restricted vocabulary only.
- It never writes/deletes/renames vault notes.
- It never recommends autonomous content generation.
- All remediation decisions belong to the user.

## Usage

- `/kb-obsidian-lint` — run all five checks against the default vault.
- `/kb-obsidian-lint {vault-id}` — scope to a named vault entry in `system.md`.
- `/kb-obsidian-lint --skip {check-id}` — suppress a single named check from the run (e.g. `--skip stale-candidate`).
- `/kb-obsidian-lint --skip {id1},{id2}` — suppress multiple checks in a single flag (comma-separated, no spaces).
- `/kb-obsidian-lint --check {check-id}` — run only the specified check; all others are skipped.
- `/kb-obsidian-lint --threshold N` — set the minimum zettel count for the `unassembled-cluster` check (default: `5`).
- `/kb-obsidian-lint {vault-id} --skip stale-candidate --threshold 3` — flags may be combined freely.

**Canonical check IDs** (the only valid values for `--skip` / `--check`):

| ID | Short description |
|----|-------------------|
| `orphan-zettel` | Zettels with zero inbound wikilinks |
| `unassembled-cluster` | Tags with N+ zettels but no resource note |
| `missing-crosslink` | Zettel pairs sharing 2+ tags but zero mutual wikilinks |
| `stale-candidate` | Older zettels potentially superseded by newer same-tag siblings |
| `candidate-contradiction` | Same-topic zettels with apparently conflicting core claims |

## Planned Checks (unimplemented)

The following check IDs are reserved as planned and are **not part of the active five-check set**:

| Planned ID | Status | Planned behavior |
|------------|--------|------------------|
| `broken-wikilink` | planned/unimplemented | Detect `[[wikilinks]]` that point to non-existent zettels or resource notes. |
| `frontmatter-integrity` | planned/unimplemented | Validate required frontmatter fields against zettel/resource templates. |

## Hard Rules

1. MUST resolve `zettel_root`, `resources_root`, and `zettel_template` from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode vault paths.
2. MUST refuse with an explicit error if the vault entry is missing `zettel_root` or `resources_root`, or if either directory does not exist on disk.
3. This skill is read-only against zettel content and structural metadata per the Zettel Mutability Policy (S3). The sole permitted write is `total_access` counter increments, which are mandatory on file read.
4. MUST NOT recommend autonomous content generation as remediation. Permitted remediation vocabulary is limited to:
   - "run `/kb-obsidian-assemble`"
   - "run `/kb-obsidian-archive`"
   - "consider adding a wikilink from `[[A]]` to `[[B]]`"
   - "review for potential update"
   - "consider updating `Modification Date`"
   Lint MUST NEVER suggest "create a new zettel to fill this gap" (or equivalent content-generation instructions).
5. MUST enumerate exactly five checks: `orphan-zettel`, `unassembled-cluster`, `missing-crosslink`, `stale-candidate`, `candidate-contradiction`. Adding/removing checks requires a version bump.
6. MUST increment `total_access` once for every zettel file opened during scanning.
7. MUST NOT increment `use_count`.
8. For `candidate-contradiction`, MUST present conflicting claims **verbatim** from zettel bodies and MUST NOT assert correctness/authority/recency.
9. For `stale-candidate`, MUST label every finding as **high false-positive rate**, MUST NOT autonomously mark/archive zettels, and MUST surface `--skip stale-candidate` as the noise workaround.
10. NEVER run git operations (`git add`, `git commit`, `git push`, or any other git command).
11. NEVER read or modify files outside `zettel_root` and `resources_root`, except reading `system.md` for vault resolution.

## Execution Contract

### Inputs
- Optional `{vault-id}`
- Optional `--skip {check-id}` or `--skip {id1},{id2}`
- Optional `--check {check-id}` (run only one check)
- Optional `--threshold N` for `unassembled-cluster` (default `5`)

### Required Data Sources
- `system.md` (vault resolution only)
- `zettel_root/**/*.md`
- `resources_root/**/*.md`

### Output Contract
- Emit one report section per executed check in canonical order.
- Emit a final Summary table with per-check counts, skipped checks, scanned counts, and safety statement.
- If a required vault path is missing/invalid, refuse with explicit error.

## Steps

### 1. Resolve target vault

1. Read `system.md`, then locate `## Knowledge Bases`.
2. If `{vault-id}` is provided, select matching `### \`{vault-id}\``; otherwise select the entry marked `(default)`.
3. Extract `zettel_root`, `resources_root`, and `zettel_template`.
4. Refuse with explicit error if required fields are missing.
5. Verify `zettel_root` and `resources_root` exist on disk; refuse if either is missing.
6. Read `zettel_template` to capture canonical frontmatter fields, especially:
   - `Tags`
   - `Links`
   - `Creation Date`
   - `Modification Date`
   - `total_access`
   - `use_count`
7. Log resolved paths in the report.

### 2. Index the vault

Build an in-memory index by scanning `.md` files in `zettel_root` and `resources_root`:

- **Per-zettel record**: `{path, title, aliases, tags, links_frontmatter, wikilinks_in_body, creation_date, modification_date}`
- **Inbound-link map**: for every wikilink target found in any vault file, map `target → {source_path}`
- **Tag index**: `tag → [zettel_path, ...]`
- **Resource-note set**: all `.md` file paths in `resources_root`

Increment `total_access` once per zettel file opened. Do not touch `use_count`.

Apply `--skip` / `--check` flags:
- Skip building data used only by suppressed checks when unambiguously safe.
- Otherwise build the full index, then filter at report time.

### 3. Run checks

Execute each non-skipped check in canonical order. Collect findings into a structured list keyed by check ID.

#### Check: `orphan-zettel`

- For each zettel, look up its `title` (and `aliases`) in the inbound-link map.
- A zettel is **orphaned** if neither its title nor any alias appears as the target of a wikilink from any other vault file.
- Note: new zettels are often orphaned by design — this is a signal, not an error. The finding description must say so.
- Remediation vocabulary: "consider adding a wikilink from a related zettel or resource note."

#### Check: `unassembled-cluster`

- Count zettels per tag across the tag index.
- For each tag with count ≥ `threshold` (default: `5`), check whether `resources_root` contains a note whose `Tags` frontmatter or filename corresponds to that tag.
- Matching strategy: normalize tag path separators (`/`, `-`) and compare case-insensitively against resource note titles and tags.
- A tag cluster is **unassembled** if no plausible resource note exists for it.
- Remediation vocabulary: "run `/kb-obsidian-assemble \"{tag-topic}\"`."

#### Check: `missing-crosslink`

- For each ordered pair of zettels `(A, B)` where `A < B` (to avoid duplicates), compute tag intersection.
- If `|intersection| ≥ 2`: check whether `A` links to `B` (in `links_frontmatter` or `wikilinks_in_body`) AND whether `B` links to `A`.
- If **neither** direction has a link, the pair is a **missing-crosslink** candidate.
- Report the pair with their shared tags and the direction(s) absent.
- Remediation vocabulary: "consider adding a wikilink from `[[A]]` to `[[B]]` or vice versa."

#### Check: `stale-candidate`

> ⚠️ **High false-positive rate.** This check uses heuristics (date comparison + tag overlap) to surface potentially superseded zettels. Many findings will be false positives — the check does NOT know whether a newer zettel truly replaces an older one. Use `--skip stale-candidate` if the signal-to-noise ratio is too low for your vault.

- Group zettels by their tag set (require at least one shared tag).
- Within each group, sort by `Modification Date` descending (fall back to `Creation Date` if `Modification Date` is absent).
- Flag the older zettel(s) as **stale candidates** when:
  - A newer zettel in the same tag group exists.
  - The older zettel's `Modification Date` is more than 90 days behind the newest sibling.
  - The older zettel's title is a substring or near-duplicate of the newer zettel's title (similarity heuristic).
- NEVER assert that the older zettel is obsolete — report it as a candidate only.
- Remediation vocabulary: "review for potential update"; "run `/kb-obsidian-archive`" if the zettel is confirmed superseded.

#### Check: `candidate-contradiction`

- Identify zettel groups that share at least one common tag and whose titles suggest the same topic (substring or fuzzy match).
- Within each group, compare the opening declarative sentence(s) of each zettel's body (the claim).
- Flag a pair as a **candidate-contradiction** when the claims use negating or opposing language (e.g., "X is Y" vs. "X is not Y", or factual quantities that differ).
- **Present both claims verbatim** in the finding. NEVER paraphrase, synthesize, or assert which is correct.
- Lint's role ends at presentation. The user decides whether a contradiction exists, which zettel is authoritative, or whether both are valid in context.
- Remediation vocabulary: "review for potential update"; "consider adding a wikilink acknowledging the relationship."

### 4. Produce the lint report

Emit one section per executed check (even if a check has zero findings — report "No findings"):

```
## {check-id}

**Status**: {N findings | No findings}

{Per-finding block:}

### Finding {n}
- **Check**: {check-id}
- **Affected**: {path(s)}
- **Evidence**: {description — verbatim claims for candidate-contradiction; shared tags for missing-crosslink; etc.}
- **Suggested remediation**: {from permitted vocabulary only}
```

Follow with a **Summary** section:

```
## Summary

| Check                  | Findings |
|------------------------|----------|
| orphan-zettel          | N        |
| unassembled-cluster    | N        |
| missing-crosslink      | N        |
| stale-candidate        | N (high false-positive rate — review manually) |
| candidate-contradiction| N        |

Skipped checks: {list or "none"}
Zettels scanned: {count}
Resources scanned: {count}
No git operation was performed. No vault file was modified.
```

## Validation

Before claiming success:

- [ ] `zettel_root` and `resources_root` came from `system.md`; no path was hardcoded.
- [ ] Exactly five checks were defined and either executed or explicitly skipped via `--skip` / `--check`.
- [ ] `total_access` was incremented once per zettel file opened. `use_count` was not touched.
- [ ] Zettel content and structural metadata remained immutable per the Zettel Mutability Policy (S3); the only permitted mutation was mandatory `total_access` increments on file read.
- [ ] For `candidate-contradiction` findings: conflicting claims are quoted verbatim; no assertion of correctness appears.
- [ ] For `stale-candidate` findings: high false-positive rate is disclosed on each finding; no zettel was marked or archived.
- [ ] All remediation suggestions are within the permitted vocabulary (no "create a new zettel" recommendations).
- [ ] The report includes a Summary table with per-check finding counts and a skipped-checks list.
- [ ] No git operation was performed.
