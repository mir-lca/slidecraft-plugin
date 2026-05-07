# PPTX export

## Purpose

Generate a branded PowerPoint deck that reuses a template's slide layouts and theme. Uses `python-pptx` + `lxml`.

Use this path when the user needs an editable `.pptx` (collaborative editing in PowerPoint, template compliance is mandatory, or the audience expects PowerPoint format). For non-editable visual-first decks, prefer the HTML path.

## Steps

### 1. Convert `.potx` to `.pptx`

`.potx` files need a content-type rewrite via `zipfile` before python-pptx can open them:

```python
import zipfile

with zipfile.ZipFile(potx_path, 'r') as zin:
    with zipfile.ZipFile(tmp_path, 'w') as zout:
        for item in zin.infolist():
            data = zin.read(item.filename)
            if item.filename == '[Content_Types].xml':
                data = data.replace(
                    b'presentationml.template.main+xml',
                    b'presentationml.presentation.main+xml'
                )
            zout.writestr(item, data)
```

### 2. Remove sample slides (template purge)

Templates ship with example slides. The common mistake — just clearing `sldIdLst` entries — leaves every original slide XML inside the package and downstream zip-clean steps guess wrong.

Correct purge — remove from slide list AND drop relationships AND free the filenames:

```python
from pptx.oxml.ns import qn

slide_rids = [sldId.get(qn('r:id')) for sldId in list(prs.slides._sldIdLst)]
for sldId in list(prs.slides._sldIdLst):
    prs.slides._sldIdLst.remove(sldId)
pres_part = prs.part
for rId in slide_rids:
    try:
        slide_part = pres_part.related_part(rId)
    except KeyError:
        continue
    try:
        pres_part.drop_rel(rId)
    except Exception:
        pass
    pkg = slide_part.package
    if slide_part.partname in pkg._parts:
        del pkg._parts[slide_part.partname]
```

After this, new slides are written as `slide1.xml`, `slide2.xml`, ... with no orphan parts.

### 3. Extract theme colors

Read `ppt/theme/theme1.xml` to get brand colors (dk1, dk2, lt1, lt2, accent1–6). Use as `RGBColor` constants. See @template-extraction.md.

### 4. Title / cover slides — always use a branded layout

Templates ship with multiple pre-branded cover and section-divider layouts (e.g. `Cover Slide`, `Cover Slide 2/3`, `Title Slide`, `Transition 1`..`Transition 11`, `Thank You`). These carry full-bleed backgrounds (wave patterns, gradients, accent imagery) built into the slide master.

**Never rebuild the title slide manually.** Do not add a full-slide dark `RECTANGLE` plus custom `TextBox` shapes for the title/subtitle. That bypasses the template's branded background and produces a flat, off-brand cover.

Instead:

1. List all cover / transition layouts the template exposes:
   ```python
   cover_layouts = [l for l in prs.slide_layouts
                    if any(k in l.name for k in ('Cover', 'Title Slide', 'Transition'))]
   for l in cover_layouts:
       print(l.name, [(p.placeholder_format.idx, p.name) for p in l.placeholders])
   ```
2. Prefer a layout with a visible branded background. For the Teradyne `Presentation1.potx` template, `Transition 3` is a good default cover (wave pattern on dark navy, native `Title 1` + `Subtitle 2` placeholders, template footer + accent edge).
3. Fill only the native placeholders — do not override colors or fonts on title/subtitle runs. The master already styles them correctly.

```python
cover_layout = next(l for l in prs.slide_layouts if l.name == 'Transition 3')
slide = prs.slides.add_slide(cover_layout)
slide.placeholders[0].text = "Deck title"     # Title 1
slide.placeholders[1].text = "Subtitle text"  # Subtitle 2
# DO NOT add a dark rectangle, DO NOT add custom textboxes, DO NOT set run colors.
```

For multi-divider decks, cycle through `Transition N` layouts — each carries a different branded background.

