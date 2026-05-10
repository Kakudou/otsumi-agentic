---
name: kb-obsidian-log
description: "Append-only operations log for vault skill events (remember, zettelize, assemble, lint, write, search, archive) and a read interface for querying that history. Delegates all append mechanics to core-atomic-log."
interaction_model: single-shot
---

# KB Obsidian Log

Provide append-only operations logging for kb-obsidian skill events and a structured read interface for querying that history.

Every vault skill event (`remember`, `zettelize`, `assemble`, `search`, `lint`, `write`, `archive`) is captured as a timestamped structured entry in:

`.otsumi/kb-vault/{vault-id}/events.json`

This skill is infrastructure support:
- It NEVER re-implements append mechanics.
- It ALWAYS delegates append behavior to `core-atomic-log`.
- If append fails, the parent vault operation is NOT blocked.

## Usage

### Append mode (called by orchestration after a vault skill completes)

- `/kb-obsidian-log append {vault-id} {event-type} "{details}"` — append one vault skill event for the named vault.
- `/kb-obsidian-log append {vault-id} zettelize.completed "created=3 updated=1 skipped=2 source=1712345678.md"` — example zettelize outcome.
- `/kb-obsidian-log append {vault-id} remember.completed "written=1712345678.md bytes=4201"` — example remember outcome.
- `/kb-obsidian-log append {vault-id} assemble.completed "resource=Indian-Attack.md zettels=7 gaps=1"` — example assemble outcome.
- `/kb-obsidian-log append {vault-id} search.completed "query=\"Indian Attack\" hits=12"` — example search outcome.
- `/kb-obsidian-log append {vault-id} lint.completed "checks=5 findings=2 check-ids=orphan-links,missing-template"` — example lint outcome.
- `/kb-obsidian-log append {vault-id} write.completed "output=chess-digest.md words=1240"` — example write outcome.
- `/kb-obsidian-log append {vault-id} archive.completed "target=old-note.md status=archived"` — example archive outcome.

### Read mode (query existing history)

- `/kb-obsidian-log read {vault-id}` — dump full history for the vault.
- `/kb-obsidian-log read {vault-id} --last {N}` — show the last N events.
- `/kb-obsidian-log read {vault-id} --skill {skill-name}` — filter by skill event prefix (e.g. `zettelize`, `remember`).
- `/kb-obsidian-log read {vault-id} --since {YYYY-MM-DD}` — filter events on or after a date.
- `/kb-obsidian-log read {vault-id} --event {event-type}` — filter by exact event type string.

## Hard Rules

- MUST resolve `vault-id` from `system.md` → `## Knowledge Bases`. NEVER construct a scope name from a hardcoded vault identifier.
- MUST delegate all append operations to `core-atomic-log`. NEVER re-implement append logic, file initialization, array parsing, corruption detection, or atomic write mechanics.
- MUST use the scope-name format `kb-vault/{vault-id}` for every vault skill log. This namespace is distinct from pipeline/workflow scopes (which use bare feature names such as `user-auth`) and MUST NOT be reused by dev-workflow pipelines.
- MUST NOT block the parent vault operation on append failure. If `core-atomic-log` fails or is unavailable, log the failure at most once to the conversation and proceed. Silent degradation is mandatory.
- NEVER `git add`, `git commit`, or `git push`.
- NEVER read, write, or modify vault files (`zettel_root`, `raw_root`, `resources_root`, or any other vault-managed path). This skill only touches `.otsumi/kb-vault/{vault-id}/events.json`.
- NEVER participate in usage scoring. This skill does not read zettels.
- NEVER invent event details. Details MUST be derived from the invoking skill's return values; if no details are available, use an explicit `"details": "no output data"` rather than fabricating counts or paths.
- NEVER write more than one event per invocation.
- MUST report corruption in the existing log explicitly (delegating to `core-atomic-log` step 3 behavior) and stop. Never silently reset a corrupted log.

## Steps

### 1. Resolve target vault

1. Read `system.md` → `## Knowledge Bases`.
2. Locate the vault entry matching `{vault-id}`. If `{vault-id}` was not provided, use the entry marked `(default)`.
3. Confirm the vault entry exists. Refuse with an explicit error if not found.
4. Derive the scope name: `kb-vault/{vault-id}`. Example: vault id `volgna-gath` → scope `kb-vault/volgna-gath`.

Note: this skill does NOT read `raw_root`, `zettel_root`, or any other vault-managed path. Resolution is limited to obtaining the vault id for scope construction.

### 2. Architecture decision (wrapper model, Option A)

This skill is invoked by the orchestration layer (Ōshō or Kakugyō) after a kb-obsidian skill completes, not from within each skill's own Steps.

#### Options evaluated

##### Option A — Wrapper (Ōshō-level integration)
The orchestration layer appends one log event after each vault skill returns. The vault skills themselves have no logging awareness.

- Pro: zero modification to existing skill contracts; skills stay unaware of logging infrastructure.
- Pro: the orchestration layer already has access to the skill's return values, enabling meaningful details.
- Pro: kb-obsidian SKILL.md files are specification documents, not executable code — adding a logging step to each one creates a maintenance surface without mechanical enforcement.
- Con: cannot capture skill-internal events (e.g., intermediate dedup pass counts in zettelize). Outcome-level logging only.

##### Option B — Augmentation (per-skill authorized_skills inclusion)
Each vault skill declares `kb-obsidian-log` in its `authorized_skills` or adds an explicit logging step to its Steps section.

- Pro: finer-grained events possible.
- Con: requires modifying every existing skill's contract; couples the skills to logging infrastructure.
- Con: any future vault skill must remember to include the log step.
- Con: SKILL.md files have no enforcement mechanism — the coupling becomes a documentation burden, not a contract guarantee.

