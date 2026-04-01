# html-slides

A Claude Code plugin for creating modern, visually stunning HTML slide presentations.

## What it does

Generates single-file HTML presentations with zero external dependencies. Every deck is one self-contained `.html` file with inline CSS, inline JS, and embedded base64 images. Open it in any browser to present. Export to PDF via Playwright or Cmd+P.

**Why HTML instead of PowerPoint?**
- Viewport-filling layouts that look modern, not template-constrained
- Enforced design rules prevent the "wall of bullets" problem
- Single file — share via email, Slack, or any file system
- Works on any device with a browser
- Keyboard navigation (arrows, spacebar) built in

## Install

```bash
# From a Claude Code session:
/plugin install html-slides@your-marketplace

# Or load from local directory during development:
claude --plugin-dir /path/to/html-slides
```

## Usage

The skill activates automatically when you ask Claude Code to create a presentation, deck, or slides. You can also invoke it directly:

```
/html-slides:html-slides
```

**Example prompts:**
- "Create a presentation about our Q2 results from these notes"
- "Convert this PowerPoint to a modern HTML deck"
- "Build a pitch deck for the AI initiative — clean and visual"
- "Make slides from this markdown document"

## What's included

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — workflows, design rules, anti-patterns, checklists |
| `references/base-structure.md` | Complete HTML/CSS/JS boilerplate template |
| `references/css-components.md` | All layout components (card grids, compare, architecture flow, steps, agenda) |
| `references/design-rules.md` | 12 design rules with implementation details and anti-patterns |
| `references/template-extraction.md` | Extract brand colors/fonts from PowerPoint templates |
| `references/pdf-export.md` | Playwright script for 16:9 PDF export |
| `references/devtools-review.md` | Chrome DevTools measurement scripts for QA |
| `references/image-generation.md` | Icon generation patterns and base64 embedding |

## Design philosophy

Every rule exists because we hit the problem in practice:

- **One idea per slide** — if you need "and", split it
- **Content fills the viewport** — no dead zones, no gravity-sink
- **Body text >= 1rem** — anything smaller fails at presentation distance
- **Cards over bullets** — visual containment aids scanning
- **No AI slop** — explicit anti-pattern list prevents generated-looking output

## Requirements

- Claude Code with plugin support
- For PDF export: Python + Playwright (`pip install playwright && playwright install chromium`)
- For DevTools review: Chrome DevTools MCP server (optional but recommended)
- For image generation: Any image generation API (optional — decks work without images)

## License

MIT
