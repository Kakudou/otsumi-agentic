---
name: backlog
description: Display all planned features from the Feature Backlog Protocol.
---

List all planned features that were escalated from gold plating or queued for future work.

## Usage

`/backlog`

## Steps

1. Scan `.otsumi/` for all files matching `*-planned.json`.
2. For each file, read and extract:
   - `feature_name`
   - `status`
   - `description`
   - `origin`
   - `source_feature`
   - `source_stage`
   - `planned_at`
3. Sort by `planned_at` descending (most recent first).
4. Display:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Feature Backlog]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Planned features: <n>

  1. <feature-name>
     Description: <description>
     Origin: <origin> from <source_feature> (Stage <source_stage>)
     Planned: <planned_at>
     Start: /start-pipeline --mode assisted --lang <lang> <description>

  ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

5. If no planned features exist: report that the backlog is empty.
6. For each planned feature, suggest a `/start-pipeline` command. The `--lang` selector should be suggested based on the source feature's pipeline if available, otherwise left as a placeholder for User to fill.

## Hard Rules

- Never modify planned feature files. This is a read-only command.
- Never auto-start a planned feature. The user decides when to start.
- Never invent planned features. Report exactly what the files contain.