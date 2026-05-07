---
name: 'PowerPoint export'
description: 'Patterns for generating branded PPTX files using python-pptx, including template extraction, slide layout, card rendering, and text sizing'
applyTo: '**/*.py'
---

# PowerPoint export patterns

When generating Python scripts that produce PPTX presentations, follow these patterns. Uses `python-pptx` and `lxml`.

The brand template is `Presentation1.potx` in the repository root. Always use this as the base for new decks.

## Template extraction

Extract brand colors and fonts from a PPTX/POTX template before generating slides.

### Convert .potx to .pptx

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

### Extract colors from theme XML

```python
from lxml import etree

zf = zipfile.ZipFile('template.pptx')
theme_xml = zf.read('ppt/theme/theme1.xml')
root = etree.fromstring(theme_xml)
ns = {'a': 'http://schemas.openxmlformats.org/drawingml/2006/main'}

clrScheme = root.find('.//a:clrScheme', ns)
colors = {}
for child in clrScheme:
    tag = child.tag.split('}')[1]
    for attr_el in child:
        colors[tag] = attr_el.get('val', attr_el.get('lastClr', ''))
```

Map to constants: dk1 (body text), dk2 (navy/dark bg), lt1 (white), accent1-6 (brand palette).

### Extract fonts

```python
major = root.find('.//a:majorFont/a:latin', ns)  # heading font
minor = root.find('.//a:minorFont/a:latin', ns)  # body font
```

Font extraction is mandatory. Use the extracted font as the primary font throughout the generated deck.

## Slide generation

### Use native layout placeholders

Always use the template's built-in placeholders for titles and subtitles. Never create custom TextBox shapes for these.

```python
from pptx import Presentation

prs = Presentation('template.pptx')

# Find the "Title and Subtitle" layout
title_sub_layout = None
for layout in prs.slide_layouts:
    if layout.name == 'Title and Subtitle':
        title_sub_layout = layout
        break

slide = prs.slides.add_slide(title_sub_layout)
title_ph = slide.placeholders[0]    # idx 0 = TITLE
title_ph.text = "Slide title here"

sub_ph = slide.placeholders[13]     # idx 13 = BODY (subtitle)
sub_ph.text = "Subtitle text here"
```

### Do not duplicate template chrome

The slide master provides gradient bars, logos, footers, and slide numbers. Never add these programmatically. Check what the master provides first.

### Title / cover slides — always use a branded layout

Teradyne-style templates ship with multiple pre-branded cover and section-divider layouts (e.g. `Cover Slide`, `Cover Slide 2`, `Cover Slide 3`, `Title Slide`, `Transition 1`..`Transition 11`, `Thank You`). These layouts carry full-bleed backgrounds (wave patterns, gradients, accent imagery) built into the slide master.

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
3. Fill only the native placeholders — do not override colors or fonts on the title/subtitle runs. The master already styles them (light weight white title, teal/blue uppercase subtitle). Setting colors explicitly overrides and breaks the branded look.

```python
cover_layout = next(l for l in prs.slide_layouts if l.name == 'Transition 3')
slide = prs.slides.add_slide(cover_layout)
slide.placeholders[0].text = "Deck title"     # Title 1
slide.placeholders[1].text = "Subtitle text"  # Subtitle 2
# DO NOT add a dark rectangle, DO NOT add custom textboxes, DO NOT set run colors.
```

If a deck needs several section-divider slides (between topic groups), cycle through the `Transition N` layouts rather than duplicating one — each carries a different branded background.

### Remove sample slides from templates

Templates ship with example slides. Remove all `sldId` entries from `sldIdLst` in `presentation.xml` and delete matching slide XML files from the zip before generating.

## Card rendering

### Card backgrounds

```python
from pptx.util import Emu as emu, Pt
from pptx.enum.shapes import MSO_SHAPE

shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE, left, top, width, height)
shape.fill.solid()
shape.fill.fore_color.rgb = fill_color
shape.line.fill.background()
shape.shadow.inherit = False
if len(shape.adjustments) > 0:
    shape.adjustments[0] = 0.04  # subtle rounding
```

### Colored top borders

```python
line = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, x + emu(0.1), y, w - emu(0.2), Pt(3))
line.fill.solid()
line.fill.fore_color.rgb = accent_color
line.line.fill.background()
line.shadow.inherit = False
```

### Card text consolidation

Each card's content (label, heading, description) goes in ONE text frame with multiple paragraphs. Never separate TextBoxes.

```python
from pptx.oxml.ns import qn

txBox = slide.shapes.add_textbox(card_left + pad, card_top + pad, card_w - 2*pad, card_h - 2*pad)
tf = txBox.text_frame
tf.word_wrap = True
body_pr = tf._txBody.find(qn('a:bodyPr'))
if body_pr is not None:
    body_pr.set('anchor', 'ctr')  # vertical center

# Label paragraph
p_label = tf.paragraphs[0]
p_label.space_after = Pt(8)
run = p_label.add_run()
run.text = "LABEL TEXT"
run.font.size = Pt(12)
run.font.bold = True
run.font.color.rgb = label_color

# Heading paragraph
p_heading = tf.add_paragraph()
p_heading.space_after = Pt(12)
run = p_heading.add_run()
run.text = "Heading text"
run.font.size = Pt(22)
run.font.bold = True
run.font.color.rgb = heading_color

# Description paragraph
p_desc = tf.add_paragraph()
run = p_desc.add_run()
run.text = "Description text"
run.font.size = Pt(15)
run.font.color.rgb = desc_color
```

