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
