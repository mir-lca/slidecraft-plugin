---
name: html-slides
description: |
  Create modern, brand-matched slide presentations. Supports three output formats: HTML (single-file, viewport-filling, browser-native), PPTX (native PowerPoint via python-pptx using a .potx/.pptx template), and PDF (16:9 export from either). Use for presentations, pitch decks, weekly meeting slides, converting PPTX to HTML or HTML to PPTX, and any slide creation where visual quality matters.
---

# HTML slides

## Description

Build presentation-grade slide decks in three output formats:

- **HTML** — single self-contained file with inline CSS/JS + base64 images. Viewport-filling, browser-native, keyboard-navigable. Best for pitch decks, external shares, modern visual feel.
- **PPTX** — native PowerPoint built via `python-pptx` using a `.potx`/`.pptx` template. Reuses template layouts, theme, master chrome (footers, gradient bars, branded covers). Best when collaborative editing or template compliance is required.
- **PDF** — 16:9 landscape export from either HTML (via Playwright) or PPTX (via PowerPoint). Best for email/file-share when editability is not needed.

All three paths share the same design rules (one idea per slide, viewport-filling content, ≥1rem body text) and brand extraction (colors, fonts from theme).

## Choose output format BEFORE building

| User says / situation | Output |
|----------------------|--------|
| "Modern / visual / clean deck", "pitch deck", no template mentioned | HTML |
| "Convert this PowerPoint to HTML", "make these slides modern" | HTML |
| ".pptx", "PowerPoint", "editable slides", "weekly meeting slides matching template", template file provided | **PPTX** |
| "Match the other slides in this folder" + other files are .pptx | **PPTX** |
| "Share a PDF", "export for email" | PDF (from HTML or PPTX) |

Ask the user if ambiguous. Never default to HTML when a template is provided or when the user references an existing PPTX deck they want to match — the correct answer is PPTX. HTML is the wrong output even if the skill name starts with "html".

## When NOT to use this skill

- User needs a quick text-only deck with no visual design (use plain markdown)
- User wants Google Slides natively (neither HTML nor PPTX import perfectly; use Google Slides directly)

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

1. **Extract template DNA** — Colors, fonts, design elements (see @references/template-extraction.md). **Font extraction is mandatory** — read the `fontScheme` from `ppt/theme/theme1.xml` and use the extracted font as the primary `font-family` in the generated HTML.
2. **Extract content** — Use `markitdown` or similar tool to get all slide text
3. **Reduce content** — Apply the "one idea per slide" rule; cut bullets aggressively
4. **Map to HTML layouts** — Choose the best CSS component for each slide
5. **Generate HTML** — Build single-file deck with brand colors and extracted font
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

## Workflow 3: Export to PPTX (branded PowerPoint)

See @references/pptx-export.md for the full workflow.

Quick summary:
1. Convert `.potx` to `.pptx` (content-type rewrite via `zipfile`)
2. Purge sample slides (remove `sldIdLst` entries AND drop relationships AND free filenames)
3. Extract theme colors (`dk1`, `dk2`, `accent1`–`accent6`) from `ppt/theme/theme1.xml`
4. Title slide: use a native Cover/Transition layout (default: `Transition 3`). Never rebuild with dark rectangle + custom textboxes.
5. Content slides: use `Title and Subtitle` layout placeholders for title + subtitle. Never create custom TextBoxes for these.
6. Build content shapes (cards, bullets, steps) below the placeholder zone. One text frame per card, one text frame for all bullets in a group — never one TextBox per bullet.
7. Do not duplicate template chrome (gradient bars, logos, footer — the master provides these).

Brand reference (Teradyne `Presentation1.potx`), text-sizing rules, card-centering math, three-panel value layouts, process flows, and OneDrive upload all live in the reference.

## Workflow 4: Export to PDF

See @references/pdf-export.md for the complete Playwright export script.

Quick summary:
1. Make all `.reveal` elements visible (no scroll interaction in PDF)
2. Hide interactive chrome (`#progress`, `#nav-dots`) via `@media print`
3. Export at 16:9 landscape (13.33in x 7.5in) with `print_background=True`
4. Output goes alongside the HTML file with same name, `.pdf` extension

## Workflow 5: Review with Chrome DevTools

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
