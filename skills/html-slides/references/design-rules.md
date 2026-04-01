# Design rules

## 2026 presentation design principles

These rules synthesize the latest presentation design research and practical lessons from building HTML slide decks.

## The 10 rules

### 1. One idea per slide

Every slide communicates exactly one concept. If you need the word "and" in your slide title, split it into two slides. This is the single highest-impact rule for readability.

### 2. Content fills the viewport

Cards, text blocks, and visual elements must be vertically centered in the available space. Dead zones (large empty areas between title and content) destroy readability and make the deck feel unfinished.

**Implementation**: Every flex-child content area needs an explicit centering and sizing strategy. The pattern varies by component type:

| Component | Centering strategy | Sizing constraint |
|-----------|-------------------|-------------------|
| Card grid | `flex: 1` + `align-content: center` | `max-height: 65vh` on cards |
| Compare layout | `flex: 1` + `align-content: center` | Natural height sides (no stretch) |
| Architecture flow | `margin-block: auto` | `min-height: 40vh` + `max-height: 55vh` |
| Agenda/sparse grid | `margin-block: auto` | Natural content size |

**Anti-pattern**: Using `flex: 1` on a content container without a sizing constraint. Grid rows auto-expand to fill a stretched container, creating over-sized empty boxes. Every `flex: 1` needs a corresponding `max-height`, `align-content: center`, or both to prevent this.

### 3. Minimum text sizes

| Element | Minimum | Recommended |
|---------|---------|-------------|
| Slide title | 2rem (32px) | 2.5-3rem |
| Subtitle | 1rem (16px) | 1.15-1.25rem |
| Card heading | 1.1rem (18px) | 1.2-1.35rem |
| Card body text | 1rem (16px) | 1.05-1.15rem |
| Labels/captions | 0.75rem (12px) | 0.8rem |
| Footnotes | 0.8rem (13px) | 0.85rem |

**Why**: Presentation distance is 2-5 meters. Text below 1rem becomes unreadable. Card body text at 0.9rem (the most common mistake) fails at any distance beyond arm's length.

### 4. White space is hierarchy, not emptiness

Padding around elements must serve the visual hierarchy:
- Between title and content: 1-2rem (close, because subtitle introduces content)
- Between cards: 2-3vw (breathing room without disconnecting)
- Card internal padding: 2.2-2.8rem (generous, readable)
- Slide edge padding: 5-6vw horizontal, 4-5vh vertical

**Anti-pattern**: Generous whitespace around tiny cards in the center of a slide. The whitespace isn't serving hierarchy — it's just wasted space.

### 5. Every element earns its place

Remove anything that doesn't directly help the audience understand the idea:
- No "Overview" or "Agenda" slides (unless the deck is 20+ slides)
- No decorative bullet points that restate the title
- No filler phrases ("In this section we will discuss...")
- No footnotes that nobody reads

### 6. Visual weight balance

When using multi-column layouts, the columns must have roughly equal visual weight:
- Similar number of items per column
- Similar text length per item
- If one column is naturally lighter, add visual mass (darker background, icon, border)

**Fix for asymmetry**: Move orphan items (like "out of scope" notes) outside the card grid as a subtle footnote below.

### 7. Headlines do the heavy lifting

The slide title + subtitle should communicate 80% of the message. If someone only reads titles, they should understand the deck. Bullets are supporting evidence, not the main argument.

**Test**: Read only the titles in sequence. Does the story make sense?

### 8. Cards over bullets

Cards (rounded rectangles with padding and background) are more readable than flat bullet lists:
- Visual containment aids scanning
- Background color creates hierarchy
- Padding prevents the "wall of text" effect
- Cards invite comparison between items

### 9. Animations serve attention, not decoration

Use reveal animations only to guide the audience's eye:
- Stagger cards appearing (0.1-0.15s delay between items)
- Fade + subtle translateY(15-20px) on enter
- Reset on scroll-out so re-entering a slide replays the animation

**Never**: Bouncing, spinning, scaling, or any animation that draws attention to itself.

### 10. Color restraint

Use a maximum of 3 colors for content elements:
- Primary (dark): titles, headings, card headings
- Secondary (grey): body text, descriptions
- Accent (brand color): labels, active states, key cards

Reserve additional brand colors for:
- Navigation elements (progress bar, nav dots)
- Accent bars and decorative elements
- Differentiation between card types (active vs passive, provides vs owns)

### 11. No redundant sections

Never present the same data in two different visual formats on the same slide or page. A pipeline/table showing staged time allocations and a bar chart showing the same percentages is redundant — pick the format that communicates best and use it once.

**Test**: For each section, ask "does this add new information or just re-visualize something already shown?" If the latter, merge or remove it.

**Common violations**:
- Table of metrics + chart of the same metrics
- Pipeline with percentages + bar chart of the same percentages
- Bullet list + card grid restating the same items
- Summary section that repeats content from detail sections

### 12. Viewport-relative sizing only

Never use fixed physical units (`mm`, `cm`, `in`, `pt`) for slide or page dimensions. Always use viewport-relative units (`100vw`, `100vh`, `vw`, `vh`) so content scales to any browser window or screen size.

**Correct**: `width: 100vw; height: 100vh;`
**Wrong**: `width: 297mm; height: 210mm;`

Physical units are only acceptable in `@page` rules for print/PDF export. The live HTML must always be viewport-relative.

This applies to one-pagers and single-page documents too — use `100vw`/`100vh` for the body, not fixed paper dimensions.

## Common mistakes and fixes

| Mistake | Fix |
|---------|-----|
| Cards pushed to bottom of slide | Add `align-content: center` to grid |
| Text too small in cards | Minimum 1rem for body, 1.1rem for headings |
| Title far from content | Reduce gap; subtitle should flow into content |
| One card much emptier than others | Move orphan content outside cards; rebalance |
| Decorative icons that add no meaning | Remove, or replace with meaningful diagram |
| "Overview" slide restating table of contents | Delete; let the deck speak for itself |
| Subtle grey text on light background | Ensure contrast ratio >= 4.5:1 |
| Same data shown as table AND chart | Pick one format; remove the redundant section |
| Fixed mm/cm/in dimensions on body | Use 100vw/100vh; reserve physical units for @page only |
| Compare panels stretch to fill viewport | Use `align-content: center` on grid; keep sides at natural height |
| Architecture flow fills full slide height | Use `margin-block: auto` + `min-height: 40vh` / `max-height: 55vh` instead of `flex: 1` |
| `flex: 1` on container without sizing constraint | Always pair with `max-height`, `align-content: center`, or `min/max-height` range |
