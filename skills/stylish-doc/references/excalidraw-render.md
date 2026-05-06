# Excalidraw Render Reference — stylish-doc

How to generate Excalidraw-compatible JSON with **real diagram semantics** — bound arrows, connected nodes, proper shapes, and auto-sized containers.

---

## CRITICAL RULE: No Mermaid Rendering

When the user's content contains diagrams described as Mermaid, C4, flowcharts, sequences, or any structured diagram format:

**NEVER embed mermaid syntax in Excalidraw.** Instead, redraw every diagram as native Excalidraw elements — rectangles, diamonds, ellipses, arrows — with proper bindings. The output must be a real interactive diagram where dragging a node moves its arrows.

---

## File Format

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "stylish-doc",
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

---

## Color Mapping

| Design System | Excalidraw Usage | Hex |
|--------------|-----------------|-----|
| `C.bg` | `viewBackgroundColor` | `#060a10` |
| `C.panel` | Rectangle `backgroundColor` | `#0c1219` |
| `C.border` | Divider strokes | `#1a2535` |
| `C.cyan` | Primary / identity | `#00e5ff` |
| `C.magenta` | Critical / enforcement | `#ff2d78` |
| `C.amber` | Data / storage | `#ffb300` |
| `C.green` | Healthy / success | `#00e676` |
| `C.purple` | Compute / processing | `#b388ff` |
| `C.red` | Danger / failure | `#ff5252` |
| `C.text` | Primary text | `#e0e6ed` |
| `C.textDim` | Secondary text | `#7a8a9e` |
| `C.textMuted` | Tertiary text | `#3a4a5e` |

---

## Shape Selection Guide

| Concept | Shape | Excalidraw `type` | When to Use |
|---------|-------|-------------------|-------------|
| Service, container, process | Rectangle | `rectangle` | Any named component, service, or container |
| Decision, condition, branch | Diamond | `diamond` | Yes/No questions, if/else, routing decisions |
| Terminal, start, end, state | Ellipse | `ellipse` | Process start/end, final states, status indicators |
| System boundary, layer | Dashed rectangle | `rectangle` (strokeStyle: dashed) | C4 system boundaries, layer groupings |
| Data flow, relationship | Arrow | `arrow` | Every connection between elements |

**Rules:**
- NEVER use a rectangle where a diamond is semantically correct (decisions)
- NEVER use a rectangle where an ellipse is semantically correct (terminals)
- EVERY connection must be an arrow with bindings, not a line or a visual suggestion
- Dashed boundaries contain other elements — they are NOT connected by arrows

---

## Arrow Wiring — THE MOST IMPORTANT SECTION

Arrows are what make a diagram a diagram. Every arrow must be properly wired to its source and target nodes.

### Arrow Geometry

The arrow's `x` and `y` are the **absolute position of the start anchor point** on the canvas. The `points` array contains **relative offsets** from that anchor.

```python
# Computing arrow geometry from node positions
if from_side == "bottom":
    sx = src_x + src_w / 2    # center of bottom edge
    sy = src_y + src_h         # bottom edge
elif from_side == "right":
    sx = src_x + src_w
    sy = src_y + src_h / 2
# ... same for "top" and "left"

if to_side == "top":
    tx = tgt_x + tgt_w / 2
    ty = tgt_y
# ... same pattern

arrow_x = sx              # absolute position
arrow_y = sy
dx = tx - sx
dy = ty - sy
points = [[0, 0], [dx, dy]]   # relative to arrow origin
```

### Arrow Binding (bidirectional wiring)

**Both sides must be wired.** The arrow references the nodes, AND the nodes reference the arrow.

```json
// ON THE ARROW:
{
  "startBinding": {"elementId": "source-node-id", "focus": 0, "gap": 4},
  "endBinding": {"elementId": "target-node-id", "focus": 0, "gap": 4}
}

// ON BOTH SOURCE AND TARGET NODES:
{
  "boundElements": [
    {"id": "label-text-id", "type": "text"},
    {"id": "arrow-id", "type": "arrow"}
  ]
}
```

### Arrow Side Selection

| Relationship | from_side | to_side |
|-------------|-----------|---------|
| Target below source | bottom | top |
| Target right of source | right | left |
| Target above source | top | bottom |
| Target left of source | left | right |
| Loop back | right | right |

### Arrow Labels

