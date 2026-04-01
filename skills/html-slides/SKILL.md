---
name: html-slides
description: |
  Create modern, visually stunning HTML slide presentations from content or by converting PowerPoint decks. Outputs single-file HTML with scroll-snap navigation, reveal animations, and brand-matched styling. Use when creating presentations, converting PPTX to HTML slides, building pitch decks, or when user asks for modern/visual/clean slides. Supports PDF export via browser print.
---

# HTML slides

## Description

Build presentation-grade HTML slide decks that are more visual, modern, and readable than traditional PowerPoint. Each deck is a single self-contained HTML file with zero dependencies — inline CSS, inline JS, embedded images. Present directly in the browser; export to PDF via Playwright or Cmd+P.

Use this skill when:
- Creating a new presentation from content or notes
- Converting an existing PPTX to a modern HTML deck
- User asks for "modern", "visual", "clean", or "less verbose" slides
- Building internal or external pitch decks
- Any slide creation where visual quality matters

## When NOT to use

- User explicitly needs a .pptx file for editing in PowerPoint
- Template compliance is mandatory and non-negotiable
- Collaborative editing in PowerPoint/Google Slides is required

## Workflow 1: Create from content

1. **Gather content** — Read the source (markdown, notes, transcript, PPTX text)
2. **Extract brand DNA** — If a template exists, run template extraction (see @references/template-extraction.md)
3. **Plan slides** — One key idea per slide. See @references/design-rules.md for constraints
4. **Select layouts** — Map each slide to a layout component (see @references/css-components.md)
5. **Generate HTML** — Use the base structure from @references/base-structure.md
6. **Generate images** — If icons are needed, generate with a consistent style prefix (see @references/image-generation.md)
7. **Review and refine** — Open in browser, measure layout quality (see @references/devtools-review.md)
8. **Export** — PDF via Playwright (see @references/pdf-export.md), or Cmd+P, or share HTML directly

## Workflow 2: Convert from PPTX

1. **Extract template DNA** — Colors, fonts, design elements (see @references/template-extraction.md)
2. **Extract content** — Use `markitdown` or similar tool to get all slide text
3. **Reduce content** — Apply the "one idea per slide" rule; cut bullets aggressively
4. **Map to HTML layouts** — Choose the best CSS component for each slide
5. **Generate HTML** — Build single-file deck with brand colors and elements
6. **Add images** — Embed icons or diagrams as base64 (resize to 256x256 first)
7. **Review** — Check every slide against the 5-point readability checklist

## Design rules (summary)

Full rules at @references/design-rules.md. The non-negotiable rules:

1. **One idea per slide** — If you need "and", it's two slides
2. **Content fills the viewport** — Cards and text vertically centered, no dead zones
3. **Body text minimum 1rem (16px)** — Anything smaller fails at presentation distance
4. **White space is intentional, not leftover** — Padding serves hierarchy, not emptiness
5. **Every element earns its place** — No decorative bullets, no filler text, no "overview" slides
6. **Consistent spatial rhythm** — Define a single CSS custom property (e.g. `--section-gap`) and apply it to all equivalent margins: page edge padding, gap between header/tagline and content, gap between content sections and any subheaders. The left/right margin between content and page edges must visually match the vertical gap between the page header and the first content row. Use the same variable for both to guarantee equality.
7. **No redundant sections** — Never present the same data in two visual formats (e.g. pipeline with percentages AND a bar chart of the same percentages). Pick the best format once.
8. **Viewport-relative sizing only** — Always use `100vw`/`100vh` for slide and page dimensions, never fixed physical units (`mm`, `cm`, `in`). Physical units are only for `@page` rules (print/PDF). This applies to one-pagers too.

## Anti-patterns

Explicit list of things to avoid — these are hallmarks of AI-generated slides:

- **Accent lines under titles** — Decorative horizontal rules beneath headings scream "generated"
- **Centered body text** — Body text should be left-aligned; centered text is harder to scan
- **Repetitive card layouts** — Vary layouts across slides; don't use 3-card grid for every slide
- **Text-only slides with no visual structure** — Use cards, steps, or columns instead of bare text
- **"In this section we will discuss..."** filler — Delete and let the content speak
- **Decorative bullet points restating the title** — If the bullets just rephrase the heading, cut them
- **Excessive gradients and shadows** — One accent gradient (e.g. bottom bar) is enough
- **Stock-photo-style AI illustrations** — Only use images that add to explanation, never decoration

## Readability checklist

Run this against every slide before delivering:

- [ ] Is the title readable from 3m away? (min 2rem / 32px)
- [ ] Is the body text readable from 2m? (min 1rem / 16px, prefer 1.1rem)
- [ ] Is content vertically centered in available space? (no gravity-sink to bottom)
- [ ] Are cards/columns balanced in visual weight?
- [ ] Is there a clear visual hierarchy? (title > subtitle > body > footnote)

## CSS components (summary)

Full specs at @references/css-components.md:

| Component | Use for | Key class |
|-----------|---------|-----------|
| Title slide | Opening/closing | `.title-slide` |
| Card grid (2-col) | Comparisons, two options | `.card-grid.cols-2` |
| Card grid (3-col) | Three pillars, categories | `.card-grid.cols-3` |
| Compare layout | Before/after, today/future | `.compare-grid` |
| Architecture flow | Pipelines, layers | `.arch-flow` |
| Steps/timeline | Sequences, onboarding | `.steps` |
| Agenda | Meeting structure, timed items | `.agenda` |
| Two-column | Side-by-side lists | `.two-col` |

## Workflow 3: Export to PDF

See @references/pdf-export.md for the complete Playwright export script.

Quick summary:
1. Make all `.reveal` elements visible (no scroll interaction in PDF)
2. Hide interactive chrome (`#progress`, `#nav-dots`) via `@media print`
3. Export at 16:9 landscape (13.33in x 7.5in) with `print_background=True`
4. Output goes alongside the HTML file with same name, `.pdf` extension

## Workflow 4: Review with Chrome DevTools

See @references/devtools-review.md for measurement scripts.

Quick summary:
1. Navigate to the HTML file in Chrome
2. Run measurement script to get card fill ratios per slide
3. Take screenshots to visually verify
4. Target: card content fills 70-100% of card interior (after padding)
5. If fill ratio < 60%, scale up text/spacing or switch centering strategy

## Editing large HTML decks

HTML slide decks with embedded base64 images routinely exceed 100K tokens, making full-file reads impossible. Use these strategies:

**Reading:** Never attempt to read the entire file. Instead:
1. Use `Grep` with targeted patterns to locate slide boundaries (`class="slide"`) and content (`card-label`, `card-heading`, `card-text`, `step-title`, etc.)
2. Use `Read` with `offset` and `limit` to read specific line ranges (CSS block, individual slides)
3. Use `Grep` with `-n` and `-A` context lines to see surrounding structure

**Editing:** Use the `Edit` tool with exact string matching. Because base64 lines are unique, target the human-readable HTML around them — labels, headings, text content. Never try to match base64 strings.

**Structural awareness:** Slide boundaries are `<section class="slide">` ... `</section>`. A quick `Grep` for `section class="slide"` with `-n` gives you the line map for the entire deck.

## Success criteria

- Single HTML file, zero external dependencies
- Every slide fits exactly in 100vh (no scrolling within slides)
- Keyboard navigation (arrows, spacebar) works
- PDF export produces one page per slide
- Body text >= 1rem everywhere
- Content fills viewport — no wasted dead zones
- Brand colors match source template (if provided)
- No AI-slop anti-patterns present
