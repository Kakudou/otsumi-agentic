---
name: kb-obsidian-archive
description: "Manage the lifecycle of vault notes (zettels and resources) by marking them as archived, superseded, or deprecated via frontmatter status fields. Non-destructive — never deletes files. Physical move is optional and human-gated."
interaction_model: multi-turn
---

# KB Obsidian Archive

Mark vault notes as `archived`, `superseded`, or `deprecated` by writing lifecycle-status fields into their frontmatter. Archive is the lifecycle counterpart to creation (`kb-obsidian-zettelize`) and curation (`kb-obsidian-assemble`) — it handles what happens when knowledge becomes stale, duplicated, or no longer relevant. Every status change is individually human-gated; no bulk-apply is possible without per-file confirmation. The skill never deletes any file. Physical relocation of a note is an optional, separate step and requires that all inbound wikilinks be resolved first.

## Usage

- `/kb-obsidian-archive {zettel-path} --status archived` — mark a zettel as archived; no replacement pointer needed.
- `/kb-obsidian-archive {zettel-path} --status superseded --by "[[replacement-title]]"` — mark as superseded with a navigable `Superseded-by:` wikilink.
- `/kb-obsidian-archive {zettel-path} --status deprecated` — mark as deprecated; content is known to be inaccurate or outdated but no replacement exists yet.
- `/kb-obsidian-archive {zettel-path} --move {target-dir}` — after applying a lifecycle status, also physically move the file to `{target-dir}`. Requires all inbound wikilinks to be updated or the move is refused.
- `/kb-obsidian-archive --bulk {tag-filter}` — find all notes matching `{tag-filter}`, present each for individual lifecycle-status approval. NEVER applies changes in bulk without per-file confirmation.
- `/kb-obsidian-archive {vault-id} {zettel-path} --status archived` — scope to a named vault entry defined in `system.md`.
- `/kb-obsidian-archive --dry-run {zettel-path} --status {status}` — show what would change without writing any file.

## Hard Rules

- MUST resolve all vault paths (`zettel_root`, `resources_root`, `zettel_template`) from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any path.
- MUST read `zettel_template` before modifying any note's frontmatter, to confirm field casing, date format (`YYYY/MM/DD HH:mm:ss`), and existing field names. Status fields (`Status:`, `Superseded-by:`) are ADDED to the note — existing template fields are not reordered or removed.
- MUST require individual human approval for every file whose status would change. NEVER apply status changes to multiple files in a single unconfirmed batch.
- MUST perform a cross-reference update pass: after marking a note, scan the vault for inbound wikilinks pointing to the now-marked note. Surface all referencing files to the user and offer the choices defined in Step 6. NEVER silently leave stale inbound links unaddressed.
- MUST write frontmatter modifications atomically — write to a temp file in the same directory, then rename to the final path. NEVER write directly to the live file.
- MUST increment `total_access` for each zettel read when checking or changing its status. NEVER increment `use_count` — archiving is lifecycle management, not productive content use.
- MUST update `Modification Date` when writing lifecycle-status fields. NEVER modify `Creation Date` for any reason.
- MUST emit a submodule write disclosure whenever the target file resolves under a directory listed as a submodule in the vault's safety caveats (`100-Personal/`, `600-Workspace/`, `4242-Otsumi/`).
- ARCHIVE NEVER DELETES FILES. Status is recorded via frontmatter. Physical move is optional, separately gated, and requires full inbound-wikilink resolution before the move executes.
- NEVER `git add`, `git commit`, or `git push`.
- NEVER modify files outside the target vault's absolute path.
- NEVER apply `--move` if any inbound wikilink pointing to the note cannot be updated — refuse and report which referencing files block the move.
- NEVER invent a `Superseded-by:` target. The replacement wikilink MUST be user-supplied via `--by`.

## Steps

### 1. Resolve target vault

1. Read `system.md` → `## Knowledge Bases` → vault entry (by `{vault-id}` or default).
2. Extract `zettel_root`, `resources_root`, `zettel_template`, and all safety caveats.
3. Read `zettel_template`. Capture:
   - canonical frontmatter keys and casing (e.g. lowercase `tags:`, capitalized `Title:`)
   - date format (`YYYY/MM/DD HH:mm:ss`)
   - existing field names, so the archive-added fields (`Status:`, `Superseded-by:`) are consistent with the template's style.
