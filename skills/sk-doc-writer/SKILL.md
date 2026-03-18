---
name: sk-doc-writer
description: Writes human-facing documentation for a feature. Standalone, no pipeline state required. Use with a feature name, the feature file, and optionally src/ files and decision records. Produces README sections, blog posts, API reference, changelogs, and Obsidian notes.
---

# doc-writer

One job: make the feature legible to humans who were not in the room.

Not specs. Not decisions. Not tests.
The narrative layer: what was built, why it matters, how to use it.

---

## Inputs

Provided by whoever calls this skill (agent or direct invocation):

| Input | Required | Description |
|-------|----------|-------------|
| `feature_name` | yes | kebab-case name of the feature |
| `feature_file` | no | path to the `.feature` file when the feature has one; omit for workflows without a spec stage |
| `language_id` | no | language family context (e.g. `python`), passed by the agent when in pipeline context, omit for standalone use |
| `src_files` | no | list of `src/` files built for this feature |
| `decisions_dir` | no | path to `docs/decisions/` (defaults to `docs/decisions/`) |
| `doc_types` | no | desired output types: `readme`, `blog`, `api-reference`, `changelog`, `obsidian`; if omitted, default to `readme` + `changelog` |
| `stage_results` | no | map of available stage result summaries for additional context |

---

## Outputs

| Output | Description |
|--------|-------------|
| one or more documentation files | written to agreed paths based on `doc_types` |
| `stage-08-result` | list of files written, returned to caller, not written to disk by this skill |

The caller is responsible for writing stage output JSONs and logging.

---

## Activation Steps

1. Confirm `feature_name` is provided by the caller.
2. Read `<feature_file>` when provided: the authoritative source of feature behavior.
3. Read `<src_files>` if provided: understand what was actually built.
4. Read `<decisions_dir>` (default: `docs/decisions/`): any ADR/TDR written for this feature.
5. Use `stage_results` for additional context on what was decided and why.
6. Determine which documentation types to produce:
   - use `doc_types` if provided
   - otherwise default to `[readme, changelog]`
7. Write the documentation files.
8. Return `stage-08-result` to the caller.

---

## Documentation Types

Choose based on the nature of the feature. Multiple types can be produced in one pass.

### README section

A self-contained block suitable for embedding in a project README.

### Blog post

A narrative explanation of the feature for a technical audience who is not familiar with the codebase.

### API reference

For features that expose a public interface such as a function, command, endpoint, or public contract.

### Changelog entry

A concise entry suitable for a `CHANGELOG.md` or release notes.

### ObsidianMD feature note

We are using ObsidianMD to document about everything occuring during the project.
We follow a strict Zettelkasten organization, so each note is a self-contained, atomic, Zettel with a unique title and wikilinks to related notes.

---

## Writing Rules

- Use **domain language first** in README/blog/changelog/Obsidian outputs.
- API reference output may use interface terms such as command, parameter, return value, or endpoint when that is the public surface being documented.
- **Examples first**: show what it does before explaining how.
- **Never invent behavior**: every claim must be traceable to the `.feature` file or `src/`.
- **No placeholder sections**: every section written must have real content or be omitted.
- **Wikilinks** for Obsidian notes: `[[ADR-XXXX-title]]` not bare paths.
- **Traps are features**: the constraint and edge case handling from Stage-2 is worth documenting when available.
- **Always** add a footer: `**Written by the Hand of <persona name fallback to model name> and the Eyes of <user name fallback to 'the Dev'>.**`

---

## Result Schema

Return `stage-08-result` to the caller:

```text
docs_written:
  - type: readme | blog | api-reference | changelog | obsidian
    path: <output file path>
```

The caller writes the stage output JSON and runs `/atomic-log` if in pipeline context.

---

## Handoff

Return `stage-08-result` to the caller.
