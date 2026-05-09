---
name: dev-python-refactorer
description: "Perform safe Python refactors as atomic behavior-preserving changes with test validation after each change."
---

# Dev Python Refactorer

Improve Python structure while preserving behavior.

## Usage

`/dev-python-refactorer {payload}`

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

## Refactor Taxonomy

### Allowed refactors

- Rename symbols (variable, function, class, module) without behavior change.
- Extract method or extract class.
- Inline method or variable when behavior is preserved.
- Move function/class/module to a better location without API behavior change.
- Simplify conditional structure without changing outcomes.
- Replace magic constants with named constants.
- Add type hints to previously untyped code.
- Split oversized modules into smaller modules.

### Forbidden refactors

- Any behavior change.
- Any public API signature change.
- Any dependency addition.
- Any test modification as part of refactor execution.
- Any speculative helper creation not required by the atomic refactor.
- Batching multiple refactors into one change set.

## Per-Change Test Cycle Protocol

- Fuhyō applies ONE atomic refactor step.
- Fuhyō runs focused tests via `dev-run-tests` immediately after that single step.
- If tests are GREEN, proceed to the next atomic step.
- If tests are RED/ERROR, revert immediately and record the event in `aborted_refactors`.
- Never apply multiple refactor steps between test runs.
- This per-change cycle is the mandatory safety net; Kakugyō must not collapse it.

## Output Contract Addendum

Return result data with an `aborted_refactors` array:

```json
{
  "aborted_refactors": [
    {
      "attempted_change": "string",
      "revert_reason": "string",
      "test_evidence": "string"
    }
  ]
}
```

- Fuhyō MUST populate `aborted_refactors` honestly.
- An empty array is valid only when zero reverts occurred.

## Architecture Preservation Rule

- Preserve or improve existing architectural boundaries (including port/adapter and dependency inversion style boundaries).
- Never introduce cross-layer imports.
- Never collapse an existing boundary.
- If the repository already uses a specific architecture pattern, follow that pattern rather than imposing a new one.
- Any refactor that degrades architecture is forbidden.

## Halt Gates (Hard)

1. Before any refactor work, run baseline tests.
   - If baseline is not GREEN, halt immediately and report status to Ōshō.
   - Do not perform refactors on RED/ERROR baseline.
2. During refactors, enforce the per-change GREEN gate via `dev-run-tests`.
3. After all refactors, run the full suite again via `dev-run-tests`.
   - If final run is not GREEN, halt and report to Ōshō before producing completion output.
4. If quality checks are requested by the invoking flow, run `dev-quality-check` after tests and report results.
5. Stage closure belongs to workflow control (`flow-complete-stage`), not to this skill contract.