4. Check whether the target path resolves under a submodule directory. If yes, set a `submodule_disclosure_required` flag — emit the disclosure unconditionally in every subsequent output that touches that file.

### 2. Resolve and validate the target note

1. Resolve the supplied `{zettel-path}` against the vault's absolute path.
2. Read the target note. Confirm it exists and is a valid markdown file with YAML frontmatter.
3. Check the current `Status:` field:
   - if `Status:` is already set to the requested value → inform the user; no write is needed unless they explicitly confirm override.
   - if `Status: superseded` is already set and `--by` was not supplied → refuse; a `Superseded-by:` pointer is required.
4. Increment `total_access` by 1 for this read.

### 3. Validate lifecycle state arguments

For each requested status, enforce:

| Status | Required args | Forbidden args |
|---|---|---|
| `archived` | `--status archived` | `--by` (not applicable) |
| `superseded` | `--status superseded --by "[[replacement]]"` | omitting `--by` |
| `deprecated` | `--status deprecated` | `--by` (not applicable) |

If `--status superseded` is given without `--by`, refuse immediately and explain. If `--by` is given without `--status superseded`, warn and ignore `--by`.

Verify that the `--by` wikilink target exists in the vault when supplied. If it does not exist, surface a warning and let the user decide: `proceed anyway` / `abort`.

### 4. Compute the frontmatter diff

Build the set of frontmatter changes to apply:

1. **`Status:`** — set to `archived`, `superseded`, or `deprecated`.
2. **`Superseded-by:`** — add only when `--status superseded`. Value is the user-supplied wikilink (e.g. `[[replacement-title]]`). Place immediately after `Status:` in the frontmatter block.
3. **`Modification Date:`** — update to current time in `YYYY/MM/DD HH:mm:ss`.
4. All other existing frontmatter fields — leave untouched.
5. `Creation Date:` — NEVER modified.

Present the proposed diff to the user before any write:

```diff
 Status: [NEW] archived
 Modification Date: 2024/11/15 14:32:00  ← was 2024/03/01 09:12:00
```

Wait for explicit `approve` / `edit` / `skip` / `abort` per file.

### 5. Write atomically

Only after approval:

1. Write the modified frontmatter + body to a temp file in the same directory as the target (e.g. `.{filename}.tmp`).
2. Rename the temp file to the final path (atomic replace).
3. Verify the final file parses as valid YAML frontmatter + markdown body.
4. If `submodule_disclosure_required`, emit the disclosure in this step's output.

### 6. Cross-reference update pass

After writing the status change:

1. Scan all markdown files in the vault for inbound wikilinks pointing to the now-marked note. Match by filename stem and any known `Aliases:` values from the note's frontmatter.
2. Build the `inbound_refs` list:
   ```
   inbound references to "old-zettel-title":
     - resources/Indian Attack overview.md (line 14): [[old-zettel-title]]
     - zettel/Chess positional play.md (line 7): [[old-zettel-title]]
   ```
3. Present the inbound refs to the user with these options per referencing file:

   **For `archived` and `deprecated` notes:**
   - `add-warning-callout` — prepend a `> [!WARNING] This note has been {status}. Content may be outdated.` callout at the top of the marked note's body (below frontmatter). Does NOT touch the referencing files.
   - `leave-unchanged` — no further action on referencing files.

   **For `superseded` notes (additionally):**
   - `update-refs` — in each listed referencing file, replace `[[old-zettel-title]]` with `[[replacement-title]]`. Each referencing-file update requires its own atomic write and its own per-file approval.
   - `leave-unchanged` — keep existing wikilinks as-is; the `Superseded-by:` field in the marked note provides the navigable trail.

4. Apply the user's chosen action. Each referencing-file modification uses the same atomic write pattern (temp + rename). Increment `total_access` for each referencing file read during the scan.