### 5. Content slides — use native layout placeholders for title and subtitle

For content slides, use the template's `Title and Subtitle` layout (or equivalent with TITLE and BODY placeholders). Set title via `placeholders[0]` (TITLE) and subtitle via `placeholders[13]` (BODY). This inherits template positioning, font, size, color:

```python
title_sub_layout = next(l for l in prs.slide_layouts if l.name == 'Title and Subtitle')
slide = prs.slides.add_slide(title_sub_layout)

slide.placeholders[0].text = "Slide title here"    # idx 0 = TITLE
slide.placeholders[13].text = "Subtitle text here" # idx 13 = BODY
```

**Never create custom TextBox shapes for titles and subtitles.** Custom TextBoxes at different positions break visual consistency with the template's built-in header styling.

**Never introduce a custom eyebrow** (e.g. "AI ENABLEMENT | TOPIC") above the title placeholder. The template's title + subtitle pair already provides information hierarchy. Eyebrows pushed above the title break the master's vertical rhythm and duplicate the subtitle's role.

For slides that need no title (e.g. full-bleed data tables), use the Blank layout.

### 6. Do not duplicate template chrome

The slide master provides gradient bars, logos, footers, slide numbers. Never add these programmatically. Open the template in PowerPoint and look at a blank slide — note what appears automatically. Only add content the template does NOT provide.

### 7. Placeholder text helper (preserves master styling)

Setting `placeholder.text = "..."` can strip master-defined run properties in some python-pptx versions. Use this helper to clear existing runs and insert a fresh run while leaving the paragraph's `pPr` (alignment, indent, master font) intact:

```python
def set_placeholder_text(ph, text, *, size=None, bold=None, color=None, align=None):
    tf = ph.text_frame
    p0 = tf.paragraphs[0]
    for r in list(p0.runs):
        r._r.getparent().remove(r._r)
    for p in list(tf.paragraphs[1:]):
        p._p.getparent().remove(p._p)
    if align is not None:
        p0.alignment = align
    run = p0.add_run()
    run.text = text
    if size is not None: run.font.size = Pt(size)
    if bold is not None: run.font.bold = bold
    if color is not None: run.font.color.rgb = color
    return run
```

Only override font attributes when you need a deliberate deviation from the master — otherwise call with just `(ph, text)` and let the template drive styling.

### 8. Build content shapes below the placeholder zone

On `Title and Subtitle` layout, the content zone starts at approximately `y = emu(1.22)` (below subtitle placeholder).

**Card backgrounds** — `ROUNDED_RECTANGLE` with solid fill, no line, subtle rounding:
```python
shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, width, height)
shape.fill.solid()
shape.fill.fore_color.rgb = fill_color
shape.line.fill.background()
shape.shadow.inherit = False
if len(shape.adjustments) > 0:
    shape.adjustments[0] = 0.04  # subtle rounding
```

**Colored top borders** — thin rectangles (`Pt(3)` height) at card top edge:
```python
line = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x + emu(0.1), y, w - emu(0.2), Pt(3))
line.fill.solid(); line.fill.fore_color.rgb = accent_color
line.line.fill.background(); line.shadow.inherit = False
```

**Numbered circle icons** — `OVAL` shapes with centered text:
```python
circ = slide.shapes.add_shape(MSO_SHAPE.OVAL, x, y, emu(0.5), emu(0.5))
circ.fill.solid(); circ.fill.fore_color.rgb = accent_color
circ.line.fill.background()
ctf = circ.text_frame
ctf.paragraphs[0].text = "1"
ctf.paragraphs[0].alignment = PP_ALIGN.CENTER
body_pr = ctf.paragraphs[0]._p.getparent().find(qn('a:bodyPr'))
if body_pr is not None: body_pr.set('anchor', 'ctr')
```

### 9. Layer order

Add card backgrounds FIRST, then border-top shapes, then text boxes. Shapes render in insertion order. Exception: axis/baseline markers in charts should be added LAST so they render on top of bar backgrounds and track fills.

