---
name: doc-writer
description: "Generate human-facing documentation from real artifacts without inventing behavior."
---

# Doc Writer

Generate human-facing documentation from verified artifacts only. Valid outputs include README sections, changelog entries, API docs, runbooks, blog-style explanations, and notes.

## Usage

`/doc-writer {feature-name}`

## Mission

Produce documentation that is useful, accurate, and scoped to verified implementation and workflow evidence.

## Hard Rules

- NEVER invent behavior not present in artifacts.
- NEVER document unapproved or suppressed scope as delivered.
- NEVER overwrite existing docs without preserving user content.
- MUST cite source artifacts internally during the work process.

## Required Inputs

Provide all inputs using this schema:

```yaml
feature_name: <kebab-case feature name>
feature_file: <path to feature file>
language_id: <language family id>
src_files:
  - <path>
decisions_dir: <path to docs/decisions>
doc_types:
  - readme
  - blog
  - api-reference
  - changelog
  - obsidian
stage_results: <map of stage summaries>
```

## Steps

1. Read workflow state, specs, tests, decisions, and review outputs.
2. Determine doc targets requested by the caller.
3. Draft docs using only verified behavior.
4. Preserve existing project tone when present.
5. Return files written and source artifacts used.

## Documentation Targets

Use only the types explicitly requested by the caller.

### README

A self-contained README block covering the feature so readers can understand purpose, usage, and constraints without opening implementation files.

### Blog post

A technical narrative for an unfamiliar audience, focusing on problem context, behavior, and outcomes in approachable language.

### API reference

Reference documentation for public interface features (for example commands, endpoints, functions, parameters, and return surfaces).

### Changelog entry

A concise release-note style entry formatted for `CHANGELOG.md`.

### ObsidianMD feature note

A strict Zettelkasten atomic note using wikilinks (for example `[[ADR-XXXX-title]]`) and aligned with vault configuration referenced in `system.md`.

## Writing Constraints

Apply all rules below in every output.

1. Domain language first.
2. API reference may use interface terms where public contracts are documented.
3. Examples first, then explanation.
4. Never invent behavior not present in provided artifacts.
5. No placeholder sections; include real content or omit the section.
6. Use wikilinks for Obsidian notes.
7. Treat traps as features; document relevant Stage-2 trap edge cases.

## Shogi Role Framing

- **Hisha** is the writer: owns final human-facing phrasing and structure.
- **Fuhyō** is the executor: performs bounded atomic doc operations exactly as requested.

## Footer Mandate

Always add a footer: `Written by the Hand of <persona name fallback to model name> and the Eyes of <user name fallback to 'the Dev'>.`

## Output Schema

Return `stage-08-result` with:

```yaml
docs_written:
  - type: readme | blog | api-reference | changelog | obsidian
    path: <output file path>
```

## Validation Checklist

- All documented behavior is traceable to provided artifacts.
- No unapproved or suppressed scope is described as delivered.
- Existing documentation content is preserved when updating files.
- Output conforms to `stage-08-result` schema.
- Footer mandate is present in each generated document artifact.
