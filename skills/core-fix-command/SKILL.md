---
name: core-fix-command
description: "Resolve command or skill lookup mismatches by finding the likely intended command and returning a safe correction."
---

# Core Fix Command

Recover from a mistyped, moved, or renamed command — return a correction without executing it.

## Usage

`/core-fix-command {failed-command}`

## Hard Rules

- NEVER execute the guessed replacement automatically.
- NEVER hide ambiguity — list every candidate that matches.
- MUST prefer exact skill folder names over fuzzy matches.
- MUST prefer prefixed canonical names over unprefixed names.

## Steps

1. Capture the failed command name and error context.
2. Search available skill folders and command references.
3. Rank candidates:
   - exact normalized match
   - prefix/domain match
   - edit distance
   - semantic match from descriptions
4. Return:
   - most likely command
   - alternatives
   - reason
   - whether user confirmation is required
