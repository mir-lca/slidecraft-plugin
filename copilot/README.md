# Presentation design for GitHub Copilot

Custom instructions that teach GitHub Copilot to generate branded PowerPoint presentations using python-pptx, with proper template extraction, card rendering, and text sizing.

## Install

Copy the `.github/` folder and the template into your repository root:

```bash
cp -r .github/ /path/to/your-repo/.github/
cp Presentation1.potx /path/to/your-repo/
```

Copilot picks up the instructions automatically. Point it at `Presentation1.potx` when generating decks.

## What's included

```
.github/
├── copilot-instructions.md                        # Design rules, anti-patterns, text sizing
└── instructions/
    ├── pptx-export.instructions.md                # python-pptx generation, template extraction, card rendering
    ├── pdf-export.instructions.md                 # PPTX verification, PDF export via LibreOffice/Playwright
    └── image-generation.instructions.md           # Icon generation and PPTX embedding
Presentation1.potx                                 # Teradyne brand template (source for color/font extraction)
```

| File | Applies to | What it does |
|------|-----------|--------------|
| `copilot-instructions.md` | All files | 10 design rules, anti-patterns, readability checklist |
| `pptx-export` | `**/*.py` | python-pptx patterns, template extraction, card rendering, text sizing, contrast rules |
| `pdf-export` | `**/*.py` | PPTX verification checklist, PDF export via LibreOffice or Playwright |
| `image-generation` | `**/*.py` | Icon style prefix, PPTX embedding patterns |

## How it works

- `copilot-instructions.md` applies globally — Copilot follows the design rules in every interaction
- Scoped `.instructions.md` files activate only when working on Python files via `applyTo` globs
- In agent mode, Copilot can run the Python scripts end-to-end (template extraction, PPTX generation, PDF export)

## Example prompts

- "Create a presentation about our Q2 results using this PowerPoint template"
- "Generate a python-pptx script that creates a branded deck from template.potx"
- "Extract the brand colors and fonts from this PPTX template"
- "Add a 3-column card slide with these categories"
- "Export this PPTX to PDF"

## Requirements

- GitHub Copilot (Chat or Agent mode)
- `pip install python-pptx lxml Pillow`
- For PDF export: LibreOffice (headless) or Playwright
- For image generation: any image generation API (optional)
