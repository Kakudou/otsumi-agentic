---
name: diff-stage
description: Show what changed between stages in a pipeline run.
---

Display a diff of what changed at a specific pipeline stage, or between two stages.

## Usage

- `/diff-stage <feature-name> <stage-id>` — show what this stage produced or changed
- `/diff-stage <feature-name> <stage-id-a> <stage-id-b>` — compare artifacts between two stages

## Steps — Single Stage

1. Validate `<feature-name>` and `<stage-id>`.
2. Read `.otsumi/<feature-name>/<stage-id>-output.json`.
3. If the file does not exist: halt and report that this stage has not been completed yet.
4. Display:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Stage Diff · <stage-id> · <feature-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Completed by: <agent | user>
Completed at: <completed_at or verified_at>

Files written or modified:
  • <list from stage output JSON — feature files, test files, src files, decision records, docs>

Key outputs:
  • <stage-specific summary derived from the output JSON>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

5. For implementation stages (stage-04, stage-05): also show the list of files changed and a summary of what was implemented or refactored.
6. For quality stages (stage-07): show the scorecard and verdict.
7. For decision stages (stage-06): show whether records were written and their paths.

## Steps — Two-Stage Comparison

1. Read both stage output JSONs.
2. Compare file lists between the two stages.
3. Display files added, removed, or modified between them.
4. Display key metric changes (e.g., test count, score changes).

## Hard Rules

- Never modify any pipeline state or artifacts. This is a read-only command.
- Never invent file contents or diff information. Report exactly what the output JSONs contain.
- If a stage output has no file list, report that file tracking is not available for that stage.