# Chrome DevTools MCP review

## Purpose

Use Chrome DevTools MCP to measure slide layout quality programmatically, replacing visual guesswork with precise metrics. Run after generating a deck and before delivering.

## Prerequisites

- Chrome DevTools MCP server connected
- HTML deck file accessible via `file://` path

## Workflow

### 1. Load the deck

```
mcp__chrome-devtools__navigate_page
  url: file:///path/to/deck.html
```

Wait for load, then optionally take a full screenshot to confirm rendering:

```
mcp__chrome-devtools__take_screenshot
```

### 2. Measure card fill ratios

Run this script via `mcp__chrome-devtools__evaluate_script` to get per-slide metrics:

```javascript
(() => {
  const slides = document.querySelectorAll('.slide');
  const results = [];

  slides.forEach((slide, i) => {
    const slideRect = slide.getBoundingClientRect();
    const cards = slide.querySelectorAll('.card');
    const grid = slide.querySelector('.card-grid');

    if (cards.length === 0) {
      results.push({ slide: i + 1, type: 'no-cards' });
      return;
    }

    const cardMetrics = Array.from(cards).map(card => {
      const cardRect = card.getBoundingClientRect();
      const style = getComputedStyle(card);
      const padTop = parseFloat(style.paddingTop);
      const padBot = parseFloat(style.paddingBottom);
      const interior = cardRect.height - padTop - padBot;

      // Measure actual content height
      let contentHeight = 0;
      Array.from(card.children).forEach(child => {
        contentHeight += child.getBoundingClientRect().height;
      });

      const fillRatio = interior > 0 ? (contentHeight / interior * 100).toFixed(1) : 0;

      return {
        height: Math.round(cardRect.height),
        interior: Math.round(interior),
        contentHeight: Math.round(contentHeight),
        fillPercent: parseFloat(fillRatio)
      };
    });

    // Grid gap analysis
    const gridRect = grid ? grid.getBoundingClientRect() : null;
    const gridGapAbove = gridRect ? Math.round(gridRect.top - slideRect.top) : null;
    const gridGapBelow = gridRect ? Math.round(slideRect.bottom - gridRect.bottom) : null;

    results.push({
      slide: i + 1,
      cards: cardMetrics,
      gridGapAbove,
      gridGapBelow,
      avgFill: parseFloat((cardMetrics.reduce((s, c) => s + c.fillPercent, 0) / cardMetrics.length).toFixed(1))
    });
  });

  return JSON.stringify(results, null, 2);
})()
```

### 3. Measure text sizes

Verify minimum text sizes meet readability thresholds:

```javascript
(() => {
  const checks = [
    { selector: '.slide-title', label: 'Slide title', min: 32 },
    { selector: '.card-heading', label: 'Card heading', min: 20 },
    { selector: '.card-text', label: 'Card text', min: 16 },
    { selector: '.card-label', label: 'Card label', min: 13 },
    { selector: '.step-title', label: 'Step title', min: 17 },
    { selector: '.step-desc', label: 'Step desc', min: 15 },
    { selector: '.agenda-title', label: 'Agenda title', min: 16 },
  ];

  const results = [];
  checks.forEach(({ selector, label, min }) => {
    const el = document.querySelector(selector);
    if (!el) return;
    const size = parseFloat(getComputedStyle(el).fontSize);
    results.push({
      label,
      selector,
      computedPx: Math.round(size * 10) / 10,
      minPx: min,
      pass: size >= min
    });
  });

  return JSON.stringify(results, null, 2);
})()
```

### 4. Take per-slide screenshots

For visual verification of specific slides:

```javascript
// Scroll to slide N (0-indexed)
document.querySelectorAll('.slide')[N].scrollIntoView();
```

Then `take_screenshot` to capture.

## Interpreting results

### Card fill ratio

| Fill % | Assessment | Action |
|--------|-----------|--------|
| 70-100% | Good | No changes needed |
| 60-70% | Acceptable | Consider bumping text size |
| 40-60% | Too sparse | Scale up text, spacing, icons; or switch centering strategy |
| < 40% | Poor | Redesign slide layout |

### Grid gap analysis

`gridGapAbove` and `gridGapBelow` show vertical dead zones around the card grid.

- **Balanced gaps** (within 20% of each other): Grid is well-centered
- **Large gap below**: Content is gravity-sinking to top — add `align-content: center` or `margin-block: auto`
- **Large gap above**: Unusual — check if title is consuming too much space

### Text size thresholds

| Element | Minimum | Preferred |
|---------|---------|-----------|
| Slide title | 32px (2rem) | 40-48px |
| Card heading | 20px (1.25rem) | 22-26px |
| Card body text | 16px (1rem) | 18-22px |
| Card label | 13px (0.85rem) | 14-15px |

## Iteration pattern

1. Run measurement script
2. Identify slides with fill < 60% or text below minimum
3. Apply CSS fix (bump sizes, adjust centering strategy)
4. Reload page: `mcp__chrome-devtools__navigate_page` with same URL
5. Re-run measurement script
6. Repeat until all slides pass thresholds
