---
name: dev-python-refactorer
description: "Perform safe Python refactors as atomic behavior-preserving changes with test validation after each change."
---

# Dev Python Refactorer

Improve Python structure while preserving behavior.

## Usage

`/dev-python-refactorer <payload>`

## Hard Rules

- NEVER change behavior intentionally.
- NEVER refactor without GREEN tests as a baseline when tests exist.
- One refactor step MUST be atomic and reversible.
- If tests fail after a refactor step, MUST revert or stop.

## Steps

1. Establish baseline GREEN evidence.
2. Identify small refactor opportunities.
3. Apply one atomic change at a time.
4. Run focused tests after each change.
5. Return changes, rationale, and test evidence.
