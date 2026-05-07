# Shape combinations

Reusable composite shape patterns for PPTX decks built on the Teradyne template. Each entry documents a multi-shape visual unit that carries a specific meaning, along with the geometry and python-pptx construction notes.

When to use this reference:
- A slide concept needs more than a single shape to land (e.g. a step-flow with annotated break-points)
- The combination should be reusable across future decks without re-designing
- python-pptx does not expose the needed shape types directly and a deep-copy template approach is required

## How to add a new combination

1. **Name it.** Short noun-phrase describing the visual function.
2. **Show the visual result.** An ASCII sketch is enough to scan-and-recognize.
3. **Specify geometry relative to anchors.** Sizes and offsets computed from the shapes the combo attaches to, not absolute coordinates.
4. **Provide the XML template.** Copy-paste ready, with variable placeholders called out.
5. **Provide python-pptx construction.** Prefer `copy.deepcopy` of a reference element when python-pptx has no wrapper.
6. **Note when to prefer it and when to avoid.**

---

## Combo: step-flow transition with break-point callout

### What it shows

A horizontal process flow where each transition between two consecutive steps is annotated with a risk, failure mode, or decision point. The transition is drawn as a horizontal arrow. A bent branch line drops from the transition midpoint down to a callout card that explains what breaks at that transition.

### Visual sketch

```
[ STEP N ]  ──────►  [ STEP N+1 ]
                │
                ▼
           [ break-point callout card ]
```

The horizontal arrow is the primary flow. The elbow branch signals "and here's what you should notice about this transition."

### When to prefer

- Process diagrams with 3–6 steps
- Each transition has a distinct, material failure mode or decision point
- Callouts need to be visually anchored to specific transitions (not generic body text under the flow)
- The reader should be able to read the flow horizontally and read the risks vertically

### When to avoid

- Steps do not have per-transition risks (use a single body paragraph under the flow instead)
- More than 6 steps (the callouts will crowd; split into two flows)
- Transitions all carry the same risk (one shared annotation is cleaner)

### Geometry

Anchor coordinates (all in inches):
- `step_right_x[i]` — right edge of step card `i`
- `step_left_x[i+1]` — left edge of step card `i+1`
- `arrow_y` — vertical position of the horizontal transition arrow. Set to `step_card_top + 0.42` or similar, placing the line below the step label text so it doesn't intersect glyphs.
- `callout_top_y` — top of the callout card row
- `callout_mid_x[i]` — horizontal center of callout card `i` (typically aligned under the transition midpoint)

Derived values per transition `i`:
- **Horizontal straight arrow**
  - `off_x = step_right_x[i]`
  - `off_y = arrow_y`
  - `cx    = step_left_x[i+1] - step_right_x[i]`  (gap width)
  - `cy    = 0`
- **Elbow branch connector** (`bentConnector2`)
  - `off_x = step_right_x[i]`
  - `off_y = arrow_y`
  - `cx    = callout_mid_x[i] - step_right_x[i]`  (horizontal delta to callout center)
  - `cy    = callout_top_y - arrow_y`             (vertical drop to callout top)

### XML templates

Both connectors use `a:ln w="38100"` (3 pt) with scheme color `accent1` and `a:tailEnd type="triangle"` for arrowheads.

**Straight transition arrow** (`straightConnector1`):
```xml
<p:cxnSp xmlns:p="http://schemas.openxmlformats.org/presentationml/2006/main"
         xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">
  <p:nvCxnSpPr>
    <p:cNvPr id="{NID}" name="Transition Arrow"/>
    <p:cNvCxnSpPr/>
    <p:nvPr/>
  </p:nvCxnSpPr>
  <p:spPr>
    <a:xfrm>
      <a:off x="{off_x_emu}" y="{off_y_emu}"/>
      <a:ext cx="{gap_emu}" cy="0"/>
    </a:xfrm>
    <a:prstGeom prst="straightConnector1"><a:avLst/></a:prstGeom>
    <a:ln w="38100"><a:tailEnd type="triangle"/></a:ln>
  </p:spPr>
  <p:style>
    <a:lnRef idx="1"><a:schemeClr val="accent1"/></a:lnRef>
    <a:fillRef idx="0"><a:schemeClr val="accent1"/></a:fillRef>
    <a:effectRef idx="0"><a:schemeClr val="accent1"/></a:effectRef>
    <a:fontRef idx="minor"><a:schemeClr val="tx1"/></a:fontRef>
  </p:style>
</p:cxnSp>
```

