---
name: dev-python-refactorer
description: "Perform safe Python refactors as atomic behavior-preserving changes with test validation after each change."
---

# Dev Python Refactorer

Refactor Python safely.

## Usage

`/dev-python-refactorer <payload>`

## Purpose

Improve structure while preserving behavior.

## Hard Rules

- NEVER change behavior intentionally.
- NEVER refactor without GREEN tests as a baseline when tests exist.
- One refactor step should be atomic and reversible.
- If tests fail after a refactor, revert or stop.

## Steps

1. Establish baseline GREEN evidence.
2. Identify small refactor opportunities.
3. Apply one atomic change at a time.
4. Run focused tests after each change.
5. Return changes, rationale, and test evidence.
