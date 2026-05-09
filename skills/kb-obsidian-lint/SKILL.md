---
name: kb-obsidian-lint
description: "Run vault-wide health checks against zettel_root and resources_root, surfacing orphans, unassembled clusters, missing crosslinks, stale candidates, and candidate contradictions. Read-only analysis — never mutates the vault, never recommends autonomous content generation."
interaction_model: single-shot
---

# KB Obsidian Lint

Scan a vault's `zettel_root` and `resources_root` for structural and relational health signals across five named checks. Lint is a **read-only analysis primitive** — it surfaces findings, assigns canonical check IDs, and suggests remediation vocabulary, but it never writes to the vault, never deletes notes, and never proposes content that should be autonomously generated. Every remediation decision belongs to the user.

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

## Hard Rules

- MUST resolve `zettel_root`, `resources_root`, and `zettel_template` from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any vault path inside this skill.
- MUST refuse with an explicit error if the resolved vault entry is missing `zettel_root` or `resources_root`, or if those directories do not exist on disk.
- MUST NOT write, delete, rename, or modify (beyond `total_access`) any file in the vault. Lint is **strictly read-only**. This is non-negotiable.
- MUST NOT recommend autonomous content generation as remediation. Permitted remediation vocabulary is limited to:
  - "run `/kb-obsidian-assemble`"
  - "run `/kb-obsidian-archive`"
  - "consider adding a wikilink from `[[A]]` to `[[B]]`"
  - "review for potential update"
  - "consider updating `Modification Date`"
  Lint MUST NEVER suggest "create a new zettel to fill this gap" or any equivalent instruction to generate content.
- MUST enumerate exactly five checks: `orphan-zettel`, `unassembled-cluster`, `missing-crosslink`, `stale-candidate`, `candidate-contradiction`. Adding or removing checks requires a skill version bump.
- MUST increment `total_access` in the frontmatter field for every zettel file opened during the scan pass. MUST NOT increment `use_count` — lint is read-only analysis, not productive zettel use.
- For `candidate-contradiction`: MUST present conflicting claims **verbatim** from each zettel's body. MUST NOT assert which claim is correct, more recent, or more authoritative. The user decides.
- For `stale-candidate`: MUST document prominently in every finding that this check has a **high false-positive rate**. MUST NOT autonomously mark or archive any zettel. MUST surface `--skip stale-candidate` as the recommended workaround when users find the signal too noisy.
- NEVER `git add`, `git commit`, or `git push`. No git operation of any kind.
- NEVER read or modify files outside `zettel_root` and `resources_root` except `system.md` (for vault resolution).

## Steps

### 1. Resolve target vault

1. Read `system.md` → locate `## Knowledge Bases`.
2. If `{vault-id}` was supplied, find the matching `### \`{vault-id}\`` subsection. Otherwise pick the entry tagged `(default)`.
3. Extract `zettel_root`, `resources_root`, and `zettel_template`. Refuse with an explicit error if any required field is absent.
4. Verify that `zettel_root` and `resources_root` exist on disk. Refuse if missing.
5. Read `zettel_template` to obtain the canonical frontmatter field list (especially: `Tags`, `Links`, `Creation Date`, `Modification Date`, `total_access`, `use_count`).
6. Log resolved paths for transparency in the report.

### 2. Index the vault

Build an in-memory index by scanning every `.md` file in `zettel_root` (and `resources_root` for inbound-link detection):

- **Per-zettel record**: `{path, title, aliases, tags, links_frontmatter, wikilinks_in_body, creation_date, modification_date}`.
- **Inbound-link map**: for each wikilink `[[Target]]` found in any vault file (zettel or resource), record `target → {source_path}` in a reverse lookup table.
- **Tag index**: `tag → [zettel_path, ...]` across all zettels.
- **Resource-note set**: all `.md` files in `resources_root`.

Increment `total_access` in the frontmatter once per zettel file opened. `use_count` is NOT touched.

Apply `--skip` and `--check` flags: skip building data only used by suppressed checks when unambiguously safe; otherwise build the full index and filter at reporting time.

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
- [ ] No vault file was written, deleted, renamed, or modified in any way.
- [ ] For `candidate-contradiction` findings: conflicting claims are quoted verbatim; no assertion of correctness appears.
- [ ] For `stale-candidate` findings: high false-positive rate is disclosed on each finding; no zettel was marked or archived.
- [ ] All remediation suggestions are within the permitted vocabulary (no "create a new zettel" recommendations).
- [ ] The report includes a Summary table with per-check finding counts and a skipped-checks list.
- [ ] No git operation was performed.
