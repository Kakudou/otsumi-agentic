# Excalidraw Render Reference

Generate Excalidraw-compatible JSON with real diagram semantics: bound arrows, connected nodes, proper shapes, auto-sized containers, and validation.

## Critical Rule: No Mermaid Rendering

When input contains Mermaid, C4, flowcharts, sequences, or structured diagram formats, NEVER embed that syntax. Redraw as native Excalidraw elements: rectangles, diamonds, ellipses, arrows, labels, and dashed boundaries.

## File Format

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "doc-stylish-excalidraw",
  "elements": [],
  "appState": {
    "viewBackgroundColor": "#060a10",
    "theme": "dark",
    "gridSize": 20,
    "currentItemFontFamily": 3
  },
  "files": {}
}
```

## Shape Rules

| Concept | Shape | Type |
|---|---|---|
| service/container/process | rectangle | `rectangle` |
| decision/condition/branch | diamond | `diamond` |
| terminal/start/end/state | ellipse | `ellipse` |
| system boundary/layer | dashed rectangle | `rectangle` |
| relationship/data flow | arrow | `arrow` |

Every arrow must include `startBinding` and `endBinding`. Source and target nodes must include the arrow in `boundElements`.

## Sizing Rules

- Measure text before setting node size.
- Use monospace approximation: `char_width = font_size * 0.6`, `line_height = font_size * 1.25`.
- Use at least 24px horizontal padding and 20px vertical padding.
- Diamonds need 1.4x computed size.
- Ellipses need 1.2x computed size.
- Only enlarge during validation; never shrink.

## Mandatory Validation Pass

Before output:

1. Check missing IDs, duplicate IDs, missing `type`.
2. Auto-size nodes/cards from text.
3. Validate bidirectional arrow bindings.
4. Check overlaps, ignoring intentional large container nesting.

If validation fails, fix the JSON before returning the file.