**Elbow branch connector** (`bentConnector2`): same structure with `prst="bentConnector2"` and `cx={delta_x_emu} cy={vertical_drop_emu}`.

### python-pptx construction (deep-copy pattern)

python-pptx does not expose `bentConnector2` or a fluent API for connector styling. The reliable approach is to build the first combination once (either by hand in PowerPoint or with a reference script), then `deepcopy` the XML elements for subsequent transitions.

```python
from copy import deepcopy
from pptx.oxml.ns import qn

NS_A = "http://schemas.openxmlformats.org/drawingml/2006/main"
NS_P = "http://schemas.openxmlformats.org/presentationml/2006/main"
EMU_PER_INCH = 914400


def clone_connector(template_el, *, off_x_in, off_y_in, cx_in, cy_in, name, nid):
    """Deep-copy a connector element and retarget it to a new geometry."""
    el = deepcopy(template_el)

    # Drop creationId extLst (tied to the source shape's identity)
    for ext_lst in list(el.iter(f"{{{NS_A}}}extLst")):
        ext_lst.getparent().remove(ext_lst)

    # Drop stCxn / endCxn (references to now-stale shape ids)
    for tag in ("stCxn", "endCxn"):
        for node in list(el.iter(f"{{{NS_P}}}{tag}")):
            node.getparent().remove(node)

    cNvPr = el.find(f"{{{NS_P}}}nvCxnSpPr/{{{NS_P}}}cNvPr")
    cNvPr.set("id", str(nid))
    cNvPr.set("name", name)

    xfrm = el.find(f"{{{NS_P}}}spPr/{{{NS_A}}}xfrm")
    xfrm.find(f"{{{NS_A}}}off").set("x", str(int(off_x_in * EMU_PER_INCH)))
    xfrm.find(f"{{{NS_A}}}off").set("y", str(int(off_y_in * EMU_PER_INCH)))
    xfrm.find(f"{{{NS_A}}}ext").set("cx", str(int(cx_in * EMU_PER_INCH)))
    xfrm.find(f"{{{NS_A}}}ext").set("cy", str(int(cy_in * EMU_PER_INCH)))

    return el


# Usage: with templates captured from an existing slide
tpl_straight = some_slide.shapes[idx_straight]._element
tpl_elbow    = some_slide.shapes[idx_elbow]._element

for i, (step_right, gap, callout_mid_x) in enumerate(transitions):
    arrow = clone_connector(
        tpl_straight,
        off_x_in=step_right, off_y_in=arrow_y,
        cx_in=gap, cy_in=0.0,
        name=f"Transition Arrow {i}", nid=next_id,
    )
    next_id += 1
    slide.shapes._spTree.append(arrow)

    branch = clone_connector(
        tpl_elbow,
        off_x_in=step_right, off_y_in=arrow_y,
        cx_in=(callout_mid_x - step_right),
        cy_in=(callout_top_y - arrow_y),
        name=f"Transition Branch {i}", nid=next_id,
    )
    next_id += 1
    slide.shapes._spTree.append(branch)
```

### Practical notes

- **Template shapes must come from an actual slide** — either a committed reference deck in the project or a scratch slide you build in PowerPoint and then delete after extraction. Elements constructed purely via `etree.fromstring` are more fragile and lose theme-color resolution.
- **Remove `stCxn` / `endCxn`.** These reference other shapes' `cNvPr/@id`. If you copy the connector to a different context, those ids are stale and Office will either silently ignore them or snap to the wrong shapes.
- **Assign fresh `cNvPr/@id`.** Duplicate ids in `spTree` cause PowerPoint repair warnings on load.
- **Don't attempt to rewrite `avLst`.** `bentConnector2` bends automatically based on the bounding box. Do not add `gd`/`prstGeom` adjustments unless a specific bend ratio is needed.
- **Reference slide:** the project that produced this reference is in `01-projects/AI Initiatives Board Overview/` (Workspace), slide 2.

---

## Future combos to document here

Add entries here as they appear in real decks. Candidates noticed but not yet codified:

- **Hero metric with annotation leader lines** — one large number with two or three short labels connected by thin leader lines (for KPI slides)
- **Dual-path decision** — one source node splitting into two labeled paths with different end states
- **Annotated timeline bar** — horizontal timeline with milestone diamonds and leader lines to event descriptions above/below the bar
- **Nested card sections** — card with a highlighted sub-card on top (for drawing focus within a category)

When any of these are built for real, promote the pattern into this file using the template above.
