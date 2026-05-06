---
name: core-atomic-log
description: "Append a single event to an append-only JSON pipeline or workflow log without corrupting existing history."
---

# Core Atomic Log

Append one event to an append-only log.

## Usage

`/core-atomic-log <scope-name> <event-type> "<details>"`

## Purpose

Maintain trustworthy execution history. This skill writes exactly one event, preserves existing entries, and refuses to hide corruption.

## Hard Rules

- NEVER rewrite old events.
- NEVER silently fix corrupt JSON.
- NEVER claim success if the event was not written.
- NEVER use prose-only logs when a structured log file exists.
- One invocation writes exactly one event.

## Steps

1. Resolve the log location:
   - workflow/pipeline scope: `.otsumi/<scope-name>/events.json`
   - generic scope: `.otsumi/<scope-name>/events.json`
2. If the file does not exist, initialize it as `[]`.
3. If the file exists but is not a JSON array, stop and report corruption.
4. Append:

```json
{
  "timestamp": "<ISO-8601>",
  "event": "<event-type>",
  "details": "<details>"
}
```

5. Write atomically when the host environment supports it.
6. Return only a compact success or failure message.
