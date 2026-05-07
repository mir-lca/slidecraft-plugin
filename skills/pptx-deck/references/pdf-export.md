# PDF export (PPTX → PDF)

Convert a generated `.pptx` to PDF for distribution.

## Option A: LibreOffice (headless, cross-platform)

Prerequisites: LibreOffice installed (`brew install --cask libreoffice` on macOS).

```bash
soffice --headless --convert-to pdf --outdir /path/to/output /path/to/deck.pptx
```

- One PDF generated per input file
- Preserves theme colors, layouts, master chrome
- Output filename matches source basename

## Option B: PowerPoint (macOS)

Use AppleScript / `osascript` to drive PowerPoint export. Higher fidelity for complex animations and embedded media but macOS-only and requires PowerPoint installed.

```bash
osascript -e 'tell application "Microsoft PowerPoint" to save active presentation in POSIX file "/path/to/deck.pdf" as save as PDF'
```

## Option C: python-pptx + COM (Windows)

On Windows, drive PowerPoint via COM (`pywin32`). Skip unless deploying to Windows runners.

## Pick the right path

| Environment | Use |
|-------------|-----|
| Local macOS dev | LibreOffice (default) or PowerPoint if installed |
| CI/CD Linux runner | LibreOffice |
| Windows runner | python-pptx + COM |

LibreOffice is the default — works headless, no license, runs anywhere.

## Verification

After export, open the PDF and check:

- Slide count matches PPTX slide count
- Theme colors render correctly (gradient bars, accent colors)
- Fonts substituted cleanly (or embedded)
- No clipped content at slide edges
- 16:9 aspect ratio preserved
