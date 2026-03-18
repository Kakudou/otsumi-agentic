---
name: atomic-log
description: Append an event entry to the atomic log of a feature pipeline. Lightweight, no agent spawn.
---

Append an entry to `.otsumi/$1/atomic-log.json` by using the commands '/atomic-log' and passing the feature name, event type, and summary.

`$3` is a **required** meaningful summary string built by the caller from the stage output data.
It must carry signal, not a timestamp, not a label, not empty.
Example: `"scenarios=3 approved=3 rejected=0 file=features/user-login.feature"`

---

## CRITICAL: Output Format

The atomic log is a **JSON array** stored in a file with the `.json` extension.

- The file MUST be named `atomic-log.json`, never `atomic-log.txt`, never any other extension.
- The file content MUST be a valid JSON array `[...]` containing objects.
- Every write MUST preserve all existing entries in the array.
- You are APPENDING to a JSON array, not writing lines to a text file.

**NEVER** do any of the following:
- Write a `.txt` file
- Write plain text, YAML, or any non-JSON format
- Write a single JSON object without the enclosing array
- Overwrite the file with only the new entry
- Use `create` mode on a file that already exists, use `edit` or `overwrite` with the full array

---

## Steps

1. **Validate inputs:**
   - `$1` (feature-name) must not be empty. If empty, respond:
     > Missing feature name. Usage: `/atomic-log <feature-name> <event> <summary>`
     Then stop.
   - `$2` (event) must not be empty. If empty, respond:
     > Missing event. Usage: `/atomic-log <feature-name> <event> <summary>`
     Then stop.
   - `$3` (summary) must not be empty. If empty, respond:
     > Missing summary. The caller must provide a meaningful summary built from stage output data.
     Then stop.

2. **Verify pipeline directory exists:**
   Confirm `.otsumi/$1/` exists. If it does not, respond:
   > No pipeline found for `$1`. Start one with `/start-pipeline`.
   Then stop.

3. **Read the existing log:**
   Read `.otsumi/$1/atomic-log.json`.
   - If the file **does not exist yet**: start with an empty array `[]`.
   - If the file **exists**: read its full content and parse it as a JSON array.
   - If the file exists but is empty or corrupt: halt and report that the append-only log is invalid. Do NOT reset it silently and do NOT discard history.

4. **Build the new entry:**
   Construct exactly this JSON object:
   ```json
   {
     "timestamp": "<ISO-8601 UTC>",
     "event": "$2",
     "details": "$3"
   }
   ```

5. **Append the entry to the array:**
   Add the new object to the END of the existing array. Do NOT replace the array.

   Example, if the file currently contains:
   ```json
   [
     { "timestamp": "2025-01-01T00:00:00Z", "event": "stage-01.completed", "details": "scenarios=3 approved=3" }
   ]
   ```
   After appending, the file MUST contain:
   ```json
   [
     { "timestamp": "2025-01-01T00:00:00Z", "event": "stage-01.completed", "details": "scenarios=3 approved=3" },
     { "timestamp": "2025-01-15T12:00:00Z", "event": "$2", "details": "$3" }
   ]
   ```

6. **Write the full updated array back to `.otsumi/$1/atomic-log.json`:**
   - Use `overwrite` mode (not `create`, not `edit` for a partial patch).
   - The file content MUST be the complete JSON array with ALL previous entries plus the new one.
   - The file extension MUST remain `.json`.

7. **Self-check before finishing:**
   - Confirm the file you wrote is `.otsumi/$1/atomic-log.json` (not `.txt`, not a different path).
   - Confirm the content is a JSON array `[...]` with all prior entries preserved.
   - If either check fails: fix it immediately before returning.

8. **Silent on success.** No output to the user unless an error occurred.