## Card text consolidation

Each card's content (label, heading, description) goes in ONE text frame with multiple paragraphs — not separate TextBoxes. This ensures consistent spacing, proper text reflow on edit, and vertical centering:

```python
txBox = slide.shapes.add_textbox(card_left + pad, card_top + pad, card_w - 2*pad, card_h - 2*pad)
tf = txBox.text_frame
tf.word_wrap = True
body_pr = tf._txBody.find(qn('a:bodyPr'))
if body_pr is not None: body_pr.set('anchor', 'ctr')  # vertical center

# Label paragraph
p_label = tf.paragraphs[0]
p_label.space_after = Pt(8)
run = p_label.add_run()
run.text = "LABEL TEXT"
run.font.size = Pt(12); run.font.bold = True; run.font.color.rgb = label_color

# Heading paragraph
p_heading = tf.add_paragraph()
p_heading.space_after = Pt(12)
run = p_heading.add_run()
run.text = "Heading text"
run.font.size = Pt(22); run.font.bold = True; run.font.color.rgb = heading_color

# Description paragraph
p_desc = tf.add_paragraph()
run = p_desc.add_run()
run.text = "Description text"
run.font.size = Pt(15); run.font.color.rgb = desc_color
```

**Rules:**
- Pad the text frame inward from card edges (0.40") for breathing room
- Use `space_before` / `space_after` for gaps (label→heading: Pt(8), heading→description: Pt(12))
- Set anchor to `ctr` so content is balanced regardless of text length
- Never create separate TextBoxes for label, heading, description within the same card

## Bullet text with proper hanging indent

All bullet items in a single text frame with multiple paragraphs — never one TextBox per bullet:

```python
from lxml import etree
ns = 'http://schemas.openxmlformats.org/drawingml/2006/main'

txBox = slide.shapes.add_textbox(left, top, width, height)
tf = txBox.text_frame
tf.word_wrap = True

for i, text in enumerate(bullet_items):
    p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
    p.space_before = Pt(4)
    p.space_after = Pt(2)

    pPr = p._p.get_or_add_pPr()
    pPr.set('marL', str(int(Pt(18))))
    pPr.set('indent', str(int(-Pt(18))))

    etree.SubElement(pPr, f'{{{ns}}}buChar').set('char', '•')
    etree.SubElement(pPr, f'{{{ns}}}buSzPct').set('val', '100000')
    buClr = etree.SubElement(pPr, f'{{{ns}}}buClr')
    etree.SubElement(buClr, f'{{{ns}}}srgbClr').set('val', hex_color)

    run = p.add_run()
    run.text = text
    run.font.name = 'Arial'
    run.font.size = Pt(16)
    run.font.color.rgb = text_color
```

## Text sizing rules

PPTX renders larger than HTML at equivalent sizes. Proven sizes:

| Element | Pt size | Notes |
|---------|---------|-------|
| Slide title | Pt(32) | Bold, dark color |
| Subtitle | Pt(18) | Regular weight, grey |
| Card label | Pt(12) | Uppercase, bold, letter-spacing |
| Card heading | Pt(22) | Bold. Allow `emu(0.65)` box height for 2-line wrap |
| Bullet text | Pt(16) | Via XML `buChar` in single text frame |
| Body paragraph | Pt(16) | For non-bulleted card text |
| Step title | Pt(21) | Bold, for numbered step items |
| Step description | Pt(16) | Grey, below step title |
| Table data cells | Pt(13) | Pt(12) too small at presentation distance |
| Table headers | Pt(12) bold | Boldness compensates for smaller size |

**Critical heading wrap allowance.** In 3-column layouts, heading text boxes must be `emu(0.65)` tall (not `emu(0.45)`) to accommodate 2-line wrapping. A heading like "AI enablement accelerates" will wrap in a narrow column; too-short box causes visual overlap with body text below.

## Card sizing and centering

Size card height to content, then vertically center in the available zone between subtitle and bottom chrome:

```python
ZONE_TOP = SUB_Y + emu(0.5)      # below subtitle
ZONE_BOT = SH - emu(0.75)        # above template footer/gradient
ZONE_H   = ZONE_BOT - ZONE_TOP

def card_y(card_height):
    """Center a card of given height in the content zone."""
    return ZONE_TOP + (ZONE_H - card_height) // 2
```

Never stretch cards to fill the full zone. Size to content and center.

**Card height guidelines:**

| Layout | Card height | Why |
|--------|------------|-----|
| 3-col, heading + paragraph | `emu(4.4)` | Heading wraps to 2 lines in narrow cols |
| 3-col, heading + 2-3 bullets | `emu(4.4)` | Same wrap risk |
| 3-col with icon + label + bullets | `emu(4.4)` | Icon adds ~0.6in vertical |
| 2-col, label + 3-5 bullets | `emu(4.2)` | Wider cols, less wrapping |
| 2-col, label + heading + 3 bullets | `emu(4.0)` | Heading rarely wraps in half-width |
| 2x2 agenda grid | `emu(2.3)` each | Title + one-line description |

Match heights within the same row (to the tallest), but rows can differ.

## Text contrast per card

When a slide has multiple cards with different fill colors, each card's text must be colored for its own background — never a single color for the whole slide.

| Card fill | Text color | Use for |
|-----------|-----------|---------|
| Dark (#1A2744, #0D1D3B) | #E2E8F0 (light) | Body text, descriptions |
| Dark (#1A2744, #0D1D3B) | #FFFFFF (white) | Headings |
| Dark (#1A2744, #0D1D3B) | #5CC4B8 (teal) | Labels |
| Light (#FFFFFF, #F4F5F7) | #2D3748 (dark slate) | Body text, descriptions |
| Light (#FFFFFF, #F4F5F7) | #1A2744 (navy) | Headings |
| Teal (#2AA198) | #FFFFFF (white) | Headings |
| Teal (#2AA198) | #CCEEE8 (light teal) | Labels, body text |

Before setting font color, check the fill of the card shape the text sits on. Applies to both run font color AND bullet color (`buClr`).

## Content editing rules

**Label redundancy** — When a keyword appears in slide title/subtitle, drop it from body labels. Title provides context; repeating adds clutter. Example: title "62% of users never exceed included credit" → body labels "Never exceeded", "Sometimes exceeded" (not "Never exceeded credit", etc.).

**Multi-sentence body text** — Split multi-sentence text into separate paragraphs. One sentence per paragraph improves readability at presentation distance. Never pack two sentences into a single paragraph in a text box, callout, or footnote.

**Chart legend placement** — Place legends BELOW chart area, not above. Data-first hierarchy. Position legend dots at `ZONE_BOT - emu(1.2)` or just above callout.

**Axis and baseline markers** — Min `Pt(4)` width for center axis lines in tornado, waterfall, sensitivity charts. `Pt(2)` is invisible at presentation distance. Use muted color (`#B0B3B2`) so the marker anchors the chart without competing with data bars.

## Layout decision tree

| HTML structure | PPTX approach |
|----------------|--------------|
| Title + subtitle only (cover) | Native Cover/Transition layout + placeholders |
| Cards with labels/headings/bullets | Blank layout + ROUNDED_RECTANGLE + text boxes |
| Two equal columns | Two cards side by side, `(CONTENT_W - GAP) // 2` each |
| Three columns | Three cards, `(CONTENT_W - 2 * GAP) // 3` each |
| 2x2 grid (agenda) | Four cards in 2 rows x 2 cols with dot accents |
| Numbered steps | OVAL circles + text boxes in vertical sequence |
| 1+2 split | Left card full height, right side has two cards stacked with "OR" separator |

## Spacing constants (13.33 × 7.5in slide)

When using `Title and Subtitle` layout, title placeholder (PH=0) is at `top=279887` (~0.306in, height ~0.704in) and subtitle placeholder (PH=13) is at `top=914589` (~1.000in, height ~0.280in). Subtitle bottom is ~1.28in. Content zone should begin with a small breathing gap below subtitle, not flush.

```python
SW, SH    = emu(13.333), emu(7.5)
EDGE_L    = emu(0.55)             # left/right margin (matches template master left edge)
EDGE_R    = emu(12.90)
ZONE_TOP  = emu(1.45)             # content zone: 0.17in gap below subtitle
ZONE_BOT  = emu(6.90)             # above template footer gradient at 7.02in
CONTENT_W = EDGE_R - EDGE_L       # usable width ~12.35in
GAP       = emu(0.25)             # between cards
```

## Brand reference — Teradyne `Presentation1.potx`

From theme1.xml, scheme "Teradyne 1":

| Token | Hex | Use |
|-------|-----|-----|
| dk1 (grey text) | `#535759` | Body text, subtitles |
| dk2 (navy) | `#0D1D3B` | Headings, dark card bg, title slide bg |
| lt1 (white) | `#FFFFFF` | Text on dark backgrounds |
| accent1 (blue) | `#003087` | Primary accent, border-tops, numbered circles |
| accent2 (sky) | `#3CB3E1` | Labels on dark cards |
| accent3 (teal) | `#197D90` | Labels, card accents, dot indicators |
| accent4 (gold) | `#F3BC44` | Bullet dots on dark cards |
| accent5 (seafoam) | `#58C2AD` | Third-card accents, category differentiation |
| accent6 (light grey) | `#B0B3B2` | De-emphasized borders, muted accents |
| Card border | `#E0E2E4` | Light card border color |
| Off-white | `#F7F8FA` | Agenda card fill, subtle backgrounds |

Font: Arial (headings and body) per template fontScheme. Template provides native gradient bar at bottom and TERADYNE footer.

## Board / executive overview patterns

Dense executive slides combine one hero metric, an enumerated coverage visual, and a supporting mechanism diagram. Three reusable patterns:

**1. Hero + scorecard grid.** Left column is a soft-fill rounded rectangle with the headline ratio (`N / total`), a one-line descriptor, and a stack of 3 supporting stats. Right column is a 4-column grid of dot-row coverage indicators grouped by category header.

- Hero panel width: `emu(3.20)`, left-aligned at `EDGE_L`
- Hero big number: `Pt(78)` bold accent; denominator `Pt(40)` muted; both on one centered line
- Hero descriptor: `Pt(11)` muted, centered below
- Hero stats: 3 rows × `emu(0.32)`, number right-aligned at `Pt(18)` bold accent, label left-aligned at `Pt(11)` dark
- Grid column widths: `(grid_w - gap*(n-1)) / n` with `gap = emu(0.20)`
- Column header: navy rounded rectangle, `Pt(11)` bold white, `emu(0.40)` tall

**2. Dot-row coverage cell (compact list).** For enumerations of 6+ items, drop filled chip rectangles in favor of colored-dot markers + text on neutral background. Cuts row height from `emu(0.45)` (filled chip) to `emu(0.30)` (dot row):

```python
def add_dot_row(slide, x, y, w, h, label, started, *, text_size=11):
    dot_d = emu(0.16)
    dot_y = y + (h - dot_d) // 2
    dot = slide.shapes.add_shape(MSO_SHAPE.OVAL, x + emu(0.05), dot_y, dot_d, dot_d)
    dot.shadow.inherit = False
    dot.fill.solid(); dot.fill.fore_color.rgb = GREEN if started else GREY_MK
    dot.line.fill.background()
    tb = slide.shapes.add_textbox(x + emu(0.30), y, w - emu(0.35), h)
    tf = tb.text_frame
    tf.margin_left = 0; tf.margin_right = 0
    tf.margin_top = 0; tf.margin_bottom = 0
    tf.vertical_anchor = MSO_ANCHOR.MIDDLE
    p = tf.paragraphs[0]; p.line_spacing = 1.0
    r = p.add_run(); r.text = label
    r.font.name = 'Arial'; r.font.size = Pt(text_size)
    r.font.bold = started
    r.font.color.rgb = DARK if started else MID
```

Use bold text + full-color dot for "active", regular + muted grey dot for "inactive". Do not use ✔/— glyph prefixes — the dot is the marker.

**3. Process flow with feedback loop (3-step chevron).** For decentralized models, mechanisms, lifecycle loops: 3 step cards with `RIGHT_ARROW` chevrons between them and a one-line caption naming the feedback loop.

- Section eyebrow: `Pt(10)` bold accent, uppercase, above the row
- Step card: white fill, `CARD_BD` border, `emu(1.35)` tall, rounded
- Number badge: `OVAL` `emu(0.45)` diameter, accent fill, `Pt(16)` bold white
- Step label: `Pt(12)` bold accent, uppercase
- Step description: `Pt(12)` dark, 1.20 line spacing, 1-2 short sentences
- Chevron between steps: `RIGHT_ARROW` `emu(0.35) × 0.36`, accent fill
- Cycle caption below: `Pt(10)` muted, centered, 1 line ("more champions → broader coverage → more impact")

Step-card geometry — compute step width symmetrically so chevrons sit at midpoint:

```python
step_w = (CONTENT_W - arrow_w * 2 - gap_x * 4) / 3
# For each step i in 0..2:
#   sx = EDGE_L + (step_w + gap_x + arrow_w + gap_x) * i
# Chevron (if i < 2):
#   ax = sx + step_w + gap_x
#   ay = step_top + step_h/2 - emu(0.18)
```

Choose process flow over bullet list whenever content describes a *mechanism* — how a system sustains itself, what makes it repeatable, how actors hand off. Bulleted descriptions lose causality that chevrons preserve.

## Three-panel value-delivery layout

For board-style value stories, use three equal panels: Delivered (left, soft fill + green accent) | In Flight (middle, soft fill + amber accent) | Completed list (right, white with border).

Each panel:
- Colored top accent bar: `emu(1.20) × emu(0.06)` at `y_top + emu(0.22)`
- Panel label: `Pt(11)` bold accent, uppercase, at `y_top + emu(0.30)`, em-dash separator
- Hero stat (dollar figure): `Pt(46)` bold accent navy, at `y_top + emu(0.75)`
- Annotation under stat: `Pt(12)` bold dark, at `y_top + emu(1.75)`
- Body content from `y_top + emu(2.20)` down

Panel heights fill content zone (`ZONE_TOP` to `ZONE_BOT - emu(0.50)`) to leave room for full-width navy banner at bottom summarizing target/commitment.

Completed-list row sizing — compute row height to fit panel exactly: `rowh = (panel_h - list_y_offset - bottom_padding) / len(items)`. Avoids bug where last row falls outside panel.

## Reusable shape combinations

For multi-shape composite patterns (step-flow transitions with break-point callouts, hero metrics with leader lines), see @shape-combinations.md. When python-pptx does not expose a shape type directly (`bentConnector2`), deep-copy a reference element from an existing slide and retarget geometry.

## OneDrive upload for verification

Files >4MB require upload sessions via Graph API. Use Chrome DevTools MCP to verify in PowerPoint Online:

1. Upload via Graph API upload session (see microsoft-graph-api skill)
2. Navigate to the file's `webUrl` in Chrome
3. Use `take_snapshot` to find slide thumbnail UIDs, then `click` to navigate
4. Screenshot each slide

**OneDrive lock conflicts:** If PowerPoint Online has the file open, Graph API returns `resourceLocked` (423). Upload to new filename instead of overwriting, then rename after closing browser tab. The OneDrive sync client can also conflict — save generated files to `/tmp/` first, then upload via API.
