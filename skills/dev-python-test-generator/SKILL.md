---
name: dev-python-test-generator
description: "Generate Python RED tests from approved behavior using pytest-bdd or plain pytest helper style."
---

# Dev Python Test Generator

Generate Python tests that start RED, from approved behavior, before implementation.

## Usage

`/dev-python-test-generator {payload}`

## Hard Rules

- NEVER write placeholder tests.
- NEVER fake RED by importing impossible modules unless that is the accepted surface contract.
- NEVER skip caller RED confirmation.
- MUST respect `test_style`.

## Modes

| `test_style` | Behavior |
|---|---|
| `auto` | Use pytest-bdd when feature files or pytest-bdd conventions exist; otherwise plain pytest |
| `pytest` | Use plain pytest with `_given/_when/_then` helper organization |

## Steps

1. Read approved scenarios or behavior contract.
2. Detect existing test layout.
3. Generate tests in the appropriate location.
4. Keep tests behavior-focused.
5. Return written files and the expected RED reason.
