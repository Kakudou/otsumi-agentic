---
name: create-skill
description: Distills a completed task, action, or conversation into a reusable skill. Runs retro-prompt to extract what worked and what caused friction, then runs prompt-master to convert the improved prompt into a clean SKILL.md ready for future use. Use after any completed work to capture learned patterns as durable, reusable automation.
---

# Create Skill

Turn the outcome of a completed task into a reusable skill file.

## Usage

- `/create-skill <feature-name>` — distill a closed pipeline into a skill
- `/create-skill <skill-name> "<description>"` — distill the current conversation/task into a skill with the given name
- `/create-skill` — interactive mode, asks what to capture

## Hard Rules

- NEVER skip Phase 1. The retro-prompt grounds the skill in real experience, not hypothetical patterns.
- NEVER generate a skill from a task that has not been completed. The retrospective requires actual outcomes.
- NEVER embed raw friction points as-is. Transform them into preventive constraints within the skill steps.
- NEVER create a skill that duplicates an existing skill. Check `skills/` for conflicts before writing.
- The skill body MUST be self-contained. A future invocation MUST NOT require conversation context to execute.
- ALWAYS let User review and approve before writing any files.

## Purpose

Captures task knowledge as a durable, reusable skill by chaining two skills:
1. **`/retro-prompt`** — extracts friction points, edge cases, and an improved natural-language prompt.
2. **`/prompt-master`** — converts that improved prompt into a production-ready, tool-optimized skill definition.

## Steps

### Phase 1 — Retrospective (retro-prompt)

1. Invoke `/retro-prompt`:
   - If `<feature-name>` provided: run feature-scoped retro-prompt.
   - Otherwise: run standalone retro-prompt on the current session context.

2. Capture the full retro-prompt output:
   - `original_prompt`, `friction_points`, `improved_prompt`, `delta_scores`, `coaching_notes`

3. Present retro-prompt output to User:
   - **approve** → proceed to Phase 2
   - **edit** → apply changes, then proceed to Phase 2
   - **abort** → halt; retro-prompt output remains available in conversation

### Phase 2 — Skill Generation (prompt-master)

4. Determine skill metadata:
   - `skill_name`: derive from feature-name or prompt User for a kebab-case name
   - `skill_description`: one-line summary
   - `target_tool`: "Claude Code" unless User specifies otherwise

5. Invoke `/prompt-master` with:
   - The improved prompt from Phase 1
   - Target tool: resolved target (default: Claude Code)
   - Additional context: friction points and coaching notes as constraints to embed

6. Reshape prompt-master output into SKILL.md structure:

   ```markdown
   ---
   name: <skill-name>
   description: <one-line description>
   ---

   <skill body — production-ready prompt restructured with:
    Usage, Purpose, Steps, Hard Rules sections>
   ```

7. Apply SKILL.md conventions:
   - `## Usage` with invocation syntax
   - `## Purpose` explaining when and why
   - `## Steps` with numbered execution steps
   - `## Hard Rules` with MUST/NEVER imperatives
   - Embed friction-point lessons as constraints within steps, NEVER as footnotes

### Phase 3 — Review and Write

8. Present the complete SKILL.md to User:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Create Skill · <skill-name>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<full SKILL.md content>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Origin: <feature-name or "conversation">
Friction points captured: <n>
Delta score: <overall>/5

Actions: [approve] [edit] [abort]
```

9. On **approve**: create `skills/<skill-name>/`, write `skills/<skill-name>/SKILL.md`, report `/<skill-name>`.
10. On **edit**: apply changes, re-present for approval.
11. On **abort**: write no files; report retro-prompt output is still available.