### 7. Optional physical move (`--move {target-dir}`)

Only execute this step if `--move` was supplied:

1. Confirm `{target-dir}` exists within the vault's absolute path. If not, create it (with user confirmation) or refuse.
2. Re-run the inbound-refs scan from Step 6 (updated, post-write state). If any inbound wikilink still points to the original path and the user did not choose `update-refs` in Step 6, REFUSE the move and report the blocking files. The user must resolve references before the move can proceed.
3. Present the move plan: source path → destination path.
4. Wait for explicit approval.
5. Execute atomically:
   a. Copy the source to `{target-dir}/{filename}` (write via temp + rename).
   b. Remove the source file.
6. Update all inbound wikilinks (if any remain pointing to the old path after Step 6) to the new path. Each requires per-file atomic write and approval.
7. Emit submodule disclosure if the destination path also resolves under a submodule.

### 8. Bulk candidate flow (`--bulk {tag-filter}`)

1. List all notes in `zettel_root` (and optionally `resources_root`) that match `{tag-filter}`.
2. Cross-reference against any `stale-candidate` findings surfaced by `kb-obsidian-lint` (if lint output is provided as context). The `stale-candidate` check ID is the canonical upstream trigger for archive decisions — when lint surfaces a `stale-candidate`, `kb-obsidian-archive` is the recommended remediation path.
3. Present each candidate one at a time with its metadata (Title, tags, Modification Date, last-known use stats). Per-file prompt:
   - `archived` — set Status: archived.
   - `superseded --by "[[...]]"` — mark superseded; user supplies the wikilink inline.
   - `deprecated` — set Status: deprecated.
   - `skip` — leave this note unchanged.
   - `abort-bulk` — stop the bulk flow; no further candidates are processed.
4. For each confirmed candidate, execute Steps 2–6 fully before advancing to the next candidate.

### 9. Dry-run mode (`--dry-run`)

If `--dry-run` was supplied:

- Execute Steps 1–4 (resolve, validate, compute diff).
- At Step 4, emit the full proposed diff for every target file.
- Stop. Write nothing. Move nothing.
- Return the dry-run report as the skill's output, clearly labelled `[DRY RUN — no files written]`.

### 10. Report

Return:

- files whose status was changed, with the new status and diff summary.
- inbound references found, with the action taken per referencing file.
- files moved (if `--move`), with source and destination paths.
- any skipped targets with the reason (e.g. status already set, user skipped, inbound-link blocker).
- a warning if any `Superseded-by:` target wikilink could not be verified to exist.
- `total_access` increment count for the session.
- if `submodule_disclosure_required`: emit submodule disclosure unconditionally.
- explicit reminder: **no git operation was performed**.
- if `--bulk` was used and lint's `stale-candidate` findings were referenced: confirm which lint findings were resolved.

## Validation

Before claiming success:

- [ ] All vault paths (`zettel_root`, `zettel_template`) came from `system.md`; no hardcoded paths appear.
- [ ] `zettel_template` was read before any frontmatter modification; field casing and date format match the template.
- [ ] Every status change received individual human approval before the write executed.
- [ ] `Status:` is exactly one of: `archived`, `superseded`, `deprecated`. No other values were written.
- [ ] For every `superseded` note, `Superseded-by:` was written immediately after `Status:` with the user-supplied wikilink.
- [ ] `Modification Date` was updated on every written file.
- [ ] `Creation Date` was not touched on any file.
- [ ] Every frontmatter write used the atomic temp-file + rename pattern; no direct in-place writes occurred.
- [ ] `total_access` was incremented for every zettel read (target notes and referencing files).
- [ ] `use_count` was not incremented on any file.
- [ ] Cross-reference scan ran after each status write; inbound refs were presented to the user.
- [ ] If `--move` was used: all inbound wikilinks were resolved or the move was refused.
- [ ] If `submodule_disclosure_required`: submodule disclosure was emitted in every output that touched the submodule path.
- [ ] No file was deleted.
- [ ] No git operation was performed.
- [ ] If `--dry-run`: no file was written to disk; the proposal was returned as output only, labelled `[DRY RUN]`.
