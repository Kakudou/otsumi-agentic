---
name: agent-create-skill
description: "Create a reusable SKILL.md from completed work by mandatory chaining agent-retro-prompt then agent-prompt-master."
---

# Agent Create Skill

Turn the outcome of completed work into a reusable skill file.

## Usage

- `/agent-create-skill <feature-name>` — distill a closed or available pipeline into a skill
- `/agent-create-skill <skill-name> "<completed work summary>"` — distill the current task or conversation into the named skill
- `/agent-create-skill` — interactive mode; ask what completed work to capture

## Purpose

Capture task knowledge as durable, reusable automation by chaining two mandatory phases:

1. `/agent-retro-prompt` — extracts actual friction points, edge cases, and an improved natural-language prompt.
2. `/agent-prompt-master` — converts the improved prompt into a clean, production-ready SKILL.md.

## Hard Rules

- NEVER skip Phase 1. The retrospective grounds the skill in actual outcomes.
- NEVER skip Phase 2. Prompt-master normalization is mandatory.
- NEVER generate a skill from work that has not been completed.
- NEVER embed raw friction points as footnotes. Transform them into preventive constraints inside the skill steps.
- NEVER create a skill that duplicates an existing skill. Check `skills/` for conflicts before writing.
- The skill body MUST be self-contained. Future invocation MUST NOT require conversation context.
- ALWAYS present the draft for user review before writing files.

## Steps

### Phase 1 — Retrospective

1. Invoke `/agent-retro-prompt`:
   - if `<feature-name>` is provided, use feature-scoped mode;
   - otherwise use standalone mode on the current completed task/conversation.
2. Capture the complete output:
   - `original_prompt`
   - `friction_points`
   - `improved_prompt`
   - `delta_scores`
   - `coaching_notes`
3. Present the retrospective result and request one of:
   - `approve`
   - `edit`
   - `abort`

### Phase 2 — Prompt-Master Conversion

4. Resolve metadata:
   - `skill_name`: kebab-case, domain-prefixed when appropriate
   - `skill_description`: one-line trigger/purpose summary
   - `target_tool`: default to the host skill format unless the user specifies another
5. Invoke `/agent-prompt-master` with:
   - the improved prompt from Phase 1
   - friction points converted into constraints
   - coaching notes converted into hard rules or validation steps
   - requested skill name and target tool
6. Reshape output into the canonical skill layout:

```markdown
---
name: <skill-name>
description: "<when to use this skill>"
---

# <Human Title>

## Usage
...

## Purpose
...

## Steps
...

## Validation
...

## Hard Rules
...
```

### Phase 3 — Review and Write

7. Present the full draft:

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Create Skill · <skill-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<full SKILL.md draft>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Origin: <feature/task/conversation>
Friction points captured: <n>
Delta score: <overall>/5
Actions: [approve] [edit] [abort]
```

8. On `approve`, create `skills/<skill-name>/SKILL.md`.
9. On `edit`, apply changes and present again.
10. On `abort`, write no files and report that the retro-prompt output remains available.
