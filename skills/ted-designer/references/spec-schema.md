# Spec schema

The ted-designer skill emits a single JSON file conforming to the schema below. Both `pptx-deck` and `html-deck` consume this format.

## Top-level shape

```json
{
  "schema_version": "1.0",
  "generated_at": "2026-05-07T01:30:00Z",
  "title": "string",
  "subtitle": "string | null",
  "speaker": "string | null",
  "duration_target_minutes": 18,
  "story": { ... },
  "brand": { ... },
  "slides": [ ... ]
}
```

## `story` block

Encodes the contrast analysis from `contrast-structure.md`.

```json
{
  "story": {
    "what_is": "One-sentence summary of the status-quo argument",
    "what_could_be": "One-sentence summary of the proposed reality",
    "sparkline": ["IS", "IS", "BRIDGE", "COULD", "IS", "COULD", "COULD"],
    "star_moment_slide": 14,
    "star_type": "vulnerable-disclosure | scale-shock | dramatization | demonstration | memorable-line",
    "structural_warnings": [
      {
        "gate": "ends_on_could",
        "severity": "warning | error",
        "message": "Final beat is descriptive, not a call to action",
        "suggested_fix": "Append a closing what-could-be beat after slide 18"
      }
    ]
  }
}
```

## `brand` block

Optional. Pass-through hints for the renderer. If absent, renderer extracts from default template (`Presentation1.potx`).

```json
{
  "brand": {
    "template_path": "/path/to/Presentation1.potx | null",
    "primary_color": "#003DA5 | null",
    "accent_color": "#00A0DF | null",
    "font_family": "Arial | null",
    "logo_path": "/path/to/logo.png | null"
  }
}
```

## `slides` array

Each slide is one beat. Indexed from 1.

```json
{
  "index": 1,
  "archetype": "full-bleed-image | big-number | quote | typographic | chart | contrast-pair | section-divider | blank",
  "tag": "IS | COULD | BRIDGE | META",
  "is_star": false,
  "on_slide_text": {
    "primary": "string | null",
    "secondary": "string | null",
    "attribution": "string | null"
  },
  "image": {
    "prompt": "string | null",
    "asset_path": "string | null",
    "alt": "string | null"
  },
  "image_pair": [
    { "prompt": "...", "asset_path": "...", "alt": "...", "label": "..." },
    { "prompt": "...", "asset_path": "...", "alt": "...", "label": "..." }
  ],
  "chart": {
    "type": "line | bar | column | pie | scatter | null",
    "data_csv": "string | null",
    "highlight_series": "string | null",
    "takeaway_title": "string | null"
  },
  "speaker_notes": {
    "script": "Verbatim script segment for this slide",
    "transition_cue": "Verbal hand-off cue to next slide",
    "duration_seconds": 45
  }
}
```

### Field-by-field rules

**`archetype`** (required) — One of the 8 values from `archetypes.md`. Determines which fields below are populated.

**`tag`** (required) — From contrast analysis. `IS`, `COULD`, `BRIDGE`, or `META`. Used by renderer for optional visual encoding (e.g. desaturated background for `IS` slides).

**`is_star`** (required) — Boolean. Exactly one slide in the deck should have `is_star: true`. The skill warns if zero or multiple.

**`on_slide_text`** — Text shown ON the slide. Keep minimal.
- `primary`: main text (number, headline, quote body, section title)
- `secondary`: supporting text (label, takeaway, subtitle)
- `attribution`: only for `quote` archetype

**`image`** — Single image. Used for `full-bleed-image`, `big-number` (background), `quote` (portrait), `section-divider` (background), `chart` (the chart asset itself).
- `prompt`: image-gen prompt (always populated when image is needed)
- `asset_path`: filled in by user after running image generation; renderer uses this
- `alt`: accessibility text

**`image_pair`** — Only for `contrast-pair` archetype. Exactly 2 entries.

**`chart`** — Only for `chart` archetype. CSV data inline, highlight key, takeaway title.

**`speaker_notes`** (required) — Per TED guidance.
- `script`: verbatim words
- `transition_cue`: words that bridge to next slide ("...and that's when..." style)
- `duration_seconds`: estimated time on this slide for pacing

### Archetype → required-fields matrix

| Archetype | Required fields |
|-----------|----------------|
| `full-bleed-image` | `image.prompt` |
| `big-number` | `on_slide_text.primary`, `on_slide_text.secondary` |
| `quote` | `on_slide_text.primary`, `on_slide_text.attribution` |
| `typographic` | `on_slide_text.primary` |
| `chart` | `chart.type`, `chart.data_csv`, `chart.takeaway_title` |
| `contrast-pair` | `image_pair` (exactly 2 entries with prompt + label) |
| `section-divider` | `on_slide_text.primary` |
| `blank` | (none — all fields optional) |

All slides require `speaker_notes.script` and `speaker_notes.transition_cue`.

## Renderer responsibilities

When `pptx-deck` or `html-deck` consumes a spec:

1. **Validate** against this schema. Reject if invalid.
2. **Surface warnings** from `story.structural_warnings` (e.g. log line, frontmatter comment).
3. **Resolve image paths** — If `image.asset_path` is null but `image.prompt` exists, surface to user: "Run image generation on these N prompts before rendering" with a list of prompts.
4. **Apply brand** — Use `brand.*` if present; otherwise fall back to default template extraction.
5. **Honor archetype** — Render each slide according to the archetype contract. Do not invent layouts.
6. **Encode tags optionally** — Renderer may apply visual treatment per `tag` (e.g. muted treatment for `IS`, accent for `COULD`). Optional, not required.
7. **Surface STAR** — Renderer may add a `<!-- STAR -->` marker at the STAR slide for editorial visibility.

## Versioning

Schema changes follow semver. Renderers must check `schema_version` and reject specs they cannot consume.

## Example minimal spec

```json
{
  "schema_version": "1.0",
  "generated_at": "2026-05-07T01:30:00Z",
  "title": "Why hiring is broken",
  "subtitle": null,
  "speaker": "Lourenco Castro",
  "duration_target_minutes": 12,
  "story": {
    "what_is": "Hiring relies on resumes that hide actual capability",
    "what_could_be": "Take-home work products replace resumes as the primary signal",
    "sparkline": ["IS", "IS", "BRIDGE", "COULD", "IS", "COULD", "COULD"],
    "star_moment_slide": 5,
    "star_type": "scale-shock",
    "structural_warnings": []
  },
  "brand": {
    "template_path": null,
    "primary_color": null,
    "accent_color": null,
    "font_family": null,
    "logo_path": null
  },
  "slides": [
    {
      "index": 1,
      "archetype": "full-bleed-image",
      "tag": "IS",
      "is_star": false,
      "on_slide_text": { "primary": null, "secondary": null, "attribution": null },
      "image": {
        "prompt": "Photorealistic close-up of a stack of paper resumes overflowing a desk inbox, harsh fluorescent lighting, cluttered office, shallow depth of field, no text",
        "asset_path": null,
        "alt": "Stack of paper resumes overflowing an inbox"
      },
      "speaker_notes": {
        "script": "We've been hiring the same way for fifty years. A resume hits an inbox, a recruiter scans it for ten seconds, and a human life gets a yes or a no.",
        "transition_cue": "And here's what that costs us.",
        "duration_seconds": 25
      }
    }
  ]
}
```
