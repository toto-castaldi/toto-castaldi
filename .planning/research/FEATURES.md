# Feature Landscape: v1.1 Bento Grid Redesign

**Domain:** Developer portfolio bento grid layout with responsive redesign and theme fix
**Researched:** 2026-02-19
**Overall confidence:** HIGH

> This document covers NEW features for milestone v1.1 only. Existing v1.0 features (project cards, language switcher, dark mode toggle, Hugo i18n, SEO tags) are already shipped and not re-evaluated here.

## Table Stakes

Features the bento grid redesign must have. Without these, the redesign fails its purpose.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| 3-column grid on desktop | The entire point of v1.1. Current single-column layout wastes horizontal space and looks broken on desktop. Three projects = three columns is the natural mapping. | LOW | CSS Grid `grid-template-columns: repeat(3, 1fr)` on the existing `.project-cards` container. No HTML changes needed -- the Hugo `range` already outputs 3 `<article>` elements. |
| Responsive collapse: 3 cols -> 1 col | Mobile screens (< 768px) cannot fit 3 columns. Single column stacking is the universal mobile pattern. | LOW | Mobile-first: default is `grid-template-columns: 1fr`, then `@media (min-width: 768px)` applies the 3-column layout. Current CSS already has a grid container; just needs column definition. |
| Consistent gap spacing | Bento grids require uniform gutters (12-24px) between all cards. Inconsistent gaps break the "intentional layout" feel that defines bento design. | LOW | Already using `gap: var(--space-lg)` (2.5rem = 40px). Tighten to `gap: var(--space-md)` (1.5rem = 24px) for a more compact bento feel. |
| Light mode passes WCAG AA contrast | Current light mode was flagged as failing contrast. Analysis shows muted text (#6b7280 on #ffffff) is 4.83:1 -- technically passes AA but barely. The real issue is likely border contrast (1.24:1) and overall "washed out" feeling. Must fix to feel legible. | LOW | Darken `--color-text-muted` to #4b5563 (7.56:1 ratio). Darken `--color-border` to #d1d5db or add card shadows instead of relying on borders for visual separation. |
| Dark mode passes WCAG AA contrast | Dark mode is already passing (muted on card: 5.78:1) but should be verified after any color changes to maintain parity. | LOW | Current dark palette is solid. Verify after light mode fixes. No changes expected. |
| Cards visually distinct from background | In bento design, each tile must read as a separate container. Cards need clear visual boundaries in both themes. | LOW | Combine border + subtle shadow + background differentiation. Current `border: 1px solid var(--color-border)` alone is insufficient in light mode (1.24:1 border contrast). |

## Differentiators

Features that elevate the redesign from "functional grid" to "polished bento portfolio." Not strictly required, but high impact for low effort.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Rounded corners on cards (12-16px) | Signature bento visual. Apple-style rounded rectangles signal modern design. Current `border-radius: 0.5rem` (8px) is too subtle for bento. | LOW | Increase `border-radius` to `0.75rem` (12px) or `1rem` (16px). The 2026 bento trend leans toward 12-24px radii. |
| Card hover elevation effect | Subtle `box-shadow` increase + slight `transform: translateY(-2px)` on hover creates depth. Makes cards feel interactive and tactile -- a hallmark of dashboard-style bento. | LOW | Pure CSS: `transition: box-shadow 0.2s, transform 0.2s` with hover state. Already have `transition: border-color 0.15s` on cards. Extend it. No JS needed. |
| Layered box-shadow for depth | Single flat shadows look dated. Layering 2-3 shadows at different spreads creates realistic depth perception. Works beautifully in both light and dark modes with different shadow colors. | LOW | Light: `box-shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04)`. Dark: `box-shadow: 0 1px 3px rgba(0,0,0,0.3), 0 4px 12px rgba(0,0,0,0.2)`. Define via CSS variables. |
| Wider page container | Current `--max-width: 680px` is perfect for single-column reading but too narrow for a 3-column bento grid. Cards would be ~200px wide -- cramped. | LOW | Increase `--max-width` to `960px` or `1080px` for the bento layout. This is the single most impactful change for the "3 cards in a row" goal. |
| Card background subtle differentiation | In bento layouts, cards often use a slightly different background from the page to create visual layers. Current light mode has card-bg = page-bg = #ffffff (no differentiation). | LOW | Light: `--color-card-bg: #f8fafc` (very light gray) on `--color-bg: #ffffff`. Dark: already differentiated (#1f2937 cards on #111827 bg). |
| 2-column tablet breakpoint | Between mobile (1 col) and desktop (3 col), tablets (768px-1024px) benefit from a 2-column layout where the third card spans full width below, or a centered 2+1 arrangement. | LOW | `@media (min-width: 768px) { grid-template-columns: repeat(2, 1fr); }` then `@media (min-width: 1024px) { grid-template-columns: repeat(3, 1fr); }`. Clean progressive enhancement. |
| Card accent/highlight per project | Each project card could have a subtle top-border color or left-accent strip, making cards visually distinguishable beyond just text. Adds personality without complexity. | LOW | `border-top: 3px solid var(--accent-docora)` etc. Define 3 accent colors as CSS variables. Works in both themes if colors are chosen carefully. |
| `prefers-reduced-motion` respect | Disable hover transitions and any animations for users who have motion sensitivity. Small effort, significant accessibility signal for a developer portfolio. | LOW | `@media (prefers-reduced-motion: reduce) { * { transition: none !important; } }`. Already should exist but is missing from current CSS. |
| Smooth theme-aware border glow on hover | Instead of just border-color change on hover, a subtle glow effect (`box-shadow: 0 0 0 1px var(--color-link)`) around the card creates a modern focus/hover indicator that works in both themes. | LOW | Replace current `border-color` hover with a combination shadow: `box-shadow: 0 0 0 1px var(--color-link), 0 4px 12px rgba(37,99,235,0.1)`. |

## Anti-Features

Features that seem natural for a bento redesign but should be explicitly avoided.

| Anti-Feature | Why Tempting | Why Avoid | What to Do Instead |
|--------------|-------------|-----------|-------------------|
| Variable-size bento tiles (2x1, 1x2, 2x2 spans) | "True" bento grids use mixed-size tiles for visual hierarchy. Apple keynotes showcase this heavily. | With exactly 3 project cards of equal importance, variable sizing creates false hierarchy. One card appearing bigger implies it matters more. All three projects are equal peers. | Equal-width 3-column grid. If extra bento sections are added later (tech stack, GitHub stats), THEN variable sizing makes sense for those secondary tiles. |
| Glassmorphism / backdrop-filter blur | Trendy in 2024-2025 bento designs. Frosted glass cards look impressive. | Requires semi-transparent backgrounds, which complicates contrast calculations for WCAG AA. Also needs a decorative background (gradient, image) behind cards to show the blur effect -- adds complexity for no content value. Performance cost of `backdrop-filter` on older devices. | Solid card backgrounds with layered shadows. Cleaner, more accessible, faster, and ages better than trends. |
| JavaScript-powered card animations | Scroll-triggered reveals, staggered fade-ins, GSAP animations. Many bento portfolio templates include these. | Violates the "zero JavaScript" constraint. Adds runtime dependencies, increases bundle size, creates accessibility issues (motion sensitivity). The site has 3 cards -- animation is gratuitous, not informative. | CSS-only hover transitions (transform, shadow). Respect `prefers-reduced-motion`. |
| Grid auto-placement with `grid-auto-flow: dense` | Useful for masonry-like bento layouts where items reorder to fill gaps. | With exactly 3 equal cards, there are no gaps to fill. Dense packing reorders content, which can confuse screen readers (DOM order vs visual order mismatch). | Explicit `grid-template-columns: repeat(3, 1fr)` -- predictable, accessible, maintainable. |
| Dark mode toggle with JavaScript + localStorage | Many sources recommend JS for theme persistence across page loads. The CSS-only checkbox hack loses state on navigation. | Already shipped with CSS-only checkbox toggle (v1.0 decision). Adding JS contradicts the zero-JS constraint. For a single-page landing, state persistence is unnecessary -- the user only visits one page. | Keep the existing CSS-only checkbox hack. It works for this use case. If the site ever grows to multiple pages, reconsider. |
| Icon fonts or SVG icon libraries for card decoration | Card icons/illustrations are common in bento layouts. Libraries like Heroicons or Font Awesome. | External dependency. The projects already have logo SVGs loaded from their subdomains. Adding generic icons alongside logos creates visual clutter. | Use the existing project logos (`{{ .logo }}`). They are already in the template. If logos fail to load, the card still works (text-only fallback). |
| CSS Container Queries for card internal layout | Modern technique where card contents respond to the card's own width rather than viewport. | Browser support is solid (Baseline 2023) but overkill for cards with simple content: logo, h2, p, link. The card internal layout does not change at any size -- content is always stacked vertically. | Standard media queries for the grid container. Container queries add complexity with no benefit here. |
| Extra bento sections in this milestone (tech stack, GitHub links, stats) | PROJECT.md mentions "additional bento sections where design warrants." Tempting to add more tiles. | Scope creep. The milestone goal is "redesign with bento grid layout" for the 3 project cards. Adding new content sections changes the information architecture, requires new translations, and multiplies the work. | Ship the 3-card bento grid first. Evaluate whether extra sections add value after seeing the redesign live. Future milestone if warranted. |

## Feature Dependencies

```
[Wider max-width container]
    |
    +--enables--> [3-column grid layout]
                      |
                      +--enhances--> [2-column tablet breakpoint]
                      +--enhances--> [Card hover effects]
                      +--enhances--> [Card accent colors]

[WCAG AA contrast fix]
    |
    +--requires--> [Updated color tokens (--color-text-muted, --color-border)]
    +--requires--> [Card background differentiation (--color-card-bg)]
    +--blocks----> [Card hover effects] (hover colors must also pass contrast)

[CSS variable-based shadows]
    |
    +--enables--> [Theme-aware shadows (different values per theme)]
    +--enables--> [Hover elevation effect]
    +--enables--> [Border glow on hover]

[prefers-reduced-motion]
    +--independent (add after all transitions are defined)
```

### Dependency Notes

- **Wider container is the gating change.** Nothing else matters if `--max-width` stays at 680px. Three 200px columns are too narrow for readable cards. Increase to 960-1080px first.
- **Contrast fixes must precede visual polish.** No point adding hover effects or shadows if the base colors fail accessibility. Fix tokens first, then layer effects.
- **No HTML changes needed for the grid.** The existing Hugo template outputs 3 `<article>` elements inside a `.project-cards` container. All changes are pure CSS.
- **Zero JavaScript remains the constraint.** Every feature listed is CSS-only. The checkbox dark mode toggle stays as-is.

## Implementation Priority (Recommended Order)

### Phase 1: Layout Foundation (must-do)

1. **Increase `--max-width`** from 680px to 1080px -- enables 3-column grid
2. **Add `grid-template-columns: repeat(3, 1fr)`** to `.project-cards` -- the core change
3. **Add responsive breakpoints** -- mobile (1 col) / tablet (2 col) / desktop (3 col)
4. **Tighten gap** from `--space-lg` to `--space-md` for bento proportions

### Phase 2: Contrast and Legibility (must-do)

5. **Fix `--color-text-muted`** in light mode: #6b7280 -> #4b5563 (4.83:1 -> 7.56:1)
6. **Fix `--color-border`** in light mode: #e5e7eb -> #d1d5db (improve card boundary visibility)
7. **Differentiate `--color-card-bg`** in light mode: #ffffff -> #f8fafc (subtle layering)
8. **Verify dark mode** ratios still pass after any changes

### Phase 3: Visual Polish (should-do)

9. **Increase `border-radius`** to 12-16px for bento aesthetic
10. **Add layered `box-shadow`** with CSS variables (theme-aware)
11. **Add hover elevation** (translateY + shadow increase)
12. **Add `prefers-reduced-motion`** media query
13. **Optional: per-project accent colors** on card top border

### Deferred

- Extra bento sections (tech stack, links) -- future milestone if needed
- Variable-size tiles -- only relevant when more than 3 equal-weight items exist
- Container queries -- not needed for this card structure

## Contrast Audit (Current State)

Computed from the existing CSS custom property values:

| Element | Light Mode | Dark Mode | WCAG AA | Status |
|---------|-----------|-----------|---------|--------|
| Body text on bg | #1a1a1a on #ffffff = 17.40:1 | #f3f4f6 on #111827 = 16.12:1 | 4.5:1 | PASS both |
| Muted text on bg | #6b7280 on #ffffff = 4.83:1 | #9ca3af on #111827 = 6.99:1 | 4.5:1 | PASS both (light barely) |
| Muted text on card | #6b7280 on #ffffff = 4.83:1 | #9ca3af on #1f2937 = 5.78:1 | 4.5:1 | PASS both (light barely) |
| Link on bg | #2563eb on #ffffff = 5.17:1 | #60a5fa on #111827 = 6.98:1 | 4.5:1 | PASS both |
| Border on bg (UI) | #e5e7eb on #ffffff = 1.24:1 | #374151 on #111827 = 1.72:1 | 3.0:1 | **FAIL both** |

**Key finding:** Muted text passes AA but feels weak in light mode. The real failure is **border contrast** -- card boundaries are nearly invisible, especially in light mode. This is likely what "light mode fails contrast" refers to. Fix: stronger borders + card shadows + background differentiation.

## Recommended Color Token Updates

### Light Mode (proposed)

| Token | Current | Proposed | Contrast on #ffffff | Rationale |
|-------|---------|----------|---------------------|-----------|
| `--color-text-muted` | #6b7280 | #4b5563 | 7.56:1 (was 4.83:1) | Comfortable margin above AA. Descriptions clearly readable. |
| `--color-border` | #e5e7eb | #cbd5e1 | ~1.8:1 | Still subtle, but combined with shadow, cards are clearly bounded. |
| `--color-card-bg` | #ffffff | #f8fafc | N/A | Creates layering against page background. Card feels like a surface. |

### Dark Mode (no changes needed)

All dark mode values already pass WCAG AA with comfortable margins. The palette is well-chosen.

## Sources

- [WeAreDevelopers: Building a Bento Grid Layout with Modern CSS Grid](https://www.wearedevelopers.com/en/magazine/682/building-a-bento-grid-layout-with-modern-css-grid-682) -- CSS Grid bento patterns (HIGH confidence)
- [iamsteve: Build a bento layout with CSS grid](https://iamsteve.me/blog/bento-layout-css-grid) -- 12-column grid, card spanning, responsive breakpoints (HIGH confidence)
- [ibelick: Creating Bento Grid Layouts](https://ibelick.com/blog/create-bento-grid-layouts) -- grid-template-columns, auto-rows, spanning techniques (HIGH confidence)
- [SaaSFrame: Designing Bento Grids That Actually Work (2026)](https://www.saasframe.io/blog/designing-bento-grids-that-actually-work-a-2026-practical-guide) -- 12-24px gap guidance, hierarchy principles (MEDIUM confidence)
- [Inkbot Design: Bento Grid Design for Conversion (2026)](https://inkbotdesign.com/bento-grid-design/) -- 6-12 blocks max, hierarchy encoding (MEDIUM confidence)
- [Desinance: Bento Grid Web Design (2026)](https://desinance.com/design/bento-grid-web-design/) -- 12-24px corner radius trend, micro-interactions (MEDIUM confidence)
- [DubBot: Dark Mode Best Practices for Accessibility](https://dubbot.com/dubblog/2023/dark-mode-a11y.html) -- avoid saturated colors, focus indicator visibility (HIGH confidence)
- [AllAccessible: Color Contrast WCAG 2025 Guide](https://www.allaccessible.org/blog/color-contrast-accessibility-wcag-guide-2025) -- 4.5:1 normal text, 3:1 large text/UI (HIGH confidence)
- [W3C: Understanding Contrast (Minimum) WCAG 2.2](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) -- authoritative contrast requirements (HIGH confidence)
- [Josh W. Comeau: Designing Beautiful Shadows in CSS](https://www.joshwcomeau.com/css/designing-shadows/) -- layered shadow technique for depth (HIGH confidence)
- [code.mendhak.com: The unpleasant hackiness of CSS dark mode toggles](https://code.mendhak.com/css-dark-mode-toggle-sucks/) -- CSS-only toggle limitations, localStorage tradeoffs (MEDIUM confidence)
- [Speckyboy: 8 CSS Snippets for Bento Grid Layouts](https://speckyboy.com/css-bento-grid-layouts/) -- practical code snippets (MEDIUM confidence)
- [BentoGrids.com](https://bentogrids.com/) -- curated bento design gallery for reference (MEDIUM confidence)
- Contrast ratios computed locally using WCAG relative luminance formula against current `main.css` color values (HIGH confidence)

---
*Feature research for v1.1 Bento Grid Redesign of toto-castaldi.com*
*Researched: 2026-02-19*
