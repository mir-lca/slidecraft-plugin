# Image generation for slides

## When to use images

Only generate images when they add to explanation, not mere illustration:
- Icons for multi-column category slides (3 pillars, 3 issues)
- Diagrams showing relationships (support model, org chart)
- Process flows

Do NOT generate images for:
- Decorative backgrounds
- Stock-photo-style illustrations
- Anything that could be communicated with text alone

## Icon generation

### Locked style prefix

All icons in a deck must use the same style prefix to ensure visual consistency:

```
STYLE = (
    "Flat vector icon. Pure white background. Single solid color #[ACCENT_COLOR]. "
    "Bold clean geometric shapes. No gradients. No shadows. No outlines. No text. "
    "Perfectly centered in a square canvas. 20% padding on all sides. "
    "Professional minimal icon design. Consistent stroke weight throughout."
)
```

Replace `#[ACCENT_COLOR]` with the brand's primary accent from template extraction.

This prefix works with any image generation API (Gemini, DALL-E, etc). The key constraint is consistency — every icon in the deck must use the identical prefix.

### Generation pattern

```python
# Example using Gemini (adapt for your preferred image generation API)
def generate_icon(prompt, output_path):
    # Call your image generation API with the prompt
    # Crop result to square if needed
    img = result_image
    w, h = img.size
    if w != h:
        side = min(w, h)
        left, top = (w - side) // 2, (h - side) // 2
        img = img.crop((left, top, left + side, top + side))
    img.save(output_path)

# Generate each icon sequentially (avoids file conflicts)
icons = [
    ("icon-name.png", f"{STYLE} Description of the icon concept."),
]
for filename, prompt in icons:
    generate_icon(prompt, f"icons/{filename}")
```

### Embedding in HTML

Resize to 256x256 before base64 encoding to keep file size manageable:

```python
import base64
from PIL import Image
from io import BytesIO

img = Image.open("icon.png").resize((256, 256), Image.LANCZOS)
buf = BytesIO()
img.save(buf, format='PNG', optimize=True)
b64 = base64.b64encode(buf.getvalue()).decode()
# Use as: src="data:image/png;base64,{b64}"
```

A 256x256 PNG icon encodes to ~20-50KB base64. Six icons total ~200KB — acceptable for a single-file HTML.

## Diagram generation

For relationship or flow diagrams, use a descriptive prompt:

```
"Clean professional diagram showing [relationship]. Use only #[ACCENT_COLOR] and
#[GREY_COLOR] on white background. Flat design, no 3D effects. Labels in Arial.
Arrows show direction of support. Two boxes connected by bidirectional arrows."
```

Embed diagrams at their native resolution (crop whitespace first).

## Mermaid diagrams (alternative)

For simple diagrams, consider generating Mermaid markup and rendering to SVG inline. This avoids the need for an image generation API entirely:

```html
<!-- Embed rendered SVG directly -->
<div class="diagram">
  <svg><!-- Mermaid-rendered SVG content --></svg>
</div>
```

Render Mermaid to SVG using the Mermaid CLI (`mmdc`) or the Mermaid JS library, then inline the SVG output.
