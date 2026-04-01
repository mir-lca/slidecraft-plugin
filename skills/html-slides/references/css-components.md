# CSS components

## Base slide

Every slide is a `<section class="slide">` with:
- `width: 100vw; height: 100vh;`
- `scroll-snap-align: start;`
- `display: flex; flex-direction: column;`
- `padding: 5vh 6vw 6vh;` (generous edge padding)
- `::after` pseudo-element for accent bar at bottom

## Title slide

```html
<section class="slide title-slide">
  <div class="title-bar"></div>
  <h1 class="slide-title">Deck Title</h1>
  <p class="slide-subtitle">Author — Date</p>
</section>
```

- Dark background (--dark or --navy)
- Centered text, largest font size in the deck (clamp 2.8rem-4.5rem)
- Small accent bar above title (80px wide gradient)
- Subtitle at 55% opacity white

## Card grid

```html
<div class="card-grid cols-3">
  <div class="card">
    <div class="card-label">CATEGORY</div>
    <div class="card-heading">Heading</div>
    <p class="card-text">Description text</p>
  </div>
  <!-- more cards -->
</div>
```

**Critical layout rules — filling the viewport:**

Cards must fill the available space below the title. Two strategies depending on content density:

**Default pattern (works for all content densities):**
Grid stretches, cards cap at `max-height: 65vh`, content centers within:
```css
.card-grid {
  display: grid;
  flex: 1;                  /* grid stretches to fill slide */
  align-content: center;    /* rows center (works because max-height caps them) */
}
.card {
  display: flex;
  flex-direction: column;
  justify-content: center;  /* content centers within capped card */
  max-height: 65vh;         /* prevents over-stretching on sparse slides */
}
```

**For sparse fixed-layout slides (agenda 2x2, small items):**
Use `margin-block: auto` — grid stays at natural content size, centered vertically:
```css
.agenda { display: grid; margin-block: auto; }
```

**WARNING:** Do NOT combine `flex: 1` with `align-content: center` WITHOUT `max-height` on cards.
Grid rows auto-expand to fill a stretched container, consuming all space — `align-content`
has nothing to redistribute. The `max-height` cap on cards prevents row expansion, allowing
`align-content: center` to work correctly.

- Card padding: `2.8rem 2.5rem` minimum (3rem for 2-col layouts)
- Card body text: minimum `1.15rem` (use `clamp(1.15rem, 1.3vw, 1.25rem)`)
- Card heading: minimum `1.25rem`
- Label: `0.85rem` uppercase, letter-spacing 0.14em

**Content vertical alignment within cards:**

The default `.card` uses `justify-content: center` to vertically center content. Override per-card when needed:

- `justify-content: center` — Default. Content vertically centered in card. Best when cards have similar content density.
- `justify-content: flex-start` — Content top-aligned. Use when cards in the same row have uneven content lengths, or when the slide has a clear top-down reading order (e.g. label → heading → description).

Apply via inline style on individual cards:
```html
<div class="card" style="justify-content: flex-start;">
```

**Icon background compatibility:**

When using base64 icons with white backgrounds inside colored cards (accent-blue, accent-navy), the white square is visible. Two solutions:
1. **Preferred:** Use `.bordered` class (white card background) — icon backgrounds blend naturally
2. **Alternative:** Regenerate icons with transparent backgrounds

**Variants:**
- `.card.accent-blue` — navy background, white text (for the primary/positive option)
- `.card.accent-navy` — dark background, white text (for emphasis)
- `.card.bordered` — white background with subtle border (for secondary option)
- `.card.issue` — left border in gold/warning color

**With icons:**
```html
<div class="card">
  <img class="card-icon" src="data:image/png;base64,..." alt="">
  <div class="card-label">LABEL</div>
  <ul class="card-text">
    <li>Item one</li>
  </ul>
</div>
```

Icon: 56px square, `object-fit: contain`, `margin-bottom: 1.2rem`

## Compare layout

For before/after, today/future, or any two-panel comparison slides.

```html
<div class="compare-grid">
  <div class="compare-side">
    <div class="compare-label old">TODAY</div>
    <div class="compare-title">Current state</div>
    <p class="compare-text">Description of the problem or current approach.</p>
  </div>
  <div class="compare-divider"></div>
  <div class="compare-side">
    <div class="compare-label new">THE OPPORTUNITY</div>
    <div class="compare-title">Future state</div>
    <p class="compare-text">Description of the proposed solution or improvement.</p>
  </div>
</div>
```

**Critical layout rules — filling the viewport:**

Compare sides use card-like backgrounds at natural height, centered vertically in the available space:
```css
.compare-grid {
  display: grid;
  grid-template-columns: 1fr 2px 1fr;
  gap: 2vw;
  flex: 1;                  /* grid stretches to fill slide */
  align-content: center;    /* row centers (sides stay at natural height) */
}
.compare-side {
  padding: 3rem 2.8rem;
  background: var(--light-grey);
  border-radius: 14px;
}
.compare-divider {
  width: 2px;
  height: 60%;
  background: linear-gradient(180deg, transparent, var(--teal), transparent);
  align-self: center;
}
```

**WARNING:** Do NOT let compare-sides stretch to fill the grid row. Compare panels have sparse text content (label + title + 3-4 lines). Stretching creates tall grey boxes with content lost in the middle. Keep sides at natural height and center the row with `align-content: center`.

