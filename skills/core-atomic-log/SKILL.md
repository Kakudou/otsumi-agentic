---
name: core-atomic-log
description: "Append a single event to an append-only JSON pipeline or workflow log without corrupting existing history."
---

# Core Atomic Log

Append exactly one event to an append-only log.

## Usage

`/core-atomic-log {scope-name} {event-type} "{details}"`

## Hard Rules

- MUST write exactly one event per invocation.
- MUST preserve existing entries.
- MUST report success only when the event was written.
- MUST report corruption explicitly when the existing file is not a JSON array.
- NEVER rewrite old events.
- NEVER silently fix corrupt JSON.
- NEVER use prose-only logs when a structured log file exists.

## Steps

1. Resolve log location: `.otsumi/{scope-name}/events.json` (workflow, pipeline, or generic scope).
2. If the file does not exist, initialize it as `[]`.
3. If the file exists but is not a JSON array, stop and report corruption.
4. Append:

```json
{
  "timestamp": "{ISO-8601}",
  "event": "{event-type}",
  "details": "{details}"
}
```

5. Write atomically when the host environment supports it.
6. Return only a compact success or failure message.

## Required Output Format Rules

- Canonical log filename is `.otsumi/$1/events.json`.
- The file content MUST be a valid JSON array `[...]` containing event objects.
- NEVER write a `.txt` file.
- NEVER write a single JSON object without the enclosing array.
- NEVER use `create` mode on a file that already exists; use `edit` or `overwrite` with the full array content.
- Keep `events.json` as canonical in this architecture.
- Migration note: old references to `.otsumi/$1/atomic-log.json` should be updated to `.otsumi/$1/events.json`.

## Required Summary Field

- `$3` is a required meaningful summary string built by the caller from stage output data.
- It must carry signal, not a timestamp, not a label, not empty.

## Detailed 7-Step Procedure

1. **Validate all inputs with exact errors**
   - If `$1` is empty: `Missing feature name. Usage: /core-atomic-log <scope-name> <event> <summary>`
   - If `$2` is empty: `Missing event. Usage: /core-atomic-log <scope-name> <event> <summary>`
   - If `$3` is empty: `Missing summary. The caller must provide a meaningful summary built from stage output data.`
2. **Verify pipeline directory exists**
   - Confirm `.otsumi/$1/` exists.
   - If missing: `No pipeline found for '$1'. Start one with /start-pipeline.`
3. **Read existing log and parse as array**
   - Read `.otsumi/$1/events.json`.
   - If file does not exist, begin from `[]`.
   - If file exists but is empty/corrupt/not a JSON array, stop and report append-only log invalid; do not silently reset.
4. **Build exact JSON entry**
   ```json
   {
     "timestamp": "<ISO-8601 UTC>",
     "event": "$2",
     "details": "$3"
   }
   ```
5. **Append (never replace)**
   - Append the new object to the end of the existing array.
   - Do NOT replace the array.
6. **Write full array back using overwrite mode**
   - Write full updated array to `.otsumi/$1/events.json` using overwrite mode.
   - Preserve all prior entries.
7. **Self-check before finishing**
   - Confirm the file you wrote is `.otsumi/$1/events.json` (not `.txt`, not a different path).
   - Confirm the content is a JSON array `[...]` with all prior entries preserved.
   - If either check fails: fix it immediately before returning.

## Before/After Append Example

Do NOT replace the array. Append to it.

Before:
```json
[
  { "timestamp": "2025-01-01T00:00:00Z", "event": "stage-01.completed", "details": "scenarios=3 approved=3" }
]
```

After append:
```json
[
  { "timestamp": "2025-01-01T00:00:00Z", "event": "stage-01.completed", "details": "scenarios=3 approved=3" },
  { "timestamp": "2025-01-15T12:00:00Z", "event": "$2", "details": "$3" }
]
```

## Silent on Success

No output to the user unless an error occurred.