Position at midpoint of arrow, offset slightly:
```python
label_x = arrow_x + dx/2 + 6
label_y = arrow_y + dy/2 - 8
```

---

## Node with Embedded Label

Every node should have its label text as a **contained text element**:
- Text's `containerId` = node's `id`
- Node's `boundElements` includes `{"id": text_id, "type": "text"}`
- Text uses `textAlign: "center"`, `verticalAlign: "middle"`

Optional subtitle: separate text element (not contained), positioned at node bottom, smaller font, dimmed color.

---

## Dashed Boundary

Visual groupings, NOT connected by arrows. Label at top-left inside.

```json
{"type": "rectangle", "strokeStyle": "dashed", "strokeWidth": 1, "opacity": 35, "backgroundColor": "transparent"}
```

Every element must have `"type"` explicitly set.

---

## Text Measurement — CRITICAL FOR SIZING

Excalidraw uses monospace font (fontFamily: 3 = Cascadia). Text dimensions can be computed precisely:

```python
# Character width depends on fontSize
# Cascadia/monospace: char_width ≈ fontSize * 0.6
# Line height ≈ fontSize * 1.25 (matching Excalidraw's lineHeight: 1.25)

def measure_text(content, font_size):
    """Compute pixel dimensions of a text block."""
    lines = content.split("\n")
    char_width = font_size * 0.6
    line_height = font_size * 1.25
    
    text_width = max(len(line) for line in lines) * char_width
    text_height = len(lines) * line_height
    
    return text_width, text_height

def compute_node_size(label, font_size=14, subtitle=None, sub_font_size=10,
                      pad_x=24, pad_y=20, min_w=80, min_h=36):
    """Compute minimum node dimensions to fit label + optional subtitle."""
    label_w, label_h = measure_text(label, font_size)
    
    total_w = label_w + pad_x * 2
    total_h = label_h + pad_y * 2
    
    if subtitle:
        sub_w, sub_h = measure_text(subtitle, sub_font_size)
        total_w = max(total_w, sub_w + pad_x * 2)
        total_h += sub_h + 4  # 4px gap between label and subtitle
    
    return max(total_w, min_w), max(total_h, min_h)
```

### Sizing Rules

- **Always compute node dimensions from text content** using `compute_node_size()` or equivalent
- **Never hardcode node width/height** without checking whether the text fits
- **Padding**: 24px horizontal, 20px vertical minimum inside nodes
- **Minimum dimensions**: 80px wide, 36px tall (even for empty nodes)
- **Diamonds need ~40% more space** than rectangles because content sits inside the diamond's inner rectangle (inscribed), not the full bounding box. Multiply computed size by 1.4 for diamonds.
- **Ellipses need ~20% more space** — multiply by 1.2

### Document Layout Cards

For document-layout mode (styled boxes with text, no arrows), card sizing follows:

```python
def compute_card_size(title, body, title_size=13, body_size=11,
                      pad_x=24, pad_y=16, min_w=200):
    """Compute card dimensions to fit title + body text."""
    title_w, title_h = measure_text(title, title_size)
    
    total_w = title_w + pad_x * 2
    total_h = title_h + pad_y  # top padding + title
    
    if body:
        body_w, body_h = measure_text(body, body_size)
        total_w = max(total_w, body_w + pad_x * 2)
        total_h += body_h + 8  # gap + body
    
    total_h += pad_y  # bottom padding
    
    return max(total_w, min_w), total_h
```

---

## Layout Rules

### Grid System

- Base grid: 20px
- Node width: **computed from text** (see above), typical range 180–300px
- Node height: **computed from text** (see above), typical range 40–70px
- Horizontal gap between siblings: 40–60px
- Vertical gap between levels: 50–80px
- Layer gap (between major sections): 80–100px
- Boundary padding: 20–30px around contained elements

### Flow Direction

| Diagram Type | Primary Flow | Secondary Flow |
|-------------|-------------|----------------|
| C2/C4 Container | Top → Bottom (layers) | Left → Right (siblings) |
| Flowchart | Top → Bottom (decisions) | Left/Right (Yes/No branches) |
| Sequence/Pipeline | Left → Right (steps) | Top → Bottom (fan-out) |
| Process/Lifecycle | Top → Bottom (steps) | Right → Right (loops) |

### Multi-Diagram Board Layout

- Divider lines (opacity 40, C.border) between zones
- 2×N grid layout, largest content top-left
- 100–150px gaps between zones
- Section title above each zone

