---
name: agent-retro-prompt
description: "Retrospective prompt analysis. Reviews a completed pipeline, task, or conversation and returns a better natural-language starting prompt grounded in actual events."
---

# Agent Retro Prompt

Run a retrospective analysis and produce a better starting prompt.

## Usage

- `/agent-retro-prompt <feature-name>` — analyze a closed or available pipeline by feature name
- `/agent-retro-prompt` — standalone mode; analyze the current task, conversation, files, or provided context

## Purpose

Ground prompt improvement in what actually happened, not generic advice. Identify friction in the original request and produce a clearer natural-language prompt that would have reached the same delivered outcome faster, with less ambiguity and less rework.

## Hard Rules

- NEVER invent friction points not grounded in actual events.
- NEVER rewrite the improved prompt to include scope that was not delivered.
- The improved prompt MUST be plain natural language; no commands, flags, stage numbers, or tool-specific syntax.
- Zero friction points is valid. Report it honestly when the work ran clean.
- This skill MAY be invoked at any time, not only after pipeline closure.

## Steps

1. Resolve input mode:
   - **Feature-scoped**: read `.otsumi/<feature-name>/pipeline.json`, extract `description`, `language_id`, `stages`, `mode`, and all available stage output JSONs. Read `features/<feature-name>.feature` and `docs/decisions/` when present.
   - **Standalone**: work from session context, conversation history, provided files, or an explicitly stated original request. Identify the original ask at minimum.
2. Reconstruct what was actually delivered:
   - behaviors specified or discovered
   - edge cases and constraints that emerged
   - scope that was suppressed, escalated, or added
   - corrections or rework and what triggered them
3. Identify friction points. Each friction point MUST map to an actual event:
   - missing edge case
   - missing context
   - unstated constraint
   - scope ambiguity
   - wrong or contested assumption
4. Compose the improved prompt:
   - clear, direct natural language
   - includes missing boundaries and constraints
   - specific enough for a fresh context
   - does not exceed delivered scope
   - does not reference pipeline commands or tools
5. Score the delta between original and improved prompt:

| Dimension | Measures |
|---|---|
| `specificity` | ambiguity removed |
| `boundary_coverage` | edge cases and constraints made explicit |
| `context_clarity` | environment, stack, or constraints explained |
| `scope_discipline` | scope bounded upfront |

Score each 1–5. 5 means no improvement was needed on that axis.

## Output

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Prompt · <source>]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Original prompt:
<text>

Friction points:
• [<type>] <event-grounded friction> (cost: <effect>)

Improved prompt:
<plain natural-language prompt>

Delta:
specificity <n>/5 · boundaries <n>/5 · context <n>/5 · scope <n>/5 · overall <n>/5

Coaching notes:
• <note>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If feature-scoped, log completion through the available append-only log mechanism:
`retro-prompt.completed friction_points=<n> overall_delta=<n>`.
