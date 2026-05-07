# Presentation design

When asked to create a presentation, slide deck, or slides, generate a Python script using python-pptx that produces a branded .pptx file. Extract brand colors and fonts from the source template before generating.

## Design rules (non-negotiable)

1. **One idea per slide** — if you need "and" in the title, split it into two slides
2. **Content fills the slide** — cards vertically centered in the content zone, no dead zones between title and content
3. **Body text minimum Pt(16)** — anything smaller fails at presentation distance (2-5m)
4. **White space is hierarchy, not emptiness** — padding serves structure, not decoration
5. **Every element earns its place** — no "Overview" slides, no filler phrases, no decorative bullets restating titles
6. **Visual weight balance** — multi-column layouts need equal visual weight per column
7. **Headlines do the heavy lifting** — title + subtitle should communicate 80% of the message
8. **Cards over bullets** — rounded rectangles with padding and background beat flat bullet lists
9. **Color restraint** — max 3 content colors: primary (dark) for headings, secondary (grey) for body, accent for labels
10. **No redundant sections** — never present the same data in two visual formats

## Anti-patterns to avoid

These are hallmarks of AI-generated slides. Never do these:
- Accent lines under titles (decorative horizontal rules beneath headings)
- Centered body text (body text must be left-aligned)
- Repetitive card layouts (vary layouts across slides)
- Text-only slides with no visual structure
- "In this section we will discuss..." filler
- Decorative bullet points restating the title
- Excessive gradients and shadows
- Stock-photo-style AI illustrations

## Text size minimums (python-pptx)

| Element | Pt size | Notes |
|---------|---------|-------|
| Slide title | Pt(32) | Bold, dark color |
| Subtitle | Pt(18) | Regular weight, grey |
| Card label | Pt(12) | Uppercase, bold, letter-spacing |
| Card heading | Pt(22) | Bold. Allow `emu(0.65)` box height for 2-line wrap |
| Bullet text | Pt(16) | Via XML `buChar` in single text frame |
| Body paragraph | Pt(16) | For non-bulleted card text |

## Readability checklist

Run against every slide before delivering:
- [ ] Title readable from 3m? (min Pt(32))
- [ ] Body text readable from 2m? (min Pt(16))
- [ ] Content vertically centered in content zone?
- [ ] Cards/columns balanced in visual weight?
- [ ] Clear visual hierarchy? (title > subtitle > body > footnote)
