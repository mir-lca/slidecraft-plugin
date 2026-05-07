---
name: html-deck
description: |
  Generate single-file HTML slide presentations with inline CSS/JS, base64 images, scroll-snap navigation, reveal animations, and brand-matched styling. Zero external dependencies. Use for pitch decks, external shares, modern viewport-filling visuals, or any context where the user wants a "modern" or "visual" deck unconstrained by PowerPoint.
---

# html-deck

## Description

Build presentation-grade HTML decks as a single self-contained file. Inline CSS, inline JS, base64-embedded images. Viewport-filling, browser-native keyboard navigation, scroll-snap between slides, reveal animations. Brand colors and fonts extracted from a `.potx`/`.pptx` template when provided.

Output: one `.html` file, no external assets. Open in any browser. Optional PDF export via Playwright.

## When to use this skill

- User says "modern", "visual", "clean", "pitch deck"
- User explicitly asks for HTML output or a browser-viewable deck
- External share where PowerPoint constraints would hurt the visual
- "Convert this PowerPoint to HTML" / "make these slides modern"

## When NOT to use

- User says "PowerPoint", ".pptx", "editable slides", or references an existing template → use `pptx-deck`
- Other files in the folder are `.pptx` and the user wants format match → use `pptx-deck`
- User wants Google Slides natively → neither path imports perfectly; use Google Slides directly

## Workflow

### Workflow 1: Create from content

1. **Gather content** — Read source (markdown, notes, transcript, PPTX text)
2. **Extract brand DNA** — If a template exists, run extraction. See @../../shared/template-extraction.md. Font extraction is mandatory — read `fontScheme` from `ppt/theme/theme1.xml` and use as primary `font-family`.
3. **Plan slides** — One key idea per slide. See @../../shared/design-rules.md.
4. **Select layouts** — Map each slide to a CSS component (see @references/css-components.md)
5. **Generate HTML** — Use base structure from @references/base-structure.md
6. **Generate images** — Icons via consistent style prefix; embed as base64 (resize to 256x256 first). See @../../shared/image-generation.md.
7. **Review and refine** — Open in browser, measure layout quality. See @references/devtools-review.md.
8. **Export to PDF if requested** — See @references/pdf-export.md.

### Workflow 2: Convert from PPTX

1. **Extract template DNA** — Colors, fonts, design elements
2. **Extract content** — Use `markitdown` or similar
3. **Reduce content** — Apply "one idea per slide"; cut bullets aggressively
4. **Map to HTML layouts** — Choose the best CSS component for each slide
5. **Generate HTML** — Single-file deck with brand colors and extracted font
6. **Add images** — Embed as base64
7. **Review** — Run the readability checklist on every slide

### Workflow 3: Render from ted-designer spec

When the user has already produced a slide spec via `ted-designer`, skip the planning step and consume the spec directly.

1. **Validate spec** — Check `schema_version` is `1.0` (or compatible). Reject otherwise.
2. **Surface warnings** — Print any `story.structural_warnings` so the user can see them before render proceeds.
3. **Check image assets** — For each slide with `image.prompt` but null `image.asset_path`, halt and list missing prompts. The user runs image generation, fills in `asset_path`, and re-invokes.
4. **Apply brand** — Use `brand.*` fields if present; otherwise fall back to default `Presentation1.potx` extraction.
5. **Render slide-by-slide** — Map each archetype to a CSS component or new pattern:
   - `full-bleed-image` → `<section class="slide bleed">` with full-viewport `<img>` + optional caption overlay
   - `big-number` → centered typographic layout, number in massive font + label below
   - `quote` → centered pull-quote with attribution, optional portrait
   - `typographic` → centered headline, no other content
   - `chart` → embed chart as SVG with highlighted series + takeaway title
   - `contrast-pair` → 2-column grid, two images + two labels
   - `section-divider` → use `.title-slide` class with optional muted background
   - `blank` → solid black/brand-neutral slide, no content
6. **Embed speaker notes** — `speaker_notes.script` + `speaker_notes.transition_cue` go into a `<aside class="notes">` block, hidden by default but available via `?notes=1` query param
7. **STAR marker** — Add `<!-- STAR moment -->` HTML comment at the STAR slide

See @../ted-designer/references/spec-schema.md for full schema.

## Design rules (summary)

Full rules at @../../shared/design-rules.md. Non-negotiable:

1. **One idea per slide** — If you need "and", it's two slides
2. **Content fills the viewport** — Cards/text vertically centered; no gravity-sink
3. **Body text minimum 1rem (16px)** — Anything smaller fails at presentation distance
4. **White space is intentional, not leftover**
5. **Every element earns its place**
6. **Consistent spatial rhythm** — Single CSS custom property (`--section-gap`) applied to all equivalent margins
7. **No redundant sections** — Never present the same data twice in different visual formats
8. **Viewport-relative sizing only** — Always `100vw`/`100vh`, never fixed physical units (`mm`/`cm`/`in`). Physical units only for `@page` print rules.

## Anti-patterns

Hallmarks of AI-generated slides — avoid:

- **Accent lines under titles** — Decorative horizontal rules scream "generated"
- **Centered body text** — Body should be left-aligned
- **Repetitive card layouts** — Vary across slides; don't 3-card-grid every slide
- **Text-only slides with no visual structure** — Use cards, steps, or columns
- **"In this section we will discuss..." filler**
- **Decorative bullets restating the title**
- **Excessive gradients and shadows** — One accent gradient (e.g. bottom bar) is enough
- **Stock-photo-style AI illustrations** — Images must add to explanation, never decorate

## Readability checklist

Run against every slide:

- [ ] Title readable from 3m away (min 2rem / 32px)
- [ ] Body text readable from 2m (min 1rem / 16px, prefer 1.1rem)
- [ ] Content vertically centered (no gravity-sink to bottom)
- [ ] Cards/columns balanced in visual weight
- [ ] Clear visual hierarchy (title > subtitle > body > footnote)

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

## Editing large HTML decks

Decks with embedded base64 images routinely exceed 100K tokens. Strategies:

**Reading:** Never attempt to read the entire file. Instead:
1. Use `Grep` with patterns to locate slide boundaries (`class="slide"`) and content (`card-label`, `card-heading`, `card-text`, `step-title`)
2. Use `Read` with `offset`/`limit` for specific line ranges
3. Use `Grep` with `-n` and `-A` for surrounding context

**Editing:** Use `Edit` with exact string matching against human-readable HTML — labels, headings, text content. Never match base64 strings.

**Structural awareness:** Slide boundaries are `<section class="slide">` ... `</section>`. `Grep -n` for `section class="slide"` gives the line map.

## References

- **Base HTML/CSS/JS template** — @references/base-structure.md
- **Layout components** — @references/css-components.md
- **DevTools measurement** — @references/devtools-review.md (Chrome DevTools scripts for QA)
- **PDF export** — @references/pdf-export.md (Playwright)
- **Brand extraction** — @../../shared/template-extraction.md
- **Design rules** — @../../shared/design-rules.md
- **Image generation** — @../../shared/image-generation.md

## Success criteria

- Single HTML file, zero external dependencies
- Every slide fits exactly in 100vh (no scrolling within slides)
- Keyboard navigation (arrows, spacebar) works
- PDF export produces one page per slide
- Body text >= 1rem everywhere
- Content fills viewport — no wasted dead zones
- Brand colors match source template (if provided)
- No AI-slop anti-patterns present
