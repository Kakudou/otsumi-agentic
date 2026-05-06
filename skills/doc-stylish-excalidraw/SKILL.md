---
name: doc-stylish-excalidraw
description: "Create visually striking Excalidraw diagrams or document-layout boards with cyberpunk-terminal styling and real bound-arrow semantics."
---

# Doc Stylish Excalidraw

Create production-grade Excalidraw output with a cyberpunk-terminal design system.

## Usage

- `/doc-stylish-excalidraw <content>`
- `/doc-stylish-excalidraw <file-or-notes> --output <name>.excalidraw`

## Mandatory Prerequisites

1. Invoke `/design-frontend` principles first for creative rigor and anti-slop design thinking.
2. Read `references/excalidraw-render.md` before generating JSON.

## Purpose

Convert content into either:

1. **DIAGRAM mode**: wired nodes with bound arrows.
2. **DOCUMENT LAYOUT mode**: styled spatial cards without forced arrows.

## Hard Rules

- DEFAULT to document layout mode unless arrows genuinely add meaning.
- NEVER over-diagram lists, rubrics, notes, guides, or reference sheets.
- NEVER embed Mermaid syntax in Excalidraw.
- EVERY diagram arrow must be bound bidirectionally.
- ALWAYS run validation/autosizing before output.
- Output one `.excalidraw` JSON file.

## Classification

Use **DIAGRAM** mode for architecture, C2/C4, flowcharts, decision trees, data flows, state machines, process flows, pipelines, and lifecycle diagrams.

Use **DOCUMENT LAYOUT** mode for guides, checklists, scoring rubrics, interview cards, reference sheets, cheat sheets, red flags, evaluation criteria, and tables.

Test: if arrows add causal/flow meaning, diagram. If arrows add noise, layout.

## Workflow

1. Read prerequisites.
2. Analyze content and audience.
3. Classify each piece as diagram or document layout.
4. Design canvas zones.
5. Build diagrams with semantic shapes and bound arrows.
6. Build document layouts with styled boxes and spatial grouping.
7. Run validation and autosize.
8. Write a single `.excalidraw` file.

## Design System

| Color | Hex | Meaning |
|---|---|---|
| cyan | `#00e5ff` | primary/system |
| magenta | `#ff2d78` | critical/enforcement |
| amber | `#ffb300` | data/storage/scoring |
| green | `#00e676` | success |
| purple | `#b388ff` | compute/evaluation |
| red | `#ff5252` | danger |
| background | `#060a10` | canvas |
| panel | `#0c1219` | box fill |
| border | `#1a2535` | dividers |
| text | `#e0e6ed` | primary text |

Use monospace, roughness `0`, rounded corners, clean dense layout.
