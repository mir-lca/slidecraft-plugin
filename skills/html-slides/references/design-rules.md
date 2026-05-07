# Design rules

## 2026 presentation design principles

These rules synthesize the latest presentation design research and practical lessons from building HTML slide decks.

## The 10 rules

### 1. One idea per slide

Every slide communicates exactly one concept. If you need the word "and" in your slide title, split it into two slides. This is the single highest-impact rule for readability.

### 2. Content fills the viewport

Cards, text blocks, and visual elements must be vertically centered in the available space. Dead zones (large empty areas between title and content) destroy readability and make the deck feel unfinished.

**Implementation**: Content areas use `flex: 1` + `justify-content: center` + `gap` to distribute whitespace BETWEEN natural-height children — never inside them. Cards size to their content; whitespace lives between structural blocks (title area, card rows, discussion bar).

| Component | Content area strategy | Card sizing |
|-----------|----------------------|-------------|
| Card grid | `flex: 1; display: flex; flex-direction: column; justify-content: center; gap: 2.5vh;` | Natural height (no stretch) |
| Compare layout | `align-content: center` on grid | Natural height with padding |
| Architecture flow | `margin-block: auto` | `min-height: 40vh` + `max-height: 55vh` |
| Agenda/sparse grid | `margin-block: auto` | Natural content size |

**Core principle**: Whitespace between elements is design. Whitespace inside a card is dead space. Cards should have 80%+ usable fill ratio (content height / card height minus padding).

**Anti-pattern**: Stretching cards to fill viewport height. Grid rows with `align-content: stretch` or `grid-template-rows: 1fr` force cards taller than their content, creating empty interiors. Fix: let cards be natural height and use `justify-content: center` on the parent to distribute space between elements.

**Anti-pattern**: Using `flex: 1` on a content container without a centering strategy. Without `justify-content: center`, content gravitates to the top leaving dead space below.

### 3. Minimum text sizes

| Element | Minimum | Recommended | Sparse slide (few bullets) |
|---------|---------|-------------|---------------------------|
| Slide title | 2rem (32px) | 2.5-3rem | Same |
| Subtitle | 1rem (16px) | 1.15-1.25rem | Same |
| Card heading | 1.1rem (18px) | 1.2-1.35rem | 1.5-2rem |
| Card body text | 1rem (16px) | 1.05-1.15rem | 1.15-1.3rem |
| Compare heading | 1.4rem (22px) | 1.5-1.9rem | 1.7-2.1rem |
| Compare body text | 1rem (16px) | 1.1-1.25rem | 1.15-1.3rem |
| Labels/captions | 0.75rem (12px) | 0.8rem | Same |
| Footnotes | 0.8rem (13px) | 0.85rem | Same |

**Why**: Presentation distance is 2-5 meters. Text below 1rem becomes unreadable. Card body text at 0.9rem (the most common mistake) fails at any distance beyond arm's length.

**Sparse content rule**: When a slide has fewer than 6 bullet points total across all cards, scale up heading and body text sizes to the "sparse slide" column. Also increase `line-height` to 1.8-2.0 and bullet `margin-bottom` to 0.8-1rem. This fills card space naturally without adding filler content.

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

## PPTX-specific rules

When generating PPTX via python-pptx (Workflow 3), these additional rules apply on top of the HTML design rules:

### Text rendering differences
- PPTX text renders larger than HTML at equivalent sizes. A `Pt(16)` bullet in PPTX is roughly equivalent to `1.1rem` in HTML.
- Text wrapping in narrow PPTX columns is less predictable than CSS. Always allow extra height for heading text boxes (`emu(0.65)` minimum) to accommodate 2-line wrapping.
- Use XML `buChar` elements for proper PowerPoint bullets with `marL`/`indent` hanging indent. All bullet items in a single text frame with multiple paragraphs — never one TextBox per bullet.

### Card content structure
- Each card's content (label, heading, description) goes in ONE text frame with multiple paragraphs, not separate TextBoxes.
- Set text frame anchor to `ctr` for vertical centering within the card.
- Pad text frame 0.40" inward from card edges.
- Use `space_after` between paragraphs for consistent gaps.

### Text contrast per card
- Check each card's fill color independently before choosing text color.
- Dark fills (#1A2744, #0D1D3B): headings white, body #E2E8F0, labels teal.
- Light fills (#FFFFFF, #F4F5F7): headings #1A2744, body #2D3748.
- Never apply one text color to all cards on a slide — each card is independent.

### Template coexistence
- Never add elements the template master already provides (gradient bars, logos, footers, slide numbers). Check the blank layout first.
- Save generated files to `/tmp/` to avoid OneDrive sync conflicts. Upload via Graph API for verification.
- Use Chrome DevTools MCP to take screenshots of each slide in PowerPoint Online for visual verification.

### Card sizing strategy
- Size card height to content — cards with less text should be shorter. No large empty dead zones.
- Match card heights within the same row to the tallest card (so they align), but don't force all rows to the same height.
- In 3-col layouts, headings that are longer than ~20 characters will wrap. Account for this in card height.
- Use `card_y(height) = ZONE_TOP + (ZONE_H - height) // 2` for vertical centering.
- After placing cards, shift adjacent elements (footnotes, callouts) to avoid wasted space.

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
| Cards stretched to fill viewport with <50% fill | Let cards be natural height; use `justify-content: center` on parent flex to distribute space between elements |
| Whitespace inside cards instead of between elements | Remove `align-content: stretch` / `grid-template-rows: Xfr`; whitespace belongs between structural blocks, not inside containers |
| Architecture flow fills full slide height | Use `margin-block: auto` + `min-height: 40vh` / `max-height: 55vh` instead of `flex: 1` |
| `flex: 1` on container without sizing constraint | Always pair with `max-height`, `align-content: center`, or `min/max-height` range |
| Content area uses flex but cards don't stretch | Use `display: grid` + `grid-template-rows` on `.content-area` instead of flex for multi-row layouts |
