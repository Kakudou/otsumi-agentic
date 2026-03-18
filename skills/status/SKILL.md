---
name: status
description: Display the current state of a feature pipeline or list all pipelines.
---

Show the current pipeline status for a feature, or list all active pipelines.

## Usage

- `/status <feature-name>` — show detailed status for one pipeline
- `/status` — list all pipelines found in `.otsumi/`

## Steps — Feature-Scoped

1. Validate `<feature-name>`.
2. Read `.otsumi/<feature-name>/pipeline.json`.
3. If the file does not exist: report that no pipeline exists for this feature name.
4. Display:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Pipeline Status · <feature-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Mode:               <assisted | vibecoding>
Status:             <running | paused | closed | aborted>
Language:           <language_id>
Stages:             <stages list from pipeline.json>

Current Stage:      <current_stage or "none">
Last Completed:     <last_completed_stage or "none">
Next Stage:         <next_stage or "pipeline complete">

Started:            <started_at>
Last Resumed:       <resumed_at or "never">

Stages Completed:   <list of stage-XX-output.json files that exist>
Stages Remaining:   <remaining stages from pipeline>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

5. If the pipeline is paused: include a reminder that `/continue <feature-name>` will advance.
6. If the pipeline is closed: include a reminder that `/retro-prompt <feature-name>` is available.

## Steps — List All

1. Scan `.otsumi/` for all directories containing `pipeline.json`.
2. For each, read `pipeline.json` and extract `feature_name`, `mode`, `status`, `current_stage`, and `stages`.
3. Display a summary table:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[All Pipelines]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Feature | Mode | Status | Stage | Stages |
|---------|------|--------|-------|--------|
| ... | ... | ... | ... | ... |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

4. Also list any `*-planned.json` files as planned features at the bottom.

## Hard Rules

- Never modify pipeline state. This is a read-only command.
- Never invent status information. Report exactly what the files contain.