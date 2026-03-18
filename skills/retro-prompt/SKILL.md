---
name: retro-prompt
description: Retrospective prompt analysis. Reviews what happened during a pipeline run or any provided context and returns a better version of the original request, one that would have reached the same outcome faster, with less friction.
---

Run a retrospective analysis and produce a better starting prompt.

## Usage

- `/retro-prompt <feature-name>`, analyse a closed pipeline by feature name
- `/retro-prompt`, standalone mode, works on any task or conversation, no pipeline required

## Purpose

Look at what actually happened, identify where the original request caused friction, and return a plain natural language prompt that would have started the work more cleanly.

This is a coaching tool. It teaches better prompting by grounding feedback in real events, not generic advice.

Usable anywhere: after a pipeline closes, mid-session, after any completed task.

## Steps

1. Resolve the input mode:
   - **Feature-scoped**: read `.otsumi/<feature-name>/pipeline.json` and extract `description` as the original prompt, plus `language_id`, `stages`, `mode`. Collect all available stage output JSONs. Read `features/<feature-name>.feature` when present. Read `docs/decisions/` when present.
   - **Standalone**: work from whatever context is available in the current session, conversation history, provided files, or an explicitly stated original request. At minimum, identify the original ask.

2. Reconstruct what was actually delivered:
   - behaviors specified or discovered during the work
   - edge cases and constraints that emerged mid-run
   - scope that was suppressed, escalated, or added after the initial request
   - corrections or rework that happened and what triggered them

3. Identify friction points, each one must be grounded in something that actually happened:
   - a trap or edge case the original prompt did not anticipate (discovered during the work)
   - missing context that had to be inferred, asked, or assumed
   - a constraint that caused rework because it was not stated upfront
   - a scope ambiguity that expanded or contracted mid-work
   - an implicit assumption that turned out to be wrong or contested

4. Compose the improved prompt.

   The improved prompt is plain natural language, not a command, not a pipeline invocation, not a tool-specific format. It must work as a request in any context: a chat session, a new pipeline, a different tool entirely.

   It must:
   - be written as a clear, direct natural language request
   - include the intent rewritten with the missing boundaries, constraints, and scope that the work discovered
   - embed key edge cases and constraints as part of the description, not as footnotes
   - be specific enough that a fresh context would understand exactly what to build, for whom, and within what limits
   - not exceed what was actually delivered, do not add scope that was suppressed or out of bounds
   - not reference pipeline commands, flags, stage numbers, or tool-specific syntax

5. Score the delta between the original prompt and the improved prompt:

   | Dimension | What it measures |
   |-----------|-----------------|
   | `specificity` | how much ambiguity was removed |
   | `boundary_coverage` | how many edge cases were pre-specified |
   | `context_clarity` | whether the environment, constraints, or stack were explicit from the start |
   | `scope_discipline` | whether scope was bounded upfront instead of discovered mid-run |

   Score each 1–5. 5 = no improvement possible on that axis.

6. Present the result to User:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Retro-Prompt · <feature-name or "standalone">]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Original prompt:
  <original prompt as received>

Friction points: <n>
  • [<where it surfaced>] <type>, <what was missing> (cost: <what it caused>)

Improved prompt:
  <plain natural language, copy-pasteable into any context, any tool, any session>

Delta: specificity <n>/5 · boundaries <n>/5 · context <n>/5 · scope <n>/5 · overall <avg>/5

Coaching notes:
  • <one concrete tip per friction point, what to include next time and why>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

7. If feature-scoped: log via `/atomic-log <feature-name> retro-prompt.completed "friction_points=<n> overall_delta=<score>"`.

## Hard Rules

- Never invent friction points not grounded in actual events.
- Never rewrite the improved prompt to include scope that was not delivered.
- If the work ran clean with no friction: report that honestly. Zero friction points is a valid result.
- This command may be run at any time, not only after pipeline closure.
- The improved prompt must be plain natural language. No commands, no flags, no stage references.
