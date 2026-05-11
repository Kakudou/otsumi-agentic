---
name: kb-obsidian-remember
description: "Capture raw content into the vault's raw_root with an epoch-timestamped filename, byte-for-byte unmodified, no frontmatter, no formatting."
interaction_model: single-shot
---

# KB Obsidian Remember

Persist arbitrary content into a vault's `raw_root` exactly as supplied — no transformation, no metadata, no opinion — using a Unix epoch filename for uniqueness.

## Usage

- `/kb-obsidian-remember "{content}"` — capture inline content into the default vault's `raw_root`.
- `/kb-obsidian-remember --from-file {path}` — capture file into default vault's `raw_root`.
- `/kb-obsidian-remember --from-clipboard` — capture clipboard into default vault's `raw_root`.
- `/kb-obsidian-remember {vault-id} "{content}"` — same, scoped to a named vault entry in `system.md`.
- `/kb-obsidian-remember {vault-id} --from-file {path}` — capture the bytes of a local file (the file is read; the original is not moved or deleted).
- `/kb-obsidian-remember {vault-id} --from-clipboard` — capture clipboard contents (skill MUST surface that it is reading the clipboard before doing so).
- `/kb-obsidian-remember {vault-id} --ext {ext}` — override the file extension. Default is `.md`.

## Hard Rules

- MUST resolve `raw_root` from system context (`## Knowledge Bases` → `{vault-id}` → `Note-type roots`, injected as `custom_instruction` at session start). NEVER hardcode a folder path inside this skill.
- MUST refuse with an explicit error if the resolved vault has no `raw_root` slot, or if the directory does not exist on disk.
- MUST write the content **byte-for-byte unchanged**. NEVER add frontmatter, NEVER reformat, NEVER trim trailing whitespace, NEVER re-encode line endings, NEVER summarize, NEVER translate, NEVER reflow.
- MUST use a Unix epoch timestamp (seconds) as the bare filename: `{epoch}.md` by default. On collision (same-second double capture), MUST fall through to millisecond precision: `{epoch_ms}.md`. On a further collision, append `-{n}` with `n` starting at `2`.
- MUST verify the target file does not already exist before writing. NEVER overwrite an existing raw file.
- MUST warn the user if source content exceeds 1 MB before proceeding. MUST refuse with an explicit error if source content exceeds 10 MB.
- MUST log `remember.failed` via `kb-obsidian-log` on any error path, including the concrete error reason, so failed captures are traceable alongside `remember.completed` success events.
- NEVER `git add`, `git commit`, or `git push` from this skill. Capture is local-only.
- NEVER read or modify any other file in the vault.
- NEVER prompt the user for "improvements" or "tags" — this skill is the dumb-fast capture path. Refinement is `kb-obsidian-zettelize`'s job.

## Steps

### 1. Resolve target vault

1. Resolve from system context (`## Knowledge Bases` section, already available as `custom_instruction`).
2. If `{vault-id}` was provided, find the matching `### \`{vault-id}\`` subsection. Otherwise pick the entry tagged `(default)`.
3. Extract `raw_root`. Refuse if missing.
4. Verify `raw_root` exists on disk. Refuse if missing.

### 2. Source the content

- **Inline mode**: use the quoted `{content}` argument verbatim.
- **`--from-file`**: read the file at the given path as bytes. NEVER decode/re-encode.
- **`--from-clipboard`**: announce the clipboard read, then capture.

Check source size before proceeding: warn if > 1 MB, refuse with an explicit error if > 10 MB.

### 3. Compute filename

1. `epoch_seconds = floor(now().unix())`.
2. Candidate path: `{raw_root}/{epoch_seconds}.{ext}` (`ext` defaults to `md`).
3. If the candidate exists:
   - Recompute with millisecond precision: `{raw_root}/{epoch_ms}.{ext}`.
   - If THAT exists, append `-2`, `-3`, ... until free.

### 4. Write atomically

1. Write to a sibling temp file in `raw_root` (e.g. `.{epoch}.{ext}.tmp`).
2. Rename over the final path.
3. Verify the written byte length equals the source byte length. Refuse / report mismatch otherwise.

### 5. Report

Return:

- absolute path written
- byte length
- vault id and `raw_root` used
- if the target path resolves under a directory listed as a submodule in the vault's safety caveats, emit a submodule write disclosure unconditionally
- explicit reminder that no git operation was performed
- next-step suggestion: `/kb-obsidian-zettelize {vault-id} {written-path}` when the user is ready to atomize this capture into zettels

## Validation

Before claiming success:

- [ ] `raw_root` came from system context; nothing was hardcoded.
- [ ] Filename is a pure epoch (seconds → ms → suffixed) with no human title baked in.
- [ ] No frontmatter, no formatting, no transformation was applied.
- [ ] Pre-existing files in `raw_root` are untouched.
- [ ] Byte length on disk matches the source byte length.
- [ ] No git operation was performed.
- [ ] If --from-clipboard: clipboard read was disclosed to the user before capture proceeded.
