# slidecraft

A Claude Code plugin for brand-matched slide presentations. Three skills, shared design DNA.

## Skills

| Skill | Output | Best for |
|-------|--------|----------|
| **pptx-deck** | Native `.pptx` via python-pptx | Editable PowerPoint decks, weekly meetings, board material, template compliance |
| **html-deck** | Single-file HTML with inline CSS/JS + base64 | Pitch decks, external shares, modern viewport-filling visuals, browser-native nav |
| **ted-designer** | JSON slide spec | Convert a presentation script into a TED-style spec; hand off to pptx-deck or html-deck for rendering |

All three share the design rules in `shared/design-rules.md` and the brand extraction logic in `shared/template-extraction.md`. Pick the right output for the audience and workflow.

## Install

```bash
# From a Claude Code session:
/plugin install slidecraft@your-marketplace

# Or load from local directory during development:
claude --plugin-dir /path/to/slidecraft
```

## Usage

The right skill activates based on the request. You can also invoke directly:

```
/slidecraft:pptx-deck
/slidecraft:html-deck
/slidecraft:ted-designer
```

**Example prompts:**

PPTX:
- "Build a PowerPoint deck matching the template in `Presentation1.potx`"
- "Generate a .pptx for the weekly meeting matching the other files in this folder"
- "Create editable slides using this brand template"

HTML:
- "Create a modern pitch deck about our Q2 results from these notes"
- "Convert this PowerPoint to a clean, visual HTML deck"
- "Build a presentation — make it look modern, not like PowerPoint"

ted-designer:
- "Turn this script into a TED-style deck spec"
- "Design slides for this talk using TED principles"
- "Apply Duarte's contrast structure to my draft"

PDF export is a method on `pptx-deck` (LibreOffice/PowerPoint) and `html-deck` (Playwright). Just ask after generating the source deck.

## Repository layout

```
slidecraft/
├── .claude-plugin/plugin.json
├── README.md
├── Presentation1.potx              # default brand template (Teradyne)
├── shared/                         # cross-skill references
│   ├── design-rules.md
│   ├── template-extraction.md
│   └── image-generation.md
├── skills/
│   ├── pptx-deck/
│   │   ├── SKILL.md
│   │   └── references/  (pptx-export.md, shape-combinations.md, pdf-export.md)
│   ├── html-deck/
│   │   ├── SKILL.md
│   │   └── references/  (base-structure.md, css-components.md, devtools-review.md, pdf-export.md)
│   └── ted-designer/
│       ├── SKILL.md
│       └── references/  (deferred)
└── copilot/                        # parallel GitHub Copilot integration (PPTX-focused)
```

## Design philosophy

Every rule exists because we hit the problem in practice:

- **One idea per slide** — if you need "and", split it
- **Content fills the available space** — no dead zones, no gravity-sink
- **Body text minimum readable size** — anything smaller fails at presentation distance
- **Cards over bullets** — visual containment aids scanning
- **Pick the output that fits the workflow** — PPTX for editable/templated, HTML for visual-first, ted-designer for narrative-first
- **No AI slop** — explicit anti-pattern list prevents generated-looking output

## Requirements

- Claude Code with plugin support
- For PPTX: Python + `python-pptx` + `lxml` (`pip install python-pptx lxml`)
- For PDF from PPTX: LibreOffice (headless) or PowerPoint
- For PDF from HTML: Python + Playwright (`pip install playwright && playwright install chromium`)
- For DevTools review: Chrome DevTools MCP server (optional, recommended)
- For image generation: any image generation API (optional — decks work without images)

## License

MIT