---

## Validation and Auto-Sizing Pass — MANDATORY

After generating all elements, run the validation pass before outputting. The pass has 4 stages:

1. **Collect IDs and check types** — catch missing IDs and types
2. **Auto-size nodes** — measure contained text, resize nodes that are too small
3. **Validate arrow bindings** — check start/end bindings exist and are bidirectional
4. **Check overlaps** — catch same-level nodes stacked on each other

### Validation Script

```python
def measure_text(content, font_size):
    """Compute pixel dimensions of a text block in monospace."""
    lines = content.split("\n")
    char_width = font_size * 0.6
    line_height = font_size * 1.25
    text_width = max(len(line) for line in lines) * char_width
    text_height = len(lines) * line_height
    return text_width, text_height


def validate_and_autosize(elements):
    """Validate + auto-resize nodes to fit their text content."""
    errors = []
    warnings = []
    id_set = set()
    id_map = {}     # id → element
    nodes = {}      # id → {x, y, w, h}

    # ── Pass 1: Collect IDs, check types ──
    for el in elements:
        eid = el.get("id")
        if not eid:
            errors.append(f"Element missing id: {el.get('type', '?')}")
            continue
        if eid in id_set:
            errors.append(f"Duplicate id: {eid}")
        id_set.add(eid)
        id_map[eid] = el

        if not el.get("type"):
            errors.append(f"Element missing type: id={eid}")

        if el.get("type") in ("rectangle", "diamond", "ellipse"):
            nodes[eid] = {
                "x": el["x"], "y": el["y"],
                "w": el["width"], "h": el["height"],
            }

    # ── Pass 2: Auto-size nodes to fit contained text ──
    resized = 0
    for el in elements:
        if el.get("type") not in ("rectangle", "diamond", "ellipse"):
            continue
        if el.get("strokeStyle") == "dashed":
            continue  # skip boundaries — they contain other nodes, not just text

        node_id = el["id"]
        pad_x = 24
        pad_y = 20

        # Find all text elements contained in this node
        contained_texts = [
            t for t in elements
            if t.get("type") == "text" and t.get("containerId") == node_id
        ]

        # Find non-contained text positioned inside this node (subtitles, body)
        nx, ny, nw, nh = el["x"], el["y"], el["width"], el["height"]
        child_texts = [
            t for t in elements
            if t.get("type") == "text"
            and t.get("containerId") is None
            and t.get("boundElements") is None
            and nx <= t["x"] <= nx + nw
            and ny <= t["y"] <= ny + nh
        ]

        if not contained_texts and not child_texts:
            continue

        # Compute required width from widest text
        required_w = 0
        required_h = pad_y  # top padding

        for t in contained_texts:
            tw, th = measure_text(t["text"], t.get("fontSize", 14))
            required_w = max(required_w, tw + pad_x * 2)
            required_h = max(required_h, th + pad_y * 2)

        # For child (non-contained) texts, compute bounding box
        if child_texts:
            min_tx = min(t["x"] for t in child_texts)
            max_tx_w = max(t["x"] + measure_text(t["text"], t.get("fontSize", 14))[0]
                          for t in child_texts)
            max_ty_h = max(t["y"] + measure_text(t["text"], t.get("fontSize", 14))[1]
                          for t in child_texts)

            content_w = (max_tx_w - nx) + pad_x
            content_h = (max_ty_h - ny) + pad_y

            required_w = max(required_w, content_w)
            required_h = max(required_h, content_h)

        # Diamonds need 40% more space (text inscribed in inner rect)
        if el["type"] == "diamond":
            required_w *= 1.4
            required_h *= 1.4

        # Ellipses need 20% more space
        if el["type"] == "ellipse":
            required_w *= 1.2
            required_h *= 1.2

        # Only resize if node is too small
        if el["width"] < required_w - 2 or el["height"] < required_h - 2:
            old_w, old_h = el["width"], el["height"]
            el["width"] = max(el["width"], required_w)
            el["height"] = max(el["height"], required_h)
            resized += 1
            warnings.append(
                f"Auto-resized {node_id}: {old_w:.0f}x{old_h:.0f} → "
                f"{el['width']:.0f}x{el['height']:.0f}"
            )

            # Update the nodes dict for overlap check
            if node_id in nodes:
                nodes[node_id]["w"] = el["width"]
                nodes[node_id]["h"] = el["height"]

    # ── Pass 3: Validate arrow bindings ──
    for el in elements:
        if el.get("type") != "arrow":
            continue
        aid = el["id"]

        for side, key in [("start", "startBinding"), ("end", "endBinding")]:
            b = el.get(key)
            if not b or not b.get("elementId"):
                errors.append(f"Arrow {aid}: missing {key}")
            elif b["elementId"] not in id_set:
                errors.append(f"Arrow {aid}: {key} → non-existent {b['elementId']}")
            else:
                tgt = id_map.get(b["elementId"])
                if tgt:
                    bound = tgt.get("boundElements") or []
                    if isinstance(bound, list) and not any(
                        x.get("id") == aid for x in bound
                    ):
                        errors.append(
                            f"Arrow {aid}: {b['elementId']} missing arrow in boundElements"
                        )

    # ── Pass 4: Check overlaps ──
    node_list = [
        (nid, n) for nid, n in nodes.items()
        if not nid.startswith("bd") and not nid.startswith("div")
    ]
    for i, (id_a, a) in enumerate(node_list):
        for id_b, b in node_list[i + 1 :]:
            if (
                a["x"] < b["x"] + b["w"]
                and a["x"] + a["w"] > b["x"]
                and a["y"] < b["y"] + b["h"]
                and a["y"] + a["h"] > b["y"]
            ):
                a_area = a["w"] * a["h"]
                b_area = b["w"] * b["h"]
                if a_area > b_area * 3 or b_area > a_area * 3:
                    continue
                errors.append(f"Overlap: {id_a} and {id_b}")

    # ── Report ──
    arrows = sum(1 for e in elements if e.get("type") == "arrow")

    if warnings:
        print(f"  AUTO-SIZED {resized} nodes:")
        for w in warnings:
            print(f"    ↻ {w}")

    if errors:
        print(f"  VALIDATION FAILED — {len(errors)} errors:")
        for e in errors:
            print(f"    ✗ {e}")
        return False
    else:
        print(
            f"  VALIDATION PASSED — {len(nodes)} nodes, {arrows} arrows, "
            f"{resized} auto-resized, all bindings valid, no overlaps"
        )
        return True
```

