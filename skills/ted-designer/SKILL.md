---
name: ted-designer
description: |
  Convert presentation scripts into TED-style slide specs grounded in Duarte/Reynolds/TED official design principles. Output is a JSON spec describing each slide's archetype, copy, image-generation prompt, and speaker notes with verbal transition cues. Designed for handoff to pptx-deck or html-deck for rendering. Enforces Duarte's contrast structure (what-is vs what-could-be) and STAR-moment identification before emitting the spec.
---

# ted-designer

## Description

Transform a written presentation script into a structured slide spec following TED-style design principles. Does not render slides directly — produces a portable JSON spec that downstream skills (`pptx-deck`, `html-deck`) consume to render the actual deck. Also produces image-generation prompts so the user can run image generation separately.

Core principles encoded in the skill:

- **One idea per slide** (TED, Duarte)
- **3-second glance test** — slide reads in <3 seconds (Duarte)
- **Subtract, don't add** — 90% destructive process (Duarte)
- **Slides as digital scenery** (Duarte)
- **Picture Superiority Effect** — images outlast words (Reynolds)
- **Avoid the slideument** (Reynolds)
- **Contrast as structure** — ebbs and flows; what-is vs what-could-be (Duarte)
- **STAR moments** — at least one engineered, unforgettable beat per talk (Duarte)
- **Storytelling spine** — beginning/middle/end; status quo → conflict → resolution

## When to use this skill

- User has a written script (full prose or detailed outline) and wants slides
- User wants TED-style: photo-driven, minimal text, strong narrative spine
- User asks to "design a deck", "build a TED-style talk", or references TED/Duarte/Reynolds principles
- Pre-step before invoking `pptx-deck` or `html-deck` for the actual render

## When NOT to use

- User wants format/template-compliant deck (use `pptx-deck` directly)
- User has slide-by-slide content already drafted (skip narrative analysis, go straight to renderer)
- User wants a quick text-only deck with no visual design

## Workflow

1. **Read script** — Full prose, outline, or transcript
2. **Story analysis** — Identify status-quo and what-could-be beats; flag missing contrast spine; locate or recommend a STAR moment
3. **Block** — If contrast structure is absent or no STAR moment exists, surface this and offer to refine the script before generating slides
4. **Segment** — Break script into one-idea beats. Each beat = one slide
5. **Archetype assignment** — Map each beat to a slide archetype (full-bleed image, big number/quote/typographic, chart, contrast pair, section divider, blank pause)
6. **Copy generation** — For each slide, write minimal on-slide text (0–6 words typical)
7. **Image prompts** — For image-bearing slides, write a detailed photorealistic/cinematic prompt (subject, composition, mood, lens/light cues)
8. **Speaker notes** — For each slide, capture the verbatim script segment + verbal transition cue to next slide
9. **Emit JSON spec** — Single file, one entry per slide, ready for handoff to `pptx-deck` or `html-deck`

## Slide archetypes

| Archetype | Use for | On-slide text | Image |
|-----------|---------|---------------|-------|
| `full-bleed-image` | Metaphor, atmospheric beat | 0–3 words optional caption | Required (full bleed) |
| `big-number` | Single statistic | The number + 1-line label | Optional background |
| `quote` | Pull-quote, attribution | Quote + attribution | Optional |
| `typographic` | 1–5 word statement | The statement | None |
| `chart` | Streamlined data viz with one takeaway | Title + highlighted insight | Chart svg/img |
| `contrast-pair` | Duarte what-is vs what-could-be | 2 short labels | Two paired images |
| `section-divider` | Chapter break | Section title | Optional muted background |
| `blank` | Intentional pause for speaker focus | None | None |

## Story enforcement

Before emitting the spec, verify:

- [ ] Script has a clear what-is (status quo) section
- [ ] Script has a clear what-could-be (resolution) section
- [ ] At least one STAR moment is identified or recommended
- [ ] Beats alternate or rhythm between contrast poles (no flat sequences)
- [ ] No beat carries multiple ideas (split into separate slides)

If any check fails, surface to user before generating slides. Offer concrete revision suggestions.

## Spec output schema

The skill emits a single JSON file. Schema details deferred to a later pass — for now, target this shape:

```json
{
  "title": "Talk title",
  "duration_target_minutes": 18,
  "story": {
    "what_is": "...",
    "what_could_be": "...",
    "star_moment_slide": 7
  },
  "slides": [
    {
      "index": 1,
      "archetype": "full-bleed-image",
      "on_slide_text": "",
      "image_prompt": "Photorealistic cinematic close-up of ...",
      "speaker_notes": "Verbatim script segment ...",
      "transition_cue": "...and that's when I realized..."
    }
  ]
}
```

## Handoff

After emitting the spec:

1. User runs image generation separately (gpt-image-2, nano-banana-pro) on the prompts
2. User invokes `pptx-deck` or `html-deck` with the spec + generated images
3. Renderer skill consumes the spec and produces the final deck

## References

- @references/archetypes.md — detailed archetype descriptions (deferred)
- @references/contrast-structure.md — Duarte contrast pattern detection (deferred)
- @references/spec-schema.md — full JSON schema for spec handoff (deferred)
- @../../shared/design-rules.md — shared design rules
- @../../shared/image-generation.md — image-gen prompt patterns

## Sources

Foundational principles drawn from:

- TED — [Create + prepare slides](https://www.ted.com/pages/create-prepare-slides)
- TED Blog — [10 tips for better slide decks](https://blog.ted.com/10-tips-for-better-slide-decks/)
- TEDx Speaker Guide
- Aaron Weyenberg — How to make slides the TED way
- Nancy Duarte — Slide:ology, Resonate, Slidedocs
- Garr Reynolds — Presentation Zen, Presentation Zen Design

## Success criteria

- Spec JSON is valid and self-contained
- Every slide has exactly one idea
- Contrast structure (what-is/what-could-be) is explicit
- STAR moment is identified
- Image prompts are detailed enough for one-shot generation
- Speaker notes include verbal transition cues per TED guidance
- Spec is renderer-agnostic (works with both pptx-deck and html-deck)
