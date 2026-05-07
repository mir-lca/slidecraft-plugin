---
name: 'Icon and image generation for slides'
description: 'Consistent icon generation patterns and embedding in PPTX presentations via python-pptx'
applyTo: '**/*.py'
---

# Image generation for slides

## When to use images

Only generate images when they add to explanation:
- Icons for multi-column category slides (3 pillars, 3 issues)
- Diagrams showing relationships (support model, org chart)
- Process flows

Never generate images for: decorative backgrounds, stock-photo-style illustrations, anything communicable with text alone.

## Icon generation

### Locked style prefix

All icons in a deck must use the same style prefix for visual consistency:

```
Flat vector icon. Pure white background. Single solid color #[ACCENT_COLOR].
Bold clean geometric shapes. No gradients. No shadows. No outlines. No text.
Perfectly centered in a square canvas. 20% padding on all sides.
Professional minimal icon design. Consistent stroke weight throughout.
```

Replace `#[ACCENT_COLOR]` with the brand's primary accent from template extraction. Works with any image generation API (Gemini, DALL-E, etc).

### Embedding in PPTX

Add images to slides using python-pptx. Resize to a consistent size before embedding:

```python
from PIL import Image
from pptx.util import Emu as emu

# Resize icon to consistent dimensions
img = Image.open("icon.png").resize((256, 256), Image.LANCZOS)
img.save("icon_resized.png", optimize=True)

# Add to slide
pic = slide.shapes.add_picture("icon_resized.png", left, top, emu(0.6), emu(0.6))
```

### Icon background compatibility

When using icons with white backgrounds on dark card fills (#1A2744, #0D1D3B), the white square is visible. Solutions:
1. Place icons on light-fill cards only
2. Regenerate icons with transparent backgrounds
3. Add a small white rounded rectangle behind the icon as a deliberate design element