### What the Validation Catches

| Pass | Check | What It Prevents |
|------|-------|-----------------|
| 1 | Missing `type` on elements | Invisible/broken elements |
| 1 | Duplicate IDs | Binding confusion |
| 2 | **Node too small for contained text** | **Text overflow, manual resizing needed** |
| 2 | **Node too small for child text elements** | **Content clipped or overflowing** |
| 2 | **Diamond/ellipse undersized** | **Text doesn't fit inscribed area** |
| 3 | Arrow missing startBinding | Arrow floats free from source |
| 3 | Arrow missing endBinding | Arrow floats free from target |
| 3 | Binding → non-existent ID | Broken reference |
| 3 | Node missing arrow in boundElements | Arrow won't follow drag |
| 4 | Same-level node overlap | Nodes stacked on each other |

### Auto-Sizing Behavior

- **Only enlarges**, never shrinks — if a node is already big enough, it's left alone
- **Skips dashed boundaries** — boundaries are sized to contain other nodes, not text
- **Accounts for shape geometry** — diamonds get 1.4x multiplier, ellipses get 1.2x
- **Logs every resize** as a warning so you can see what was adjusted
- **Updates the overlap check** with new dimensions after resizing

### When to Skip

- Dashed boundaries (contain nodes, not just text)
- Intentional nesting (area ratio > 3x)

---

## Generation Strategy

1. **Parse content** — identify diagram types (C2, flowchart, sequence, etc.)
2. **Select shapes** — rectangle/diamond/ellipse per the Shape Selection Guide
3. **Compute text dimensions** — use `measure_text()` for every label
4. **Size nodes from text** — use `compute_node_size()`, never hardcode
5. **Calculate layout** — assign x, y using grid rules and flow direction
6. **Create nodes** — with contained label text and optional subtitle
7. **Create boundaries** — dashed rectangles around grouped nodes
8. **Create arrows** — with proper geometry (anchor points, relative offsets)
9. **Wire bindings** — bidirectional: arrow → nodes AND nodes → arrow
10. **Add labels** — arrow labels at midpoints, section titles as free text
11. **Run `validate_and_autosize()`** — mandatory before output, auto-fixes sizing
12. **Output** — write `.excalidraw` JSON file
