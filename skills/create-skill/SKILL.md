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

## Purpose

After a task is done, the knowledge of *how* it was done lives only in conversation history — which disappears. This command captures that knowledge as a durable, reusable skill.

It chains two existing skills:
1. **`/retro-prompt`** — extracts friction points, edge cases, and the improved natural-language prompt from what actually happened.
2. **`/prompt-master`** — converts that improved prompt into a production-ready, tool-optimized skill definition.

The result is a `SKILL.md` file that encodes the lessons learned, ready to be invoked next time the same kind of work comes up.

## Steps

### Phase 1 — Retrospective (retro-prompt)

1. Invoke `/retro-prompt` in the appropriate mode:
   - If a `<feature-name>` was provided: run feature-scoped retro-prompt.
   - Otherwise: run standalone retro-prompt on the current session context.

2. Capture the full retro-prompt output:
   - `original_prompt` — what was originally asked
   - `friction_points` — what caused rework, ambiguity, or wasted effort
   - `improved_prompt` — the rewritten natural-language request
   - `delta_scores` — specificity, boundary_coverage, context_clarity, scope_discipline
   - `coaching_notes` — concrete tips per friction point

3. Present the retro-prompt output to User for review. User may:
   - **approve** — proceed to Phase 2
   - **edit** — modify the improved prompt before proceeding
   - **abort** — stop here, retro-prompt output is still useful on its own

### Phase 2 — Skill Generation (prompt-master)

4. Determine the skill metadata:
   - `skill_name`: derive from feature-name or ask User for a kebab-case name
   - `skill_description`: one-line summary of what the skill does
   - `target_tool`: "Claude Code" (Agentic AI category) unless User specifies otherwise

5. Invoke `/prompt-master` with:
   - The improved prompt from Phase 1 as the input
   - Target tool: the resolved target (default: Claude Code)
   - Additional context: the friction points and coaching notes as constraints to embed

6. Take the prompt-master output and reshape it into a SKILL.md structure:

   ```markdown
   ---
   name: <skill-name>
   description: <one-line description>
   ---

   <skill body — the production-ready prompt from prompt-master,
    restructured with SKILL.md conventions:
    Usage section, Purpose section, Steps section, Hard Rules section>
   ```

7. Apply SKILL.md conventions to the generated content:
   - Add a `## Usage` section with invocation syntax
   - Add a `## Purpose` section explaining when and why to use this skill
   - Convert the prompt body into a `## Steps` section with numbered execution steps
   - Extract constraints and anti-patterns into a `## Hard Rules` section
   - Embed friction-point lessons as constraints within the steps, not as footnotes

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

9. On **approve**:
   - Create directory `skills/<skill-name>/`
   - Write `skills/<skill-name>/SKILL.md`
   - Report the invocation command: `/<skill-name>`

10. On **edit**:
    - Apply User's changes
    - Re-present for approval

11. On **abort**:
    - Do not write any files
    - Report that the retro-prompt output is still available in the conversation

## Hard Rules

- Never skip Phase 1. The retro-prompt grounds the skill in real experience, not hypothetical patterns.
- Never generate a skill from a task that has not been completed. The retrospective needs actual outcomes to analyse.
- Never embed raw friction points as-is. Transform them into preventive constraints within the skill steps.
- Never create a skill that duplicates an existing skill. Check `skills/` for conflicts before writing.
- The skill body must be self-contained. A future invocation must not require context from the conversation that created it.
- Always let User review and approve before writing. This is a durable artifact, not a throwaway.