- Compare label: `1rem` uppercase, letter-spacing 0.14em. Use `.old` (coral) and `.new` (teal) variants.
- Compare title: `clamp(1.8rem, 2.2vw, 2.1rem)` bold
- Compare body text: `clamp(1.3rem, 1.5vw, 1.45rem)`, line-height 1.7
- Label margin-bottom: `1.4rem`
- Title margin-bottom: `1.2rem`

## Architecture flow

For horizontal layer diagrams showing system architecture, pipelines, or process stages.

```html
<div class="arch-flow">
  <div class="arch-layer layer-collect">
    <div class="arch-num">1</div>
    <div class="arch-label">Collection</div>
    <div class="arch-desc">What changed in the organization?</div>
  </div>
  <div class="arch-layer layer-semantic">
    <div class="arch-num">2</div>
    <div class="arch-label">Semantic extraction</div>
    <div class="arch-desc">What does it mean for our goals?</div>
  </div>
  <!-- more layers -->
</div>
```

**Critical layout rules — filling the viewport:**

Architecture flows use `margin-block: auto` for vertical centering with `min-height`/`max-height` constraints:
```css
.arch-flow {
  display: flex;
  align-items: stretch;
  gap: 4px;
  margin-block: auto;        /* centers vertically in slide */
  min-height: 40vh;           /* prevents thin strip */
  max-height: 55vh;           /* prevents over-stretching */
  width: 100%;
}
.arch-layer {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 4rem 2.5rem;
  text-align: center;
}
.arch-layer:first-child { border-radius: 14px 0 0 14px; }
.arch-layer:last-child { border-radius: 0 14px 14px 0; }
```

**WARNING:** Do NOT use `flex: 1` on `.arch-flow`. It stretches the flow to fill the full slide height, creating tall colored bars where content occupies only 15-20% of the block. Use `margin-block: auto` with `min-height`/`max-height` instead — this centers the flow at a proportional size.

- Layer number: `3.5rem`, font-weight 800, opacity 0.25
- Layer label: `clamp(1.3rem, 1.6vw, 1.5rem)` uppercase, letter-spacing 0.08em
- Layer description: `clamp(1.15rem, 1.3vw, 1.25rem)`, opacity 0.8
- Gap between layers: `4px` (tight, no arrows needed — color differentiation is sufficient)
- Each layer gets a distinct background color (use brand palette: dark navy, mid navy, teal, gold)

## Steps / timeline

```html
<div class="steps">
  <div class="step">
    <div class="step-num">1</div>
    <div class="step-content">
      <div class="step-title">Step title</div>
      <div class="step-desc">Description</div>
    </div>
  </div>
</div>
```

- Number circle: 40px, navy background, white text
- Step title: bold, 1.1-1.15rem
- Step description: grey, 0.95-1rem
- Gap between steps: 1.6rem
- Container: `max-width: 750px` for readability

## Agenda

```html
<div class="agenda">
  <div class="agenda-item">
    <span class="agenda-time">10 min</span>
    <div>
      <div class="agenda-title">Topic</div>
      <div class="agenda-desc">Details</div>
    </div>
  </div>
</div>
```

- 2-column grid for 4 items, 1-column for 3 or fewer
- Left border accent (3px navy)
- Time badge: bold navy, 0.8rem
- Title: bold, 1.05-1.1rem
- Description: grey, 0.9-0.95rem

## Two-column

```html
<div class="two-col">
  <div>
    <div class="col-header">Column A</div>
    <ul class="card-text">
      <li>Item</li>
    </ul>
  </div>
  <div>
    <div class="col-header">Column B</div>
    <ul class="card-text">
      <li>Item</li>
    </ul>
  </div>
</div>
```

- Column header: bold, navy, bottom border 2px
- Gap: 4vw between columns
- Text: standard card-text sizing (1.05rem+)

## Navigation and chrome

### Progress bar
```html
<div id="progress" style="width: 0%"></div>
```
Fixed top, 3px height, navy-to-teal gradient.

### Nav dots
```html
<nav id="nav-dots"></nav>
```
Fixed right, vertical, 8px dots. Active dot: navy, scale 1.5.

### Slide footer
```html
<span class="slide-num">03</span>
<span class="slide-footer">BRAND</span>
```
Bottom-left number, bottom-right brand name. Both in light-grey, small.

### Accent bar
Every slide gets a `::after` bottom bar:
```css
.slide::after {
  height: 4px;
  background: linear-gradient(90deg, var(--navy), var(--teal), var(--seafoam), var(--gold));
}
```

## Reveal animations

```css
.reveal {
  opacity: 0;
  transform: translateY(18px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}
.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}
.reveal.d1 { transition-delay: 0.08s; }
.reveal.d2 { transition-delay: 0.16s; }
.reveal.d3 { transition-delay: 0.24s; }
```

- IntersectionObserver at threshold 0.5 toggles `.visible`
- Reset on scroll-out so re-entering replays animation
- Delay increments: 0.08s per item (fast enough to feel snappy)

## Contact badge

For final slides, a small contact badge:
```html
<div class="contact">
  <strong>Name</strong> — email@domain.com
</div>
```
Light background, 0.9rem, subtle rounded rect.
