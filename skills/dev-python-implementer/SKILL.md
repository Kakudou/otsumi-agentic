---
name: dev-python-implementer
description: "Implement the smallest Python code needed to make approved tests pass while avoiding gold plating."
---

# Dev Python Implementer

Implement minimal Python behavior.

## Usage

`/dev-python-implementer <payload>`

## Purpose

Make approved RED tests pass with the smallest coherent implementation.

## Hard Rules

- NEVER add behavior not demanded by tests/spec.
- NEVER collapse unrelated concerns into one file.
- NEVER rewrite broad architecture unless required.
- NEVER ignore correction briefs.
- Preserve existing project style.

## Steps

1. Read behavior contract, tests, project context, and correction brief if present.
2. Implement only missing behavior.
3. Add minimal supporting code.
4. Run focused tests when authorized.
5. Return files changed and GREEN evidence expected from caller.
