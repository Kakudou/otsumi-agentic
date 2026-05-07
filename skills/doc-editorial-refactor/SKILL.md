---
name: doc-editorial-refactor
description: "Rewrite Markdown with sharp editorial voice, TL;DR, clean typography, no em dashes, and preserved technical blocks."
---

# Doc Editorial Refactor

Refactor technically correct Markdown that reads like committee sludge into sharp, direct editorial prose, without breaking technical content.

## Usage

- `/doc-editorial-refactor {file}`
- `/doc-editorial-refactor {file} --output {target}`

## Hard Rules

- NEVER rewrite headings, tables, code fences, Gherkin specs, blockquotes, horizontal rules, or list items.
- NEVER leave a single em dash in the output.
- NEVER place a bold opener at the end of a preceding line.
- NEVER produce one-sentence-per-blank-line paragraphs.
- NEVER exceed three sentences on one prose line.
- NEVER soften a verdict to sound polite.
- MUST insert TL;DR immediately after the title.
- MUST verify zero em dashes before reporting done.
- If `--output` is specified, NEVER edit the source file.

## Steps

1. Read the full source document before editing.
2. Identify:
   - title
   - section hierarchy
   - protected technical blocks
   - unusual formatting
3. Write output in this order:
   - exact same title
   - `## TL;DR` immediately after title
   - 5–10 proportional bullets
   - refactored remaining prose
4. Voice rules:
   - state verdicts plainly
   - remove hedging
   - call out problems directly
   - use short punchy closers when useful
5. Typography:
   - replace every em dash with `:`, `;`, `,`, or parentheses per context
   - target ~110 characters max per prose line
6. Verify:
   - search for `—` returns zero
   - TL;DR is immediately after title
   - protected blocks are unchanged
   - paragraphs are naturally grouped
