---
name: otsumi-doc-refacto
description: Rewrite any markdown document with a sharp, direct editorial voice, a TL;DR section, clean typography, and natural paragraph formatting.
---

## Usage

- `/otsumi-doc-refacto <path/to/doc.md>` — rewrite in-place
- `/otsumi-doc-refacto <path/to/doc.md> --output <path/to/out.md>` — write refactored version to a new file

## Purpose

Takes any markdown document — technical analysis, architecture notes, decision records, research, tutorials — and produces a version that:
- Drops hedging; states verdicts plainly
- Adds a proportional TL;DR section at the top
- Removes all em dashes in favor of contextually correct punctuation
- Reflows prose into natural editorial paragraphs

Use when a document is technically correct but reads like it was written by committee, or when you want a consistent sharp editorial voice.

## Steps

1. Read the full source document before touching anything. Note:
   - Title (line 1), section hierarchy
   - Technical blocks that MUST NOT change: tables, code fences, Gherkin, diagrams, list items, PEP references, verdict/arrow lines
   - Whether the source has unusual structure (one sentence per blank-line paragraph is common in AI-generated docs; detect before reflowing)

2. Write the output file (or edit in-place) in this order:
   a. **Title** — keep exactly as-is
   b. **TL;DR section** — insert immediately after the title, before any subtitle or author line. 5–10 bullets proportional to doc length. Format: `- **Bold decision or finding.** 1–2 supporting sentences. No padding.`
   c. **Remaining sections** — rewrite prose with voice and formatting rules below. Leave all technical blocks untouched.

3. **Voice rules** (every prose sentence):
   - State verdicts plainly: "This is wrong", not "This may present challenges"
   - Call out problems explicitly: "This breaks in production", not "This could potentially have issues"
   - Remove hedging: cut "might", "could potentially", "it's worth noting", "generally speaking"
   - Short punchy closers for key conclusions — a one-sentence paragraph is fine
   - Never soften a finding to sound polite

4. **Typography rules** (whole document):
   - Replace every em dash (—) with a contextually correct alternative:
     - `:` when introducing a list, definition, or explanation
     - `;` when joining two independent clauses
     - `,` for light connective breaks
     - `(...)` for inline parenthetical asides
   - Verify with a search before finishing. Zero em dashes in output.

5. **Paragraph formatting rules** (prose blocks only):
   - Related sentences supporting the same point: same paragraph, no blank line between them
   - Distinct ideas or argument shifts: new paragraph, blank line between
   - Long explanatory sentences (>90 chars) may stand alone
   - Short punchy conclusions and verdicts stand alone
   - Bold openers (`**Word...`) ALWAYS start a fresh line; never append to the end of a preceding sentence
   - Target ~110 chars max per prose line
   - Do NOT put every sentence on its own line
   - Do NOT exceed 3 sentences per line

6. **Never touch:**
   - Headings (any level)
   - Tables, code fences, blockquotes, horizontal rules
   - Ordered and unordered list items
   - Verdict/arrow lines (e.g., `**→ Use UV.**`)
   - The source file if `--output` was specified

7. Verify before reporting done:
   - Search for `—` in output: must return 0 matches
   - Confirm TL;DR exists immediately after the title
   - Spot-check 3 prose paragraphs: no single-sentence-per-blank-line structure remains
   - All code blocks, tables, and Gherkin specs are intact

## Hard Rules

- NEVER rewrite headings, tables, code blocks, Gherkin specs, or list items
- NEVER leave a single em dash in the output
- NEVER put a bold opener (**Word) at the end of a preceding line
- NEVER produce one-sentence-per-blank-line paragraphs — most common failure mode
- NEVER exceed 3 sentences on one prose line
- NEVER soften a verdict or finding to sound polite
- ALWAYS insert TL;DR immediately after the title, not at the end
- ALWAYS verify 0 em dashes before reporting done
