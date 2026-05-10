---
name: kb-obsidian-write
description: "Generate polished prose from vault knowledge (zettels and/or assembled resource notes), with mandatory citation of the source zettel for every factual claim. Writes original knowledge-driven prose — NOT a preprocessor for doc-writer, NOT a wrapper around assemble."
interaction_model: multi-turn
---

# KB Obsidian Write

Build original, citation-backed prose from a vault knowledge graph.

This skill:
- reads zettels and/or an assembled resource note,
- synthesizes coherent long-form prose (article, essay, explanation, report, etc.),
- enforces that **every factual claim has an inline source zettel citation**.

Pipeline role:
- downstream of `kb-obsidian-assemble`,
- upstream of optional style refinement (`doc-writer`),
- can also run standalone from raw zettels or a topic query.

---

> **Disambiguation — kb-obsidian-write vs doc-writer**
>
> These two skills operate on fundamentally different inputs and produce fundamentally different outputs. They are **not variants of the same operation**.
>
> | Aspect | kb-obsidian-write | doc-writer |
> |---|---|---|
> | **Input** | Vault knowledge — zettels and/or resource notes | Project artifacts — code, specs, decisions, tests |
> | **Output** | Original prose built from personal knowledge | Human-facing documentation derived from verified artifacts |
> | **Citation** | MUST cite source zettel (`[[zettel-title]]`) for every factual claim | Cites source artifacts internally during the work process |
> | **Role** | Builds NEW prose from knowledge | Refactors / structures EXISTING artifact behavior into docs |
> | **Voice** | Neutral, knowledge-driven, author's own synthesis | Preserves existing project tone |
> | **Footer** | No footer mandate | Mandatory "Written by the Hand of…" footer |
>
> Use `kb-obsidian-write` when the source of truth is the vault. Use `doc-writer` when the source of truth is the codebase or delivery artifacts.

---

> **Pipeline position — assemble → write → doc-writer**
>
> This skill occupies the **middle** stage of the vault prose pipeline:
>
> ```
> kb-obsidian-assemble  →  kb-obsidian-write  →  doc-writer
>    (extract/curate)         (build prose)       (refine style/voice)
> ```
>
> - **assemble is OPTIONAL**: write accepts zettels directly and bypasses assembly entirely. Skipping assemble means the LLM synthesizes from raw vault knowledge without a pre-curated structure — this is intentional and valid.
> - **doc-writer is OPTIONAL**: write's output is complete and usable as-is. Invoke doc-writer only when additional style harmonisation with a project is needed.

---

## Invocation

- `/kb-obsidian-write "{topic}"` — find relevant zettels by topic query, generate prose.
- `/kb-obsidian-write --from {zettel-path}` — write from a specific zettel. Repeat `--from` for multiple zettels.
- `/kb-obsidian-write --from-resource {resource-path}` — write from an assembled resource note (output of `kb-obsidian-assemble`).
- `/kb-obsidian-write --template {template-path-or-description}` — follow a template's structure and format. See template behavior below.
- `/kb-obsidian-write --output {path}` — override output location. Defaults to `resources_root`.
- `/kb-obsidian-write --lang {EN|FR|...}` — set output language.
- `/kb-obsidian-write --dry-run` — produce the full prose proposal without writing to disk.
- `/kb-obsidian-write {vault-id} "{topic}"` — scope to a named vault.

Flags may be combined freely:

```
/kb-obsidian-write "Indian Attack chess opening"
/kb-obsidian-write --from zettels/indian-attack-principles.md --from zettels/king-side-fianchetto.md --lang EN
/kb-obsidian-write --from-resource resources/Indian Attack.md --template templates/blog-post.md --output ~/blog/indian-attack.md
/kb-obsidian-write chess "Indian Attack" --dry-run
```

### `--template` behavior

- **With `--template {path-or-description}`**: The generated prose MUST follow that template's section structure, heading hierarchy, and field layout exactly. If the value resolves to a file path (vault-relative or absolute), read the file and derive structure from it. If the value is a format description string (e.g. `"blog post with intro, three sections, conclusion"`), treat it as a structural specification.
- **Without `--template`**: The skill produces freestyle prose in whatever form best serves the content — length, heading depth, and structure are chosen to fit the synthesised material, not a fixed mold.

## Hard Rules

