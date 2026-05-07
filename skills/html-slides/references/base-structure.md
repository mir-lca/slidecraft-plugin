# Base structure

## HTML boilerplate

Every HTML slide deck follows this structure. The entire presentation is a single self-contained file — all CSS inline, all JS inline, all images embedded as base64.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Presentation Title</title>
  <style>
    /* === RESET === */
    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

    /* === BRAND VARIABLES === */
    :root {
      --navy: #1a2744;
      --dark: #0d1520;
      --teal: #2bb5a0;
      --grey: #4a5568;
      --light-grey: #a0aec0;
      --bg: #ffffff;
      --section-gap: 5vh;
      /* Override these with brand colors from template extraction */
    }

    /* === TYPOGRAPHY === */
    body {
      font-family: 'Helvetica Neue', Arial, sans-serif; /* OVERRIDE with template font — see template-extraction.md */
      color: var(--grey);
      background: var(--bg);
      overflow-x: hidden;
    }

    /* === SCROLL SNAP === */
    html {
      scroll-snap-type: y mandatory;
      overflow-y: scroll;
      scroll-behavior: smooth;
    }

    /* === BASE SLIDE === */
    .slide {
      width: 100vw;
      height: 100vh;
      scroll-snap-align: start;
      display: flex;
      flex-direction: column;
      padding: var(--section-gap) 6vw 6vh;
      position: relative;
      overflow: hidden;
    }

    /* Accent bar on every slide */
    .slide::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      height: 4px;
      background: linear-gradient(90deg, var(--navy), var(--teal));
    }

    /* === SLIDE HEADER === */
    .slide-title {
      font-size: clamp(2rem, 3vw, 2.8rem);
      font-weight: 700;
      color: var(--navy);
      margin-bottom: 0.3rem;
    }

    .slide-subtitle {
      font-size: clamp(1rem, 1.3vw, 1.15rem);
      color: var(--grey);
      margin-bottom: var(--section-gap);
    }

    /* === TITLE SLIDE === */
    .title-slide {
      background: var(--dark);
      justify-content: center;
      align-items: center;
      text-align: center;
    }
    .title-slide .slide-title {
      color: #fff;
      font-size: clamp(2.8rem, 4vw, 4.5rem);
    }
    .title-slide .slide-subtitle {
      color: rgba(255,255,255,0.55);
      font-size: clamp(1rem, 1.4vw, 1.3rem);
    }
    .title-slide .title-bar {
      width: 80px;
      height: 4px;
      background: linear-gradient(90deg, var(--teal), var(--navy));
      margin-bottom: 2rem;
    }

    /* === CARD GRID === */
    .card-grid {
      display: grid;
      gap: 2vw;
      flex: 1;
      align-content: center;
    }
    .card-grid.cols-2 { grid-template-columns: repeat(2, 1fr); }
    .card-grid.cols-3 { grid-template-columns: repeat(3, 1fr); }

    .card {
      background: #f7fafc;
      border-radius: 14px;
      padding: 2.8rem 2.5rem;
      display: flex;
      flex-direction: column;
      justify-content: center;
      max-height: 65vh;
    }
    .card-label {
      font-size: 0.85rem;
      text-transform: uppercase;
      letter-spacing: 0.14em;
      color: var(--teal);
      margin-bottom: 0.8rem;
      font-weight: 600;
    }
    .card-heading {
      font-size: clamp(1.25rem, 1.5vw, 1.35rem);
      font-weight: 700;
      color: var(--navy);
      margin-bottom: 0.6rem;
    }
    .card-text {
      font-size: clamp(1rem, 1.15vw, 1.1rem);
      line-height: 1.65;
      color: var(--grey);
    }
    .card-text li {
      margin-bottom: 0.4rem;
      list-style: none;
      padding-left: 1.2rem;
      position: relative;
    }
    .card-text li::before {
      content: '';
      position: absolute;
      left: 0;
      top: 0.55em;
      width: 6px;
      height: 6px;
      border-radius: 50%;
      background: var(--teal);
    }
    .card-icon {
      width: 56px;
      height: 56px;
      object-fit: contain;
      margin-bottom: 1.2rem;
    }

    /* Card variants */
    .card.accent-blue { background: var(--navy); color: #fff; }
    .card.accent-blue .card-heading { color: #fff; }
    .card.accent-blue .card-text { color: rgba(255,255,255,0.85); }
    .card.accent-blue .card-label { color: var(--teal); }

    .card.bordered {
      background: #fff;
      border: 1px solid #e2e8f0;
    }

    /* === COMPARE LAYOUT === */
    .compare-grid {
      display: grid;
      grid-template-columns: 1fr 2px 1fr;
      gap: 2vw;
      flex: 1;
      align-content: center;
    }
    .compare-side {
      padding: 3rem 2.8rem;
      background: var(--light-grey, #f7fafc);
      border-radius: 14px;
    }
    .compare-divider {
      width: 2px;
      height: 60%;
      background: linear-gradient(180deg, transparent, var(--teal), transparent);
      align-self: center;
    }
    .compare-label {
      font-size: 1rem;
      text-transform: uppercase;
      letter-spacing: 0.14em;
      font-weight: 600;
      margin-bottom: 1.4rem;
    }
    .compare-label.old { color: #e53e3e; }
    .compare-label.new { color: var(--teal); }
    .compare-title {
      font-size: clamp(1.8rem, 2.2vw, 2.1rem);
      font-weight: 700;
      color: var(--navy);
      margin-bottom: 1.2rem;
    }
    .compare-text {
      font-size: clamp(1.3rem, 1.5vw, 1.45rem);
      line-height: 1.7;
      color: var(--grey);
    }

    /* === ARCHITECTURE FLOW === */
    .arch-flow {
      display: flex;
      align-items: stretch;
      gap: 4px;
      margin-block: auto;
      min-height: 40vh;
      max-height: 55vh;
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
    .arch-num {
      font-size: 3.5rem;
      font-weight: 800;
      opacity: 0.25;
    }
    .arch-label {
      font-size: clamp(1.3rem, 1.6vw, 1.5rem);
      text-transform: uppercase;
      letter-spacing: 0.08em;
      font-weight: 700;
    }
    .arch-desc {
      font-size: clamp(1.15rem, 1.3vw, 1.25rem);
      opacity: 0.8;
      margin-top: 0.5rem;
    }

    /* === STEPS === */
    .steps {
      max-width: 750px;
      display: flex;
      flex-direction: column;
      gap: 1.6rem;
      margin-block: auto;
    }
    .step {
      display: flex;
      align-items: flex-start;
      gap: 1.2rem;
    }
    .step-num {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: var(--navy);
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: 700;
      flex-shrink: 0;
    }
    .step-title {
      font-weight: 700;
      font-size: 1.1rem;
      color: var(--navy);
    }
    .step-desc {
      color: var(--grey);
      font-size: 0.95rem;
    }

    /* === AGENDA === */
    .agenda {
      display: grid;
      gap: 1.5rem;
      margin-block: auto;
    }
    .agenda-item {
      display: flex;
      gap: 1rem;
      align-items: flex-start;
      padding-left: 1rem;
      border-left: 3px solid var(--navy);
    }
    .agenda-time {
      font-weight: 700;
      color: var(--navy);
      font-size: 0.8rem;
      white-space: nowrap;
    }
    .agenda-title {
      font-weight: 700;
      font-size: 1.05rem;
    }
    .agenda-desc {
      color: var(--grey);
      font-size: 0.9rem;
    }

    /* === TWO COLUMN === */
    .two-col {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 4vw;
      flex: 1;
      align-content: start;
    }
    .col-header {
      font-weight: 700;
      color: var(--navy);
      border-bottom: 2px solid var(--navy);
      padding-bottom: 0.5rem;
      margin-bottom: 1rem;
    }

    /* === NAVIGATION === */
    #progress {
      position: fixed;
      top: 0;
      left: 0;
      height: 3px;
      background: linear-gradient(90deg, var(--navy), var(--teal));
      z-index: 100;
      transition: width 0.3s ease;
    }

    #nav-dots {
      position: fixed;
      right: 1.5vw;
      top: 50%;
      transform: translateY(-50%);
      display: flex;
      flex-direction: column;
      gap: 8px;
      z-index: 100;
    }
    .nav-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: var(--light-grey);
      cursor: pointer;
      transition: transform 0.2s, background 0.2s;
    }
    .nav-dot.active {
      background: var(--navy);
      transform: scale(1.5);
    }

    /* === FOOTER === */
    .slide-num {
      position: absolute;
      bottom: 1.5vh;
      left: 6vw;
      font-size: 0.75rem;
      color: var(--light-grey);
    }
    .slide-footer {
      position: absolute;
      bottom: 1.5vh;
      right: 6vw;
      font-size: 0.75rem;
      color: var(--light-grey);
      text-transform: uppercase;
      letter-spacing: 0.1em;
    }

    /* === REVEAL ANIMATIONS === */
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
    .reveal.d4 { transition-delay: 0.32s; }
    .reveal.d5 { transition-delay: 0.40s; }

    /* === PRINT / PDF === */
    @media print {
      #progress, #nav-dots { display: none !important; }
      .slide { break-after: page; break-inside: avoid; }
      .reveal { opacity: 1 !important; transform: none !important; }
    }
  </style>
</head>
<body>

  <!-- Progress bar -->
  <div id="progress" style="width: 0%"></div>

  <!-- Nav dots (generated by JS) -->
  <nav id="nav-dots"></nav>

  <!-- === SLIDE 1: TITLE === -->
  <section class="slide title-slide">
    <div class="title-bar"></div>
    <h1 class="slide-title">Presentation Title</h1>
    <p class="slide-subtitle">Author — Date</p>
  </section>

  <!-- === SLIDE 2: CONTENT === -->
  <section class="slide">
    <h2 class="slide-title">Slide title</h2>
    <p class="slide-subtitle">Supporting context</p>
    <div class="card-grid cols-3">
      <div class="card reveal d1">
        <div class="card-label">CATEGORY</div>
        <div class="card-heading">Card heading</div>
        <p class="card-text">Card body text goes here.</p>
      </div>
      <!-- More cards... -->
    </div>
    <span class="slide-num">02</span>
    <span class="slide-footer">BRAND</span>
  </section>

  <!-- Add more slides following the same pattern -->

  <script>
    // === SCROLL PROGRESS ===
    const progress = document.getElementById('progress');
    const slides = document.querySelectorAll('.slide');
    const navDots = document.getElementById('nav-dots');

    // Generate nav dots
    slides.forEach((_, i) => {
      const dot = document.createElement('div');
      dot.className = 'nav-dot' + (i === 0 ? ' active' : '');
      dot.onclick = () => slides[i].scrollIntoView({ behavior: 'smooth' });
      navDots.appendChild(dot);
    });

    // Update progress and active dot on scroll
    const updateProgress = () => {
      const scrollTop = window.scrollY;
      const scrollHeight = document.documentElement.scrollHeight - window.innerHeight;
      const pct = scrollHeight > 0 ? (scrollTop / scrollHeight) * 100 : 0;
      progress.style.width = pct + '%';

      const dots = document.querySelectorAll('.nav-dot');
      const currentSlide = Math.round(scrollTop / window.innerHeight);
      dots.forEach((d, i) => d.classList.toggle('active', i === currentSlide));
    };
    window.addEventListener('scroll', updateProgress, { passive: true });

    // === REVEAL ANIMATIONS ===
    const observer = new IntersectionObserver(
      entries => entries.forEach(e => {
        e.target.classList.toggle('visible', e.isIntersecting);
      }),
      { threshold: 0.5 }
    );
    document.querySelectorAll('.reveal').forEach(el => observer.observe(el));

    // === KEYBOARD NAVIGATION ===
    document.addEventListener('keydown', e => {
      const current = Math.round(window.scrollY / window.innerHeight);
      if (e.key === 'ArrowDown' || e.key === ' ' || e.key === 'ArrowRight') {
        e.preventDefault();
        if (current < slides.length - 1) slides[current + 1].scrollIntoView({ behavior: 'smooth' });
      } else if (e.key === 'ArrowUp' || e.key === 'ArrowLeft') {
        e.preventDefault();
        if (current > 0) slides[current - 1].scrollIntoView({ behavior: 'smooth' });
      }
    });
  </script>
</body>
</html>
```

## Customization checklist

When starting a new deck from this template:

1. **Replace `:root` variables** with brand colors from template extraction
2. **Replace font-family** with the template's theme font as the **primary** family (extracted from `fontScheme` major/minor fonts). The template font must come first in the stack, not as a fallback.
3. **Update accent bar gradient** colors in `.slide::after`
4. **Update nav dot active color** to match brand
5. **Set title slide background** to brand's darkest color
6. **Adjust `--section-gap`** if content density requires tighter or looser spacing

## Key structural rules

- Every `<section class="slide">` is one presentation slide
- Cards, steps, agenda items get `class="reveal d1"`, `d2`, `d3` for staggered entry
- Slide numbers are manual (`<span class="slide-num">02</span>`) — update when reordering
- The `@media print` block ensures clean PDF export
- All images must be base64-encoded inline — no external URLs
