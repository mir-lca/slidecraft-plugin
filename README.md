# html-slides

A Claude Code plugin for creating brand-matched slide presentations in three output formats: **HTML**, **PPTX**, and **PDF**.

## What it does

Generates presentation-grade slides with enforced design rules (one idea per slide, viewport-filling content, ≥1rem body text) and brand extraction (colors and fonts pulled from a `.potx` or `.pptx` template).

Three output paths, choose per use case:

| Output | Tool | Best for |
|--------|------|----------|
| **HTML** | Inline CSS + JS in a single self-contained file | Pitch decks, external shares, modern viewport-filling visuals, browser-native keyboard nav |
| **PPTX** | `python-pptx` + template layouts and theme | Editable PowerPoint decks, collaborative editing, matching existing template-based deck series (weekly meetings, board material) |
| **PDF** | Playwright (from HTML) or PowerPoint (from PPTX) | Email/file-share when editability is not needed |

All three share the same design rules and brand extraction. Pick the right output for the audience and workflow — do not default to HTML when the user wants an editable deck.

## Install

```bash
# From a Claude Code session:
/plugin install html-slides@your-marketplace

# Or load from local directory during development:
claude --plugin-dir /path/to/html-slides
```

## Usage

The skill activates when you ask Claude Code to create a presentation, deck, or slides. You can also invoke it directly:

```
/html-slides:html-slides
```

**Example prompts:**

HTML output:
- "Create a modern pitch deck about our Q2 results from these notes"
- "Convert this PowerPoint to a clean, visual HTML deck"
- "Build a presentation — make it look modern, not like PowerPoint"

PPTX output:
- "Build a PowerPoint deck matching the template in `Presentation1.potx`"
- "Generate a .pptx for the weekly meeting — match the other files in this folder"
- "Create editable slides using this brand template"

PDF output:
- "Export the deck to PDF for email"
- "Share a PDF version of these slides"

The skill asks which output format to use when the request is ambiguous.

## What's included

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — output-format decision matrix, workflows, design rules, anti-patterns, checklists |
| `references/base-structure.md` | Complete HTML/CSS/JS boilerplate template |
| `references/css-components.md` | HTML layout components (card grids, compare, architecture flow, steps, agenda) |
| `references/design-rules.md` | 12 shared design rules with implementation details and anti-patterns |
| `references/template-extraction.md` | Extract brand colors/fonts from PowerPoint templates |
| `references/pptx-export.md` | PPTX generation via python-pptx — template purge, branded covers, placeholders, card patterns, text sizing |
| `references/shape-combinations.md` | Multi-shape composite patterns for PPTX |
| `references/pdf-export.md` | Playwright script for 16:9 PDF export |
| `references/devtools-review.md` | Chrome DevTools measurement scripts for QA |
| `references/image-generation.md` | Icon generation patterns and base64 embedding |

## Design philosophy

Every rule exists because we hit the problem in practice:

- **One idea per slide** — if you need "and", split it
- **Content fills the viewport** — no dead zones, no gravity-sink
- **Body text ≥ 1rem** — anything smaller fails at presentation distance
- **Cards over bullets** — visual containment aids scanning
- **Pick the output that fits the workflow** — HTML for visual-first, PPTX for editable/templated, PDF for distribution
- **No AI slop** — explicit anti-pattern list prevents generated-looking output

## Requirements

- Claude Code with plugin support
- For PPTX export: Python + `python-pptx` + `lxml` (`pip install python-pptx lxml`)
- For PDF export from HTML: Python + Playwright (`pip install playwright && playwright install chromium`)
- For DevTools review: Chrome DevTools MCP server (optional but recommended)
- For image generation: Any image generation API (optional — decks work without images)

## License

MIT