- MUST resolve `zettel_root`, `resources_root`, `zettel_template`, and any safety caveats from `system.md` → `## Knowledge Bases` → `{vault-id}`. NEVER hardcode any of those paths.
- MUST read `zettel_template` before generating any output that includes frontmatter. Frontmatter key names, casing (`tags:` lowercase, `Lang:` capitalised, etc.) and date format (`YYYY/MM/DD HH:mm:ss`) MUST match the template exactly, NEVER guessed.
- MUST cite the source zettel for **every factual claim** in generated prose. A factual claim is any statement that conveys information originating from a zettel rather than structural or connective prose. Citations use `[[zettel-title]]` wikilink notation inline. **Uncited factual claims are prohibited.**
- MUST NEVER fabricate information not present in the source zettels. If the zettels do not support a claim, the claim does not appear in the prose. If knowledge gaps prevent a complete treatment of the topic, surface them explicitly rather than filling them with invention.
- MUST present the full proposed prose for `approve` / `edit` / `abort` BEFORE writing anything to disk.
- MUST increment `total_access` on the zettel's usage score when reading a zettel during source resolution or citation scan. MUST increment `use_count` when that zettel is actually cited as a source in the produced prose.
- MUST write atomically: write to a temp file in the output directory, then rename into the final path. NEVER write directly to the final path.
- MUST honor every safety caveat declared on the vault entry. If the output path resolves under a directory listed as a submodule in the vault's safety caveats, emit a **submodule write disclosure** unconditionally, before writing.
- MUST NEVER `git add`, `git commit`, or `git push`. No git operation is performed at any stage.
- Zettel content and structural metadata are immutable per the Zettel Mutability Policy (S3). Counter fields (total_access, use_count) are the sole permitted writes to existing zettels and are mandatory on access.
- NEVER produce output with frontmatter unless `zettel_template` was read first and field casing was derived from it.
- NEVER overwrite an existing output file without showing a diff and receiving explicit approval.

## Steps

### Step 1 — Resolve target vault

1. Read `system.md` → `## Knowledge Bases` → vault entry (by id or default).
2. Extract `zettel_root`, `resources_root`, `zettel_template`, and any safety caveats.
3. Read `zettel_template`. Capture the canonical frontmatter key names, casing, `Author` default, date format, and `Lang` default.
4. If `--template` references a file path, read that file now. Capture its section headings, field order, and structural constraints.

### Step 2 — Resolve source zettels

Depending on the invocation mode:

**Topic query** (`/kb-obsidian-write "{topic}"`):
1. List `zettel_root`.
2. MUST invoke `kb-obsidian-search` for initial candidate discovery before applying local scoring. Pass the topic, relevant tags, and any temporal constraints. Use its results as the candidate set for subsequent scoring.
3. Score zettels against the topic by title/alias match, tag intersection, and body content overlap.
4. Build a ranked `candidate_set` with per-zettel relevance reasoning. Surface the set to the user with the intended coverage before proceeding.

**Direct zettel paths** (`--from {path} [--from {path}...]`):
1. Verify each path exists in or relative to `zettel_root`.
2. Confirm the set covers the intended topic; surface any obvious thematic gaps.

**Assembled resource note** (`--from-resource {resource-path}`):
1. Read the resource note.
2. Extract all wikilinks from `Links:` frontmatter and the `## Sources` footer as the working zettel set.
3. Read each linked zettel.

**Mixed** (`--from-resource` + additional `--from` paths):
1. Execute both flows above and merge into a unified working set, deduplicating by path.

After source resolution:
- Increment `total_access` on each zettel's frontmatter for each zettel read.
- Confirm the working set with the user before proceeding to prose generation. Surface any detected coverage gaps.

### Step 3 — Plan prose structure

1. Define the output structure:
   - **With `--template`**: map sections from the template to zettel coverage. Flag uncovered mandatory sections as gaps.
   - **Without `--template`**: derive a section outline that best fits the working zettel set and the requested topic.
2. Map each working zettel to one or more sections where its content contributes.
3. Present the outline and zettel-to-section mapping to the user. Allow adjustment before writing.

### Step 4 — Generate prose

1. Write prose section by section, drawing only from the working zettel set.
2. For every factual claim, embed an inline citation in `[[zettel-title]]` wikilink format immediately at the point of the claim (e.g. `The Indian Attack favours a King-side fianchetto [[King-side fianchetto motif]].`).
3. Structural prose — headings, transition sentences that name without claiming, introductory framing, connective glue — does not require citation.
4. If a section's required content is not supported by any working zettel:
   - Do NOT invent the content.
   - Insert a `> [!GAP] {section title}: no zettel covers this claim.` callout and continue.
