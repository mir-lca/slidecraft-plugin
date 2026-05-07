---
name: 'PPTX verification and PDF export'
description: 'Patterns for verifying generated PPTX files and exporting to PDF via PowerPoint or Playwright'
applyTo: '**/*.py'
---

# PPTX verification

After generating a .pptx file, verify the output by opening it in PowerPoint or uploading to OneDrive for browser-based review.

## OneDrive upload for verification

Files >4MB require upload sessions via Graph API. Upload to OneDrive, then open the `webUrl` in a browser to verify in PowerPoint Online.

Save generated files to `/tmp/` first, then upload via API. The OneDrive sync client can conflict with programmatic writes — avoid saving directly to synced folders.

If PowerPoint Online has the file open, Graph API returns `resourceLocked` (423). Upload to a new filename instead of overwriting, then rename after closing the browser tab.

## Verification checklist

For each slide in the generated deck:
- [ ] Title and subtitle use template placeholders (not custom TextBoxes)
- [ ] Card backgrounds render with correct fill colors
- [ ] Text contrast is readable on each card's background
- [ ] No template chrome is duplicated (gradient bars, logos, footers)
- [ ] Cards are vertically centered in the content zone
- [ ] Heading text does not overflow its text box (especially in 3-col layouts)
- [ ] Bullet hanging indent renders correctly
- [ ] Layer order is correct (backgrounds behind text)

## PDF export from PPTX

Use python-pptx cannot export PDF directly. Two approaches:

### Via LibreOffice (headless)

```python
import subprocess

pptx_path = "/path/to/deck.pptx"
output_dir = "/path/to/output/"

subprocess.run([
    "libreoffice", "--headless", "--convert-to", "pdf",
    "--outdir", output_dir, pptx_path
], check=True)
```

### Via PowerPoint COM automation (Windows only)

```python
import comtypes.client

powerpoint = comtypes.client.CreateObject("Powerpoint.Application")
powerpoint.Visible = 1
deck = powerpoint.Presentations.Open(pptx_path)
deck.SaveAs(pdf_path, 32)  # 32 = ppSaveAsPDF
deck.Close()
powerpoint.Quit()
```

### Via Playwright + PowerPoint Online

Upload to OneDrive, open the webUrl, and use Playwright to export:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto(web_url, wait_until="networkidle")
    page.wait_for_timeout(5000)
    # Use File > Export > PDF workflow via UI automation
```
