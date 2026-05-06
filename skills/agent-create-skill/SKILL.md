---
name: agent-create-skill
description: "Create a reusable SKILL.md from completed work by mandatory chaining agent-retro-prompt then agent-prompt-master."
---

# Agent Create Skill

Turn completed work into a reusable skill file by chaining retro-prompt then prompt-master.

## Usage

- `/agent-create-skill <feature-name>` — distill a closed or available pipeline into a skill
- `/agent-create-skill <skill-name> "<completed work summary>"` — distill the current task or conversation into the named skill
- `/agent-create-skill` — interactive mode; ask what completed work to capture

## Hard Rules

- MUST run Phase 1 (retro-prompt) — NEVER skip; the retrospective grounds the skill in actual outcomes.
- MUST run Phase 2 (prompt-master) — NEVER skip; normalization is mandatory.
- MUST present the draft for user review before writing files.
- MUST check `skills/` for conflicts before writing — NEVER duplicate an existing skill.
- MUST transform friction points into preventive constraints inside steps — NEVER embed them as raw footnotes.
- MUST make the skill body self-contained — future invocation MUST NOT require conversation context.
- NEVER generate a skill from work that has not been completed.

## Steps

### Phase 1 — Retrospective

1. Invoke `/agent-retro-prompt`:
   - feature-scoped if `<feature-name>` is provided
   - otherwise standalone on current task/conversation
2. Capture the complete output:
   - `original_prompt`
   - `friction_points`
   - `improved_prompt`
   - `delta_scores`
   - `coaching_notes`
3. Present the retrospective and request: `approve` / `edit` / `abort`.

### Phase 2 — Prompt-Master Conversion

4. Resolve metadata:
   - `skill_name`: kebab-case, domain-prefixed when appropriate
   - `skill_description`: one-line trigger/purpose
   - `target_tool`: default to host skill format unless user specifies another
5. Invoke `/agent-prompt-master` with:
   - the improved prompt from Phase 1
   - friction points converted into constraints
   - coaching notes converted into hard rules or validation steps
   - requested skill name and target tool
6. Reshape into the canonical layout — Hard Rules MUST sit in the first ~30% of the body so critical constraints are read first:

```markdown
---
name: <skill-name>
description: "<when to use this skill>"
---

# <Human Title>

<one-sentence identity/purpose in second person>

## Usage
...

## Hard Rules

- MUST ... — NEVER ...
- NEVER ...

## Steps
...

## Validation
...
```

Layout rules:
- Frontmatter only carries `name` and `description`.
- Title is followed immediately by one identity sentence — no separate `## Purpose` block.
- `## Hard Rules` MUST appear before `## Steps`.
- Use MUST / NEVER / MAY consistently — NEVER soften with "should" or "avoid".
- Drift/boundary tables, JSON contracts, and output schemas come AFTER Steps.

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

8. On `approve`: create `skills/<skill-name>/SKILL.md`.
9. On `edit`: apply changes and present again.
10. On `abort`: write nothing; report that retro-prompt output remains available.