5. After all sections are drafted, scan the full text for any factual statement lacking a citation. Either add the citation or remove the claim. There must be zero uncited factual claims at the end of this pass.
6. Increment `use_count` on each zettel that appears at least once as a citation in the final prose.

### Step 5 — Compose final output

If the output will be written as a vault note (default to `resources_root`):

1. Build frontmatter from `zettel_template`:
   - `Title`: derived from the topic or user-supplied title.
   - `Aliases`: optional aliases the user supplied.
   - `Author`: vault default.
   - `tags`: union of distinguishing tags from cited zettels (deduplicated, lowercase, matching template casing).
   - `Links`: YAML list of wikilinks to every zettel cited at least once.
   - `Lang`: `--lang` value or vault default.
   - `Template`: `Resource` (or appropriate value from template).
   - `Creation Date` / `Modification Date`: current time in `YYYY/MM/DD HH:mm:ss`.
   - `total_access`: `0`
   - `use_count`: `0`

If the output target is outside the vault (`--output` to an external path): omit frontmatter unless the user explicitly requests it.

Filename (when writing to vault): `{Title}.md` in `resources_root`. Replace Obsidian-prohibited filename characters (`/`, `\`, `:`, `|`, `?`, `*`, `<`, `>`, `"`) with safe equivalents and disclose any substitution. If an existing file with the same name is found, switch to update flow (Step 6).

Append a `## Sources` footer listing every cited zettel as wikilinks, matching the `Links:` frontmatter (same set, no extras in either direction).

### Step 6 — Update flow (existing output file)

If the output file already exists:

1. Read the existing file.
2. Compute the minimal diff:
   - new cited zettels → append to `Links:` and `## Sources`.
   - new tags → append.
   - updated body sections → show diff per section.
   - update `Modification Date`.
   - NEVER touch `Creation Date`, `Title`, `Author`, `Template`, or `Lang` unless the user explicitly approves.
3. Show the full diff. Get explicit approval. Write atomically.

### Step 7 — Approval gate and write

1. Present the full proposed output (prose + frontmatter if applicable) for `approve` / `edit` / `abort`.
2. `--dry-run`: stop here; emit the proposal as the skill's return value. Write nothing.
3. On `approve` or accepted `edit`:
   - If the output path resolves under a submodule directory, emit the **submodule write disclosure** now (unconditionally, before the write).
   - Write to a temp file in the output directory.
   - Rename temp file to the final path.
   - Verify the written file is readable and the frontmatter (if present) is valid YAML.

### Step 8 — Report

Return:

- output path written (or proposal if `--dry-run`)
- list of cited zettels with citation count per zettel
- list of zettels read but not cited (sourced but not quoted)
- any gaps surfaced as `[!GAP]` callouts, with suggested `kb-obsidian-zettelize` invocations to fill them
- submodule write disclosure if applicable
- reminder that no git operation was performed
- reminder that no zettel content or metadata was modified (counter increments are permitted per S3)

## Validation

Before claiming success, all checks below MUST be true:

- [ ] `zettel_root`, `resources_root`, and `zettel_template` all came from `system.md`. No paths were hardcoded.
- [ ] `zettel_template` was read before any frontmatter was generated. Key casing matches the template exactly.
- [ ] Every factual claim in the produced prose has an inline `[[zettel-title]]` citation. No uncited factual claims remain.
- [ ] No information was invented that was not present in the working zettel set.
- [ ] `total_access` was incremented for each zettel read during source resolution.
- [ ] `use_count` was incremented for each zettel cited at least once in the final prose.
- [ ] `Links:` frontmatter and `## Sources` footer reference the same cited zettel set — no extras in either direction.
- [ ] Full prose proposal was presented for user approval before any write occurred.
- [ ] Output was written atomically (temp file + rename). No direct write to final path.
- [ ] No zettel content or metadata was modified (counter increments are permitted per S3).
- [ ] No git operation was performed.
- [ ] Submodule writes were disclosed before the write, where applicable.
- [ ] If `--dry-run`: no file was written; proposal returned as output only.
- [ ] If `--template`: the output structure conforms to the template's section layout and field order.
