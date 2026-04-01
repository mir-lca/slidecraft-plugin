# PDF export

## Prerequisites

- Python Playwright installed: `pip install playwright && playwright install chromium`

## HTML preparation

The HTML deck must include a print media query to hide interactive-only elements:

```css
@media print {
  #progress, #nav-dots { display: none !important; }
  .slide { break-after: page; break-inside: avoid; }
}
```

Add this inside the `<style>` block before `</style>`.

## Export script

```python
from playwright.sync_api import sync_playwright

html_path = "/path/to/deck.html"
pdf_path = html_path.replace(".html", ".pdf")

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto(f"file://{html_path}", wait_until="networkidle")

    # Wait for fonts/images to load
    page.wait_for_timeout(2000)

    # Make all reveal animations visible (no scroll interaction in PDF)
    page.evaluate("document.querySelectorAll('.reveal').forEach(r => r.classList.add('visible'))")

    # Export as 16:9 landscape
    page.pdf(
        path=pdf_path,
        width="13.33in",    # 16:9 ratio at 7.5in height
        height="7.5in",
        print_background=True,
        margin={"top": "0", "right": "0", "bottom": "0", "left": "0"}
    )
    browser.close()
```

## Key parameters

| Parameter | Value | Why |
|-----------|-------|-----|
| `width` | `13.33in` | 16:9 ratio (13.33 / 7.5 = 1.777) |
| `height` | `7.5in` | Standard presentation height |
| `print_background` | `True` | Required for card backgrounds, accent bars, dark slides |
| `margin` | all `"0"` | Slides handle their own padding |
| `wait_until` | `"networkidle"` | Ensures base64 images are rendered |

## Expected output

- One page per slide (8 slides = 8 pages)
- File size: ~400-600KB with embedded icons
- Nav dots and progress bar hidden via `@media print`
- All reveal animations visible (forced via JS before export)
- Accent bars, dark backgrounds, card colors all preserved

## Troubleshooting

**Blank pages:** Increase `wait_for_timeout` to 3000-5000ms.

**Missing backgrounds:** Ensure `print_background=True`.

**Wrong aspect ratio:** Verify `width` and `height` — these are paper dimensions, not pixels.

**Reveal animations not showing:** The `evaluate` call must run BEFORE `page.pdf()`.