Rules:
- Pad the text frame 0.40" inward from card edges
- Use `space_before` / `space_after` for paragraph gaps
- Set anchor to `ctr` for vertical centering
- Never create separate TextBoxes for label, heading, and description within the same card

### Bullet text with proper hanging indent

All bullet items in a single text frame with multiple paragraphs. Never one TextBox per bullet.

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

    etree.SubElement(pPr, f'{{{ns}}}buChar').set('char', '\u2022')
    etree.SubElement(pPr, f'{{{ns}}}buSzPct').set('val', '100000')
    buClr = etree.SubElement(pPr, f'{{{ns}}}buClr')
    etree.SubElement(buClr, f'{{{ns}}}srgbClr').set('val', hex_color)

    run = p.add_run()
    run.text = text
    run.font.name = 'Arial'
    run.font.size = Pt(16)
    run.font.color.rgb = text_color
```

### Numbered circle icons

```python
from pptx.enum.text import PP_ALIGN

circ = slide.shapes.add_shape(MSO_SHAPE.OVAL, x, y, emu(0.5), emu(0.5))
circ.fill.solid()
circ.fill.fore_color.rgb = accent_color
circ.line.fill.background()
ctf = circ.text_frame
ctf.paragraphs[0].text = "1"
ctf.paragraphs[0].alignment = PP_ALIGN.CENTER
body_pr = ctf.paragraphs[0]._p.getparent().find(qn('a:bodyPr'))
if body_pr is not None:
    body_pr.set('anchor', 'ctr')
```

## Text sizing

PPTX text renders larger than HTML at equivalent sizes. Use these proven sizes:

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

In 3-column layouts, headings longer than ~20 characters will wrap. Allow `emu(0.65)` height for heading boxes.

## Text contrast per card

Check each card's fill color before choosing text color. Each card is independent.

| Card fill | Headings | Body text | Labels |
|-----------|----------|-----------|--------|
| Dark (#1A2744, #0D1D3B) | #FFFFFF | #E2E8F0 | #5CC4B8 (teal) |
| Light (#FFFFFF, #F4F5F7) | #1A2744 | #2D3748 | accent color |
| Teal (#2AA198) | #FFFFFF | #CCEEE8 | #CCEEE8 |

## Card sizing and centering

Size card height to content. Cards with less text should be shorter. Center cards vertically in the content zone:

```python
SW, SH    = emu(13.333), emu(7.5)
MX        = emu(0.8)
ZONE_TOP  = emu(1.22)        # below subtitle placeholder
ZONE_BOT  = SH - emu(0.75)   # above template footer
ZONE_H    = ZONE_BOT - ZONE_TOP
CONTENT_W = SW - 2 * MX
GAP       = emu(0.25)

def card_y(card_height):
    """Center a card of given height in the content zone."""
    return ZONE_TOP + (ZONE_H - card_height) // 2
```

Card height guidelines:
- 3-col with heading + bullets: `emu(4.4)`
- 2-col with label + bullets: `emu(4.2)`
- 2x2 agenda grid: `emu(2.3)` each

Match card heights within the same row (to the tallest), but rows can differ.

## Layer order

Add shapes in this order: card backgrounds first, then border-top shapes, then text boxes. Shapes render in insertion order.

Exception: axis/baseline markers in charts should be added **last** so they render on top of bar backgrounds and track fills.

## Content editing rules

### Label redundancy

When a keyword appears in the slide title or subtitle, drop it from body labels. The title provides context; repeating it in every label adds clutter.

Example: if the title says "62% of users never exceed included credit", body labels should be "Never exceeded", "Sometimes exceeded", "Consistently exceeded" — not "Never exceeded credit", "Sometimes exceeded credit", etc.

### Multi-sentence body text

Split multi-sentence text into separate paragraphs in PPTX. One sentence per paragraph improves readability at presentation distance. Never pack two sentences into a single paragraph in a text box or callout.

### Chart legend placement

Place color legends **below** the chart area by default, not above it. Data-first visual hierarchy: the audience should see the data bars/visualization before reading the legend key. Position legend dots at `ZONE_BOT - emu(1.2)` or just above the callout.

### Axis and baseline markers

Use minimum `Pt(4)` width for center axis lines in tornado, waterfall, or sensitivity charts. `Pt(2)` is invisible at presentation distance. Use a muted color (`LT_GREY` / `#B0B3B2`) so the marker anchors the chart without competing with data bars.

### Table body text sizing

Use `Pt(13)` for table data cells. `Pt(12)` is too small to read at presentation distance in a table context. Headers stay at `Pt(12)` bold (boldness compensates for the smaller size).

## Layout decision tree

| Structure | PPTX approach |
|-----------|--------------|
| Title + subtitle only | Dark bg rectangle + text boxes |
| Cards with labels/headings/bullets | Blank layout + ROUNDED_RECTANGLE + text boxes |
| Two equal columns | Two cards side by side, `(CONTENT_W - GAP) // 2` each |
| Three columns | Three cards, `(CONTENT_W - 2 * GAP) // 3` each |
| 2x2 grid (agenda) | Four cards in 2 rows x 2 cols with dot accents |
| Numbered steps | OVAL circles + text boxes in vertical sequence |
