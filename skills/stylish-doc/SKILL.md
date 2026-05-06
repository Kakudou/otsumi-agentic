---
name: stylish-doc
description: "Create visually striking system design diagrams and documents in Excalidraw with a cyberpunk-terminal aesthetic. Use this skill whenever the user asks for a 'stylish doc', 'visual doc', 'diagram', 'excalidraw', 'architecture diagram', 'flowchart', 'C2', 'C4', or wants to convert content (markdown, mermaid, notes, plans, architecture docs, runbooks, guides) into polished Excalidraw output. Also trigger on phrases like 'make it look good', 'visualize this', 'make this presentable', 'draw this', 'diagram this', 'turn this into something shareable', 'stylize this', or 'give this the treatment'. Output is always an Excalidraw .excalidraw JSON file."
---

# Stylish Doc — Excalidraw Forge

Create production-grade Excalidraw output with a cyberpunk-terminal design system. Diagrams get real wired connections. Documents get clean spatial layouts. Never confuse the two.

## Prerequisites — Read These First (MANDATORY)

1. **FIRST** invoke `/frontend-design` creative rigor and anti-slop principles apply to everything, including Excalidraw output.
2. **THEN** read `references/excalidraw-render.md` — element spec, arrow wiring, geometry computation, validation script. Do not skip this.

## THE MOST IMPORTANT RULE — Content Classification

Before building anything, classify every piece of content into one of two modes:

### Mode 1: DIAGRAM — Wired Nodes + Bound Arrows

Use for: architecture, C2/C4, flowcharts, decision trees, data flows, sequences, pipelines, state machines, process flows, lifecycle diagrams.

Characteristics:
- Nodes connected by **bound arrows** (startBinding + endBinding)
- **Right shape for right concept**: rectangles for services, diamonds for decisions, ellipses for terminals
- Arrows have labels describing what flows
- Dragging a node moves its connections
- Dashed boundaries for system/layer grouping

### Mode 2: DOCUMENT LAYOUT — Styled Boxes, Spatial Organization

Use for: guides, checklists, scoring rubrics, interview cards, reference sheets, cheat sheets, tables, ideal-answer callouts, red-flag lists, evaluation criteria.

Characteristics:
- **NO forced arrows between content items** — text boxes are spatially arranged, not wired
- Boxes contain readable text (titles, bullet lists, scores, questions)
- Visual hierarchy through color, size, borders — not through connections
- Organized in columns/rows with clear spatial grouping
- Dashed boundaries for section grouping (same as diagrams)
- Clean, dense, readable — like a well-designed printed reference card

**The test:** Would drawing arrows between these items add meaning, or just visual noise? If the content is "list of criteria" or "questions to ask" or "things to check" — that's a layout, not a diagram. If the content is "A sends data to B which triggers C" — that's a diagram.

## CRITICAL — Do Not Over-Diagram

**Default to document layout mode.** Most content is reference material — specs, rules, lists, tables, guides. Only switch to diagram mode when content has genuine causal or flow relationships that arrows make clearer than text alone.

**Decision rule:** Can you convey this clearly as a numbered list or table? → Layout card. Does it have branching logic, parallel paths, or non-obvious routing? → Diagram.

Common misclassifications to avoid:
- A numbered list of steps → layout card, not a pipeline diagram
- A prerequisite chain → a sentence, not a flow
- A spec with fields and values → a table, not a data flow
- A file tree → indented text, not a node graph

**Use diagram mode ONLY when:**
- Content has branching decisions (if/else) → diamonds + arrows
- Content has parallel paths or fan-out that spatial layout cannot convey
- Data or requests flow between distinct systems → C2 with bound arrows
- Content has feedback loops, cycles, or non-obvious routing
- The user explicitly asks for a diagram, flowchart, or architecture drawing

**Expected ratio:** 70–90% document layout, 10–30% diagrams. If your board is >50% diagrams and the source is not an architecture doc or flowchart spec, you're over-diagramming.

## CRITICAL — No Mermaid Rendering in Excalidraw

When content contains Mermaid, C4, or structured diagram formats, redraw as native Excalidraw elements. Never embed mermaid syntax. See `references/excalidraw-render.md` for the full wiring spec.

## Workflow

