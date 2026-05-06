---
name: dev-python-test-generator
description: "Generate Python RED tests from approved behavior using pytest-bdd or plain pytest helper style."
---

# Dev Python Test Generator

Generate Python tests that start RED.

## Usage

`/dev-python-test-generator <payload>`

## Purpose

Create failing tests from approved behavior before implementation.

## Hard Rules

- NEVER write placeholder tests.
- NEVER fake RED by importing impossible modules unless that is the accepted surface contract.
- NEVER skip RED confirmation by the caller.
- Respect `test_style`.

## Modes

| `test_style` | Behavior |
|---|---|
| `auto` | use pytest-bdd when feature files or pytest-bdd conventions exist; otherwise plain pytest |
| `pytest` | use plain pytest with `_given/_when/_then` helper organization |

## Steps

1. Read approved scenarios or behavior contract.
2. Detect existing test layout.
3. Generate tests in the appropriate location.
4. Keep tests behavior-focused.
5. Return written files and expected RED reason.