#### Decision
**Option A (Wrapper).**

The event vocabulary (`*.completed`) is inherently outcome-level. The orchestration layer observes every skill invocation and its result; it has the data needed to build the `details` string. Keeping skills unaware of logging preserves the clean single-concern contract each skill already has. If finer-grained intra-skill events are needed in the future, a Kakugyō replan can introduce them incrementally per skill without touching the rest of the family.

### 3. Construct the event payload

Build the 3 required arguments for `core-atomic-log`:

1. **Scope name**: `kb-vault/{vault-id}` (from Step 1).
2. **Event type**: one of the documented vocabulary types (see Event Vocabulary below), e.g. `zettelize.completed`.
3. **Details**: a compact key=value summary derived from the invoking skill's return values. Must carry signal. Examples:
   - `zettelize.completed` → `"created=3 updated=1 skipped=2 source=1712345678.md"`
   - `remember.completed` → `"written=1712345678.md bytes=4201"`
   - `assemble.completed` → `"resource=Indian-Attack.md zettels=7 gaps=1"`
   - `search.completed` → `"query=\"Indian Attack\" hits=12"`
   - `lint.completed` → `"checks=5 findings=2 check-ids=orphan-links,missing-template"`
   - `write.completed` → `"output=chess-digest.md words=1240"`
   - `archive.completed` → `"target=old-note.md status=archived"`

If no return-value data is available, use `"details": "no output data"`. Never fabricate counts, paths, or identifiers.

### 4. Delegate append to core-atomic-log

Invoke `core-atomic-log` as:

```
/core-atomic-log kb-vault/{vault-id} {event-type} "{details}"
```

`core-atomic-log` handles:
- Resolving `.otsumi/kb-vault/{vault-id}/events.json`
- Initializing the file as `[]` if it does not exist
- Detecting and reporting corruption
- Appending exactly one structured event
- Writing the full array back atomically

Do not duplicate any of this logic.

On failure: if `core-atomic-log` reports an error or is unavailable, surface the failure once to the conversation (e.g. `"Log append failed: {reason}"`) and allow the parent operation to conclude normally. Do not propagate the failure as a blocking error.

### 5. Read mode — query history

When invoked with `read` instead of `append`:

1. Resolve scope name (Step 1 applies identically).
2. Read `.otsumi/kb-vault/{vault-id}/events.json`. If the file does not exist, report "No events logged yet for vault `{vault-id}`." and stop.
3. Parse as JSON array. If the content is not a valid JSON array, report corruption and stop. Do not modify the file.
4. Apply filters in order:
   - `--skill {name}`: keep events where `event` starts with `{name}.` (e.g. `--skill zettelize` keeps `zettelize.completed`).
   - `--event {type}`: keep events where `event` exactly equals `{type}`.
   - `--since {YYYY-MM-DD}`: keep events where `timestamp` ≥ the given date (ISO-8601 prefix match).
   - `--last {N}`: after all other filters, return only the last N events.
   - No flags: return all events.
5. Present the filtered event set as a formatted table or structured list. Include `timestamp`, `event`, and `details` for each entry.
6. Report count of events shown vs total events in the file.

### 6. Report (append mode)

After a successful append:
- No output to the user (consistent with `core-atomic-log`'s "silent on success" contract).

After a failed append:
- Surface the failure reason once. Confirm that the parent vault operation is not blocked.

## Event Vocabulary

Canonical event types for this skill family:

| Event type | Triggering skill | Details convention |
|---|---|---|
| `remember.completed` | kb-obsidian-remember | `written={filename} bytes={n}` |
| `zettelize.completed` | kb-obsidian-zettelize | `created={n} updated={n} skipped={n} source={filename}` |
| `assemble.completed` | kb-obsidian-assemble | `resource={filename} zettels={n} gaps={n}` |
| `search.completed` | kb-obsidian-search | `query="{q}" hits={n}` |
| `lint.completed` | kb-obsidian-lint | `checks={n} findings={n} check-ids={id1,id2,...}` |
| `write.completed` | kb-obsidian-write | `output={filename} words={n}` |
| `archive.completed` | kb-obsidian-archive | `target={filename} status={status}` |
| `config.completed` | kb-obsidian-config | `vault={vault-id} graph={graph.json} status=updated` |
| `config.failed` | kb-obsidian-config | `vault={vault-id} graph={graph.json} reason={error-summary}` |

Future skills that extend this family MUST register their event type in this table.

## Scope-Name Convention

Vault skill events MUST use the prefix `kb-vault/` followed by the vault id from `system.md`:

```
kb-vault/{vault-id}
```

Example: `kb-vault/volgna-gath` → `.otsumi/kb-vault/volgna-gath/events.json`

**Namespace boundary**: dev-workflow pipeline scopes use bare feature names (e.g. `user-auth`, `chess-engine-refactor`). They MUST NOT use the `kb-vault/` prefix. Vault skill scopes MUST NOT use bare feature-name style. This boundary prevents cross-contamination between pipeline logs and vault operation logs and ensures `core-atomic-log` routing is unambiguous.

## Validation

Before claiming success:

- [ ] `vault-id` was resolved from `system.md` → `## Knowledge Bases`; nothing was hardcoded.
- [ ] Scope name is `kb-vault/{vault-id}` and does not collide with any dev-workflow pipeline scope format.
- [ ] Append was delegated entirely to `core-atomic-log`; no append logic was re-implemented.
- [ ] Event type is one of the documented vocabulary types or is explicitly disclosed as an extension.
- [ ] Details string carries signal derived from the invoking skill's return values; no values were fabricated.
- [ ] On append failure: parent operation was not blocked; failure was surfaced at most once.
- [ ] No vault files were read, written, or modified.
- [ ] No git operation was performed.