1. **Read prerequisites** — frontend-design SKILL.md + excalidraw-render.md (MUST complete before generating)
2. **Analyze content** — identify subject matter and intended audience
3. **Classify each piece** — DIAGRAM or DOCUMENT LAYOUT (apply the decision rule above; wrong classification = wrong output)
4. **Design canvas zones** — spatial regions separated by divider lines, largest content top-left
5. **Build diagrams** — wired nodes + bound arrows per excalidraw-render spec
6. **Build document layouts** — styled boxes, clean text, spatial grouping, NO forced arrows
7. **Run validation pass** — execute `validate_and_autosize()` from excalidraw-render.md; fix all errors before output
8. **Output** — deliver a single `.excalidraw` file named after the content (e.g., `pipeline-architecture.excalidraw`)

## Canvas Layout

### Multi-Zone Board
- Divide canvas into zones using thin divider lines (opacity 40, C.border)
- Position zones in a 2×N grid: largest content top-left
- 100–150px gaps between zones
- Section title above each zone

### Document Layout Patterns

| Content Type | Layout Pattern |
|-------------|---------------|
| Scoring rubric / criteria table | Column of styled boxes with consistent widths, alternating fills |
| Interview questions + probes | Question box (accent border) with indented follow-up boxes below |
| Ideal answers / callouts | Colored box with accent top bar and bullet text |
| Red flags / warnings | Red-bordered boxes with ✗ prefixed items |
| Phase/section headers | Small label text above a dashed boundary |
| Checklists | Column of checkbox-prefixed text boxes |
| Decision thresholds | Stacked colored boxes (green→cyan→amber→red gradient) |

### Document Layout Box Styling

Document boxes use lighter styling than diagram nodes:
- `strokeWidth: 1` (thinner than diagram nodes which use 2)
- `opacity: 90` for secondary boxes
- Contained text with `fontSize: 12` for body, `14` for titles
- Accent color on the stroke only (not fill), with `C.panel` fill
- Group related boxes inside dashed boundaries with section labels

### Diagram Patterns

See `references/excalidraw-render.md` for full spec. Key reminders:
- **Rectangles** for services/containers, **diamonds** for decisions, **ellipses** for terminals
- Every arrow bound bidirectionally
- Arrow labels describe what flows
- Dashed boundaries for layers/systems

## Color Palette

| Color | Hex | Meaning |
|-------|-----|---------|
| Cyan | `#00e5ff` | Primary, identity, system |
| Magenta | `#ff2d78` | Critical, enforcement |
| Amber | `#ffb300` | Data, storage, scoring |
| Green | `#00e676` | Success, ideal answers |
| Purple | `#b388ff` | Compute, evaluation |
| Red | `#ff5252` | Danger, red flags |

Background: `#060a10` · Panel fill: `#0c1219` · Border: `#1a2535`
Text: `#e0e6ed` · TextDim: `#7a8a9e` · TextMuted: `#3a4a5e`

All nodes: `roughness: 0`, `fontFamily: 3` (monospace), `roundness: {"type": 3}`.

## Validation and Auto-Sizing — MANDATORY

Run `validate_and_autosize()` from `references/excalidraw-render.md` after generating ALL elements, before producing output. The 4-pass validation:

1. **Check types and IDs** — catch missing fields and duplicates
2. **Auto-size nodes** — measure text content, enlarge any node that's too small for its text (never shrinks, logs every resize). Diamonds get 1.4x multiplier, ellipses 1.2x.
3. **Validate arrow bindings** — check bidirectional wiring
4. **Check overlaps** — using post-resize dimensions

Document layouts with no arrows pass passes 3–4 trivially. The auto-sizing pass applies to both modes.

## Examples

**Input:** "Architecture diagram" → **DIAGRAM** mode. Wired C2 with bound arrows.

**Input:** "Interview cheat sheet" → **DOCUMENT LAYOUT** mode. Styled boxes, spatial columns, scoring tables, callout cards. NO wired arrows between text items.

**Input:** "Architecture + interview guide on one board" → **MIXED**. Architecture zone uses DIAGRAM mode. Guide zone uses DOCUMENT LAYOUT mode. Each zone follows its own rules.

**Input:** "Convert mermaid flowchart" → **DIAGRAM** mode. Redraw as native Excalidraw with diamonds, arrows, ellipses.

**Input:** "Visualize this scoring rubric" → **DOCUMENT LAYOUT** mode. Styled boxes in a column. Not a flowchart.
