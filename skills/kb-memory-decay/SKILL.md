---
name: kb-memory-decay
description: "Read-only health scan over Memory notes. Surfaces expired STM, stale MTM, low-confidence LTM, review-overdue, superseded candidates, promotion-eligible memory, and orphaned canonical sources. Dry-run only — never mutates the vault, never recommends autonomous content generation."
interaction_model: single-shot
---

# KB Memory Decay

## Mission

Read-only analysis primitive for memory hygiene. Runs seven canonical checks (plus one optional heuristic) against the memory note tree, surfaces findings with a restricted remediation vocabulary, and reports without mutating any file. All remediation decisions belong to the user — decay observes and reports, never acts.

## Usage

| Command | Purpose |
|---|---|
| `/kb-memory-decay` | Scan all memory notes in the default vault. |
| `/kb-memory-decay {vault-id}` | Scan memory notes in the specified vault. |
| `/kb-memory-decay {vault-id} --project otsumi-agentic` | Scope scan to a specific project. |
| `/kb-memory-decay {vault-id} --agent hisha` | Scope scan to a specific agent. |
| `/kb-memory-decay {vault-id} --tier mtm` | Scope scan to a specific tier. |
| `/kb-memory-decay {vault-id} --skip review-overdue` | Skip a specific check. |
| `/kb-memory-decay {vault-id} --skip review-overdue,superseded-candidate` | Skip multiple checks. |
| `/kb-memory-decay {vault-id} --check ltm-low-confidence` | Run only a specific check. |
| `/kb-memory-decay {vault-id} --check agent-private-content-in-project-memory` | Run the optional heuristic check. |
| `/kb-memory-decay {vault-id} --include-archived` | Include archived memory notes in the scan. |

## The Seven Canonical Checks (frozen v1)

Adding or removing checks requires a version bump.

| ID | Default Thresholds | Description |
|---|---|---|
| `expired-stm` | review after 7d, auto-stale flag at 14d, archive-proposal at 30d | STM memory past its expected lifespan. |
| `stale-mtm` | review after 30d, auto-stale flag at 90d | MTM memory not recently recalled or updated. |
| `ltm-low-confidence` | warn at confidence < 0.8; block-promotion at < 0.7 | LTM memory below confidence floor. |
| `review-overdue` | `Review After` date in the past | Explicit user-set review date passed. |
| `superseded-candidate` | heuristic: same scope + same agents/projects + newer alternative with strong title overlap | High false-positive rate. |
| `promotion-eligible` | stm→mtm: Recall Count ≥ 5 AND Confidence ≥ 0.7; mtm→ltm: Recall Count ≥ 10 AND Confidence ≥ 0.85 AND Source Quality ∈ {direct_user_decision, project_artifact} | Memory meets eligibility for tier promotion (advisory). |
| `orphaned-canonical-source` | any wikilink in `Canonical Sources` that no longer resolves to an active zettel (archived, superseded, deprecated, or missing) | Memory wrapper points at a no-longer-canonical source. |

## Optional Heuristic Check (off by default)

| ID | Heuristic |
|---|---|
| `agent-private-content-in-project-memory` | Memory has `Scope: project` AND (`Recall When` OR `Operational Notes`) contains exactly one shogi agent name (case-insensitive match against `osho`, `kakugyo`, `kinsho`, `ginsho`, `hisha`, `kyosha`, `fuhyo`, `keima`). Suggested remediation: "review for scope correction." |

Run with `--check agent-private-content-in-project-memory` explicitly.

## Hard Rules

1. MUST resolve `memory_root` and `memory_template` from system context (`## Knowledge Bases` section, injected as `custom_instruction` at session start). NEVER hardcode.
2. MUST be dry-run only. NO file mutation. NO counter updates. NO archive. NO delete.
3. MUST enumerate exactly seven default checks. Adding/removing requires version bump.
4. MUST permit only this remediation vocabulary:
   - "review for tier promotion"
   - "review for tier demotion"
   - "consider running `/kb-memory-enrich` to refresh metadata"
   - "consider running `/kb-memory-enrich --target {path}` to update Status"
   - "review the linked Canonical Sources"
   - "review for potential supersession"
   - "review for scope correction"
5. MUST NEVER recommend "create a new memory note to fill this gap" or any equivalent content-generation instruction.
6. MUST label `superseded-candidate` findings with high false-positive rate; MUST NOT assert correctness. Surface `--skip superseded-candidate` as the noise workaround.
7. MUST resolve canonical source wikilinks during `orphaned-canonical-source` check by reading zettel frontmatter for `Status` field. Read-only access.
8. MUST handle malformed memory notes gracefully: log path, skip, continue scan. Increment `scanned.skipped_corrupt` counter.
9. MUST handle missing `memory_template`: emit warning, proceed with hardcoded fallback fields, flag every memory note as "unverified frontmatter" in `warnings[]`.
10. MUST handle missing `memory_root`: refuse with explicit error.
11. MUST NEVER `git add`, `git commit`, or `git push`.
12. MUST NEVER read or modify files outside `memory_root` (except reading linked zettels for canonical-source existence verification — read-only).

## Steps

1. Resolve target vault and `memory_template` from system context (`## Knowledge Bases` section, already available as `custom_instruction`).
2. Index memory notes under `memory_root`. Apply scope filters (`--project`, `--agent`, `--tier`).
3. Run each non-skipped check in canonical order. Optional heuristic check runs only if `--check agent-private-content-in-project-memory` is set.
4. For each finding, build a structured entry: path, title, check_id, severity, reason, current_metadata snapshot, suggested_remediation (from permitted vocabulary).
5. Build summary counts.
6. Sort findings per Output Ordering below.
7. Emit the report. Output is read-only and contains no file writes.

## Output Ordering

Findings ordered by:

1. `severity` descending (`error > warning > info`).
2. `check_id` alphabetical ascending.
3. `path` lexicographic ascending.

Stable, scannable output across runs.

## Output Contract

```yaml
memory_decay_report:
  query:
    vault_id:
    project:
    agent:
    tier:
    skipped_checks: []
    explicit_check: null|check_id
  scanned:
    memory_notes: 0
    skipped_corrupt: 0
  findings:
    - path:
      title:
      check_id:
      severity: warning|error|info
      reason:
      current_metadata:
      suggested_remediation:
      requires_user_review:
  summary:
    expired_stm: 0
    stale_mtm: 0
    ltm_low_confidence: 0
    review_overdue: 0
    superseded_candidates: 0
    promotion_eligible: 0
    orphaned_canonical_source: 0
  safety:
    dry_run_only: true
    files_modified: 0
```

## Validation Checklist

- [ ] `memory_root` and `memory_template` came from system context.
- [ ] Exactly seven default checks were defined and either executed or skipped via `--skip` / `--check`.
- [ ] No memory file was modified. `files_modified == 0`.
- [ ] Permitted remediation vocabulary only.
- [ ] `superseded-candidate` findings labeled with high false-positive rate.
- [ ] `orphaned-canonical-source` check resolved every wikilink read-only.
- [ ] Findings sorted per Output Ordering.
- [ ] Report includes per-check finding counts and skipped-check list.
- [ ] Malformed memory notes counted in `scanned.skipped_corrupt`, not raised as errors.
- [ ] Missing `memory_template` emits warning; missing `memory_root` raises explicit error.
- [ ] Optional heuristic check `agent-private-content-in-project-memory` runs only when `--check` flag explicitly names it.
- [ ] No git operation performed.
