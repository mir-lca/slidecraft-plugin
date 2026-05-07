---
name: pptx-deck
description: |
  Generate native PowerPoint (.pptx) presentations using a `.potx`/`.pptx` brand template via python-pptx. Reuses template layouts, theme colors, fonts, and master chrome (footers, gradient bars, branded covers). Use for editable PowerPoint decks, weekly meetings, board material, template-compliant slides, or any context where the user references a `.pptx` file or wants editable output.
---

# pptx-deck

## Description

Build branded PowerPoint decks via `python-pptx` from a brand template (`.potx`/`.pptx`). Output is a native `.pptx` file that opens in PowerPoint, Keynote, and Google Slides; remains editable; and inherits theme colors, fonts, slide layouts, and master chrome from the template.

Default template lives at the plugin root: `Presentation1.potx` (Teradyne brand). Override per project by passing a different template path.

## When to use this skill

- User says "PowerPoint", ".pptx", "editable slides", "weekly meeting deck", or references an existing template
- Other files in the folder are `.pptx` and the user says "match the format"
- Brand compliance, collaborative editing, or template inheritance matters
- Final deck will live in SharePoint/OneDrive alongside other PPTX files

## When NOT to use

- User wants modern, viewport-filling visuals with no PowerPoint constraints → use `html-deck`
- User wants a script-to-deck conversion using TED narrative principles → use `ted-designer` first, then this skill renders the spec
- Quick text-only slides with no visual design → plain markdown is fine

## Workflow

1. **Locate template** — Default to plugin's `Presentation1.potx`. If a project-specific template is provided, use that.
2. **Extract brand DNA** — Theme colors (`dk1`, `dk2`, `accent1`–`accent6`) and fonts from `ppt/theme/theme1.xml`. See @../../shared/template-extraction.md.
3. **Plan slides** — One key idea per slide. See @../../shared/design-rules.md for non-negotiable design constraints.
4. **Convert .potx → .pptx** — Content-type rewrite via `zipfile`. Purge sample slides (remove `sldIdLst` entries AND drop relationships AND free filenames).
5. **Map content to layouts** — Title slide uses native Cover/Transition layout (default: `Transition 3`). Content slides use `Title and Subtitle` placeholders. Never rebuild title slides with custom textboxes over rectangles.
6. **Build content shapes** — Cards, bullets, steps. One text frame per card; one text frame for all bullets in a group. Never one TextBox per bullet.
7. **Generate images if needed** — See @../../shared/image-generation.md for icon generation patterns.
8. **Verify** — Open the file, check: layouts inherited, fonts correct, theme colors applied, no duplicated chrome.
9. **Export to PDF if requested** — See @references/pdf-export.md.

## Workflow: Render from ted-designer spec

When the user has already produced a slide spec via `ted-designer`, skip the planning step and consume the spec directly.

1. **Validate spec** — Check `schema_version` is `1.0` (or compatible). Reject otherwise.
2. **Surface warnings** — Print any `story.structural_warnings` so the user can see them before render proceeds.
3. **Check image assets** — For each slide with `image.prompt` but null `image.asset_path`, halt and list missing prompts. The user runs image generation, fills in `asset_path`, and re-invokes.
4. **Apply brand** — Use `brand.template_path` if present; otherwise fall back to default `Presentation1.potx`.
5. **Render slide-by-slide** — Map each archetype to the appropriate PPTX layout:
   - `full-bleed-image` → custom layout with picture placeholder filling slide
   - `big-number` → typographic layout, large number placeholder + label
   - `quote` → text-heavy layout with quote + attribution
   - `typographic` → text-only layout, single oversized phrase
   - `chart` → chart layout with python-pptx chart construction
   - `contrast-pair` → two-column layout, two pictures + two labels
   - `section-divider` → use template's section divider layout
   - `blank` → use template's blank layout
6. **Embed speaker notes** — `speaker_notes.script` + `speaker_notes.transition_cue` go into the slide's notes pane
7. **STAR marker** — Add `<!-- STAR moment -->` comment in the slide notes for the slide where `is_star: true`

See @../ted-designer/references/spec-schema.md for full schema.

## Design rules (summary)

Full rules at @../../shared/design-rules.md. Non-negotiable:

1. **One idea per slide** — If you need "and", it's two slides
2. **Content fills the slide** — No dead zones; no shrinking content to fit a corner
3. **Body text minimum 18pt** — Anything smaller fails at presentation distance
4. **Use template layouts, not custom shapes** — The master provides chrome (gradient bars, footers, logos). Do not duplicate.
5. **Every element earns its place** — No decorative bullets, filler text, "overview" slides

## Anti-patterns (PPTX-specific)

- **Rebuilding the title slide with a dark rectangle + custom textboxes** — Use the native Cover/Transition layout. Default: `Transition 3`.
- **Duplicating master chrome** — Gradient bars, logos, and footers live in the master. Never redraw them on individual slides.
- **One TextBox per bullet** — Group bullets in a single text frame with paragraph levels.
- **Custom textboxes for title/subtitle** — Use the layout's `Title` and `Subtitle` placeholders.
- **Forgetting to purge sample slides** — `.potx` templates ship with example slides. Remove them via `sldIdLst` cleanup, relationship drop, and filename free.
- **Hardcoded brand colors** — Always extract from the theme; never hardcode hex values in code.

## Readability checklist

Run against every slide:

- [ ] Title readable from 3m away (min 32pt)
- [ ] Body text readable from 2m (min 18pt, prefer 20pt)
- [ ] Content uses template layouts, not custom shapes
- [ ] Theme colors applied via `MSO_THEME_COLOR`, not hardcoded
- [ ] No duplicated master chrome
- [ ] Card visual weight balanced across multi-card slides

## References

- **PPTX export workflow** — @references/pptx-export.md (template purge, branded covers, placeholders, card patterns, text sizing, OneDrive upload)
- **Composite shape patterns** — @references/shape-combinations.md (multi-shape patterns for value layouts, process flows, three-panel)
- **PDF export** — @references/pdf-export.md (LibreOffice headless, PowerPoint AppleScript, Windows COM)
- **Brand extraction** — @../../shared/template-extraction.md (theme XML parsing, color/font extraction)
- **Design rules** — @../../shared/design-rules.md (12 shared rules across all slidecraft skills)
- **Image generation** — @../../shared/image-generation.md (icon style, base64 embedding for PPTX)

## Success criteria

- Native `.pptx` opens cleanly in PowerPoint, Keynote, Google Slides
- All slides use template layouts (no orphan custom textboxes)
- Theme colors and fonts match the template
- Master chrome (gradient bars, footers, logos) appears on every slide via the master, not duplicated per slide
- Title slide uses the branded Cover/Transition layout
- Body text >= 18pt everywhere
- No AI-slop anti-patterns present
- File remains editable downstream (no flattened images of text)
