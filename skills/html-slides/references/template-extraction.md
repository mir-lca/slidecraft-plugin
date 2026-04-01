# Template extraction

## Purpose

Extract brand DNA from a PowerPoint template to build an HTML deck that feels native to the brand without being constrained by PowerPoint's layout limitations.

## What to extract

### Colors (from ppt/theme/theme1.xml)

```python
import zipfile, json
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

Map to CSS custom properties:
```css
:root {
  --navy: #[accent1];     /* Primary brand color */
  --dark: #[dk2];         /* Darkest background */
  --grey: #[dk1];         /* Body text */
  --light-grey: #[lt2];   /* Captions, footnotes */
  --green: #[accent2];    /* Secondary accent */
  --teal: #[accent3];     /* Tertiary accent */
  /* ... map all accent colors */
}
```

### Fonts (from ppt/theme/theme1.xml)

```python
major = root.find('.//a:majorFont/a:latin', ns)
minor = root.find('.//a:minorFont/a:latin', ns)
```

Map to CSS:
```css
body {
  font-family: '[minor font]', 'Helvetica Neue', sans-serif;
}
```

### Design elements

Inspect the slide master for recurring visual elements:

```python
from pptx import Presentation
prs = Presentation('template.pptx')
master = prs.slide_masters[0]
for shape in master.shapes:
    # Look for: gradient bars, logos, footer elements
```

Common elements to replicate in CSS:
- **Gradient accent bar** at bottom of every slide (`::after` pseudo-element)
- **Logo** in footer area (embed as base64 or use text)
- **Color blocks** for divider slides

### Background images

Extract from `ppt/media/` to identify:
- Full-bleed background photos (for divider slides)
- Logo files
- Decorative elements

```python
from PIL import Image
from io import BytesIO

for name in zf.namelist():
    if name.startswith('ppt/media/'):
        img = Image.open(BytesIO(zf.read(name)))
        # Check dimensions to classify:
        # 16:9 ratio = background photo
        # Small/square = logo or icon
```

## Output

A CSS `:root` block with all brand variables, plus notes on which design elements to replicate as CSS effects (gradient bars, color blocks, etc).
