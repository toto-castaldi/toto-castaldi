# Phase 5: Bento Grid Layout - Research

**Researched:** 2026-02-19
**Domain:** CSS Grid responsive layout, typography scale, card styling
**Confidence:** HIGH

## Summary

Phase 5 transforms the current single-column project card list into a responsive CSS Grid that shows 3 columns on desktop (>1024px), 2 on tablet (481-1024px), and 1 on mobile (<=480px). The existing `.project-cards` container already uses `display: grid` but has no column definition, so cards stack vertically. The implementation requires adding `grid-template-columns` with media queries, ensuring the card `border-radius` token is consistent, and verifying the modular type scale works across breakpoints.

The critical constraint is preserving the checkbox-hack dark mode toggle. The toggle uses `#dark-toggle:checked + .page-wrapper` (adjacent sibling selector) to control theme tokens. This selector targets `.page-wrapper` and its descendants. Since the grid lives inside `.container` which is inside `.page-wrapper`, the grid will inherit theme tokens correctly. No structural changes to the HTML that would break the sibling relationship are needed -- the grid work is purely additive CSS on the existing `.project-cards` element.

**Primary recommendation:** Use explicit media queries with `grid-template-columns: repeat(N, 1fr)` at the three defined breakpoints. Do NOT use `auto-fit`/`minmax()` -- with exactly 3 items, `auto-fit` causes unpredictable column counts and stretching. The breakpoints are precisely defined in requirements, so explicit media queries are the correct, deterministic approach.

<phase_requirements>

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| LAYOUT-01 | Landing page uses CSS Grid with 3 columns on desktop (>1024px) | `.project-cards { grid-template-columns: repeat(3, 1fr) }` at `min-width: 1025px`. See Architecture Patterns and Code Examples. |
| LAYOUT-02 | Grid adapts to 2 columns on tablet (481-1024px) | `grid-template-columns: repeat(2, 1fr)` at `min-width: 481px`. See Architecture Patterns. |
| LAYOUT-03 | Grid adapts to 1 column on mobile (<=480px) | Default (mobile-first) is single column: `grid-template-columns: 1fr`. See Architecture Patterns. |
| STYLE-01 | Cards with rounded corners (border-radius) | Add `--radius-card` token (0.75rem). Already partially present at 0.5rem on `.project-card`. See Code Examples. |
| STYLE-04 | Consistent typography with modular scale | Already implemented -- fluid clamp() scale exists in `:root`. Verify card typography uses scale tokens (`--font-size-md`, `--font-size-sm`, `--font-size-base`). See Typography Analysis. |

</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| CSS Grid | CSS3 (Level 1) | Responsive multi-column layout | Native browser feature, 97%+ support, already using `display: grid` |
| CSS Custom Properties | CSS3 | Design tokens for border-radius, spacing | Already in use, enables theming consistency |
| CSS Media Queries | CSS3 | Breakpoint-based column switching | Deterministic control for exactly 3 defined breakpoints |

### Supporting
No additional libraries needed. Plain CSS handles all requirements.

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Explicit media queries | `auto-fit` + `minmax()` | With exactly 3 items, `auto-fit` causes stretching/unpredictable column counts. Explicit queries match the exact breakpoint requirements. |
| Explicit media queries | `auto-fill` + `max()` | Clever but over-engineered for 3 items and 3 defined breakpoints. Adds cognitive load for no benefit. |
| Container Queries | Media Queries | Decided against in REQUIREMENTS.md -- single-page, media queries suffice. Container Queries add complexity without benefit here. |
| CSS Subgrid | Flat grid | Decided against in REQUIREMENTS.md -- flat layout, not needed. |

**Installation:** None needed -- plain CSS.

## Architecture Patterns

### Current Structure
```
body
  input#dark-toggle (checkbox, sr-only)
  div.page-wrapper (full-width, background, min-height: 100vh)
    div.container (max-width: 1080px, margin: 0 auto, padding)
      header.site-header
      main
        section (title + tagline)
        section.project-cards (display: grid, gap -- NO columns defined)
          article.project-card (border, border-radius: 0.5rem, padding)
          article.project-card
          article.project-card
```

**Current state:** `.project-cards` already has `display: grid` and `gap: var(--space-lg)` but no `grid-template-columns`, so cards stack in a single implicit column.

### Target Structure
```
body
  input#dark-toggle (checkbox, sr-only)           ← UNCHANGED
  div.page-wrapper                                 ← UNCHANGED
    div.container                                  ← UNCHANGED
      header.site-header                           ← UNCHANGED
      main
        section (title + tagline)                  ← UNCHANGED
        section.project-cards (grid: 3col/2col/1col responsive)  ← ADD columns
          article.project-card (border-radius: var(--radius-card))  ← TOKEN update
          article.project-card
          article.project-card
```

**Key insight:** No HTML template changes are needed. The `layouts/index.html` template already generates the correct structure. All changes are CSS-only.

### Pattern 1: Mobile-First Media Queries for Grid Columns

**What:** Define grid as single-column by default, then add columns at wider breakpoints.
**When to use:** When breakpoints are precisely defined in requirements (as they are here).

```css
/* Mobile-first: single column (<=480px) */
.project-cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-lg);
}

/* Tablet: 2 columns (481px-1024px) */
@media (min-width: 481px) {
  .project-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop: 3 columns (>1024px) */
@media (min-width: 1025px) {
  .project-cards {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

**Why `min-width: 1025px` not `1024px`:** The requirement says ">1024px" for desktop. At exactly 1024px, we want 2 columns (tablet range is 481-1024px). So the desktop breakpoint triggers at 1025px.

### Pattern 2: Border-Radius Token

**What:** Use a CSS custom property for card border-radius to ensure consistency.
**When to use:** When cards need rounded corners that should be uniform.

```css
:root {
  --radius-card: 0.75rem;  /* 12px at 16px base */
}

.project-card {
  border-radius: var(--radius-card);
}
```

**Why 0.75rem (12px):** Current value is 0.5rem (8px). The bento grid trend uses larger radii (12-24px). 12px is the conservative end -- visible but not exaggerated. The project is minimal/clean, so 12px fits better than 16-24px.

### Anti-Patterns to Avoid

- **Using `auto-fit`/`auto-fill` with exactly 3 items:** With 3 items, `auto-fit` collapses empty tracks and stretches items unpredictably. On wide screens (>1440px), items might stretch excessively. On medium screens, column count is non-deterministic. Use explicit `repeat(N, 1fr)` instead.
- **Placing CSS Grid on `.container` instead of `.project-cards`:** The container holds header + main. Making it a grid would affect the header layout. The grid must be scoped to the card section.
- **Adding wrapper elements between `#dark-toggle` and `.page-wrapper`:** This would break the adjacent sibling selector `#dark-toggle:checked + .page-wrapper`. All structural changes must preserve this relationship.
- **Using `max-width` media queries instead of `min-width`:** Mobile-first (`min-width`) is the standard pattern. The current CSS has no media query for layout (only for `prefers-color-scheme`), so there is no conflict.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Responsive columns | Flexbox with `calc()` width hacks | CSS Grid `repeat(N, 1fr)` | Grid handles equal columns natively; flexbox requires manual width calculations and wrap behavior tuning |
| Card equal heights | JavaScript height equalization | CSS Grid implicit equal rows | Grid rows automatically equalize card heights within the same row -- no JS needed |
| Breakpoint management | Custom CSS framework | Standard `@media (min-width)` | Three breakpoints don't warrant a framework. Plain media queries are clear and maintainable at this scale. |

**Key insight:** CSS Grid inherently equalizes card heights within a row and distributes columns evenly. The only thing needed is telling it how many columns at each breakpoint.

## Common Pitfalls

### Pitfall 1: Grid Gap Causing Horizontal Overflow on Mobile
**What goes wrong:** On narrow screens (320px), `gap: var(--space-lg)` (2.5rem = 40px) combined with container padding eats too much space, potentially causing overflow.
**Why it happens:** Gap is fixed at 2.5rem. On a 320px screen with 1.5rem (24px) padding on each side, content area is 272px. Single column with 40px gap between cards is fine (gap only applies between items), but if gap were applied as margin it could overflow.
**How to avoid:** Single-column layout is safe because gap only applies between rows, not at edges. The container padding (1.5rem) handles edge spacing. No issue expected, but verify visually on 320px.
**Warning signs:** Horizontal scrollbar on narrow phones.

### Pitfall 2: Cards with Unequal Content Heights Creating Jagged Rows
**What goes wrong:** On 2-column layout, if one card has a longer description than its row partner, the shorter card stretches to match (Grid behavior), leaving visual whitespace.
**Why it happens:** CSS Grid rows stretch to the tallest item. Cards with unequal content create uneven whitespace.
**How to avoid:** The 3 project cards have similar content length (name, 1-line description, link). This is not a problem for this project. If future cards have very different content lengths, consider `align-items: start` on the grid to prevent stretching.
**Warning signs:** Cards with large empty areas at the bottom.

### Pitfall 3: Card Border-Radius Clipping Card Content
**What goes wrong:** Increasing `border-radius` on cards can clip child elements that touch the card edges.
**Why it happens:** `border-radius` clips the card's bounding box. Absolutely positioned children or content with no padding can get clipped at corners.
**How to avoid:** The cards already have `padding: var(--space-lg)` (2.5rem), which keeps content well away from edges. The `.card-link::after` overlay uses `position: absolute; inset: 0` -- this will be clipped by border-radius, which is actually desirable (keeps click target within the rounded shape). No issue expected at 0.75rem radius.
**Warning signs:** Cut-off text or images in card corners.

### Pitfall 4: Dark Mode Toggle Breakage After Template Changes
**What goes wrong:** Adding HTML wrapper elements or moving the checkbox input breaks the `#dark-toggle:checked + .page-wrapper` selector.
**Why it happens:** The `+` selector requires direct adjacency. Any element inserted between `#dark-toggle` and `.page-wrapper` breaks it.
**How to avoid:** This phase requires NO HTML template changes. All changes are CSS-only. The grid is applied to the existing `.project-cards` element. Verify the toggle still works in all 4 states after any CSS change.
**Warning signs:** Theme toggle button clicks but nothing changes; theme stuck in one mode.

### Pitfall 5: Typography Inconsistency Across Breakpoints
**What goes wrong:** Font sizes look correct on desktop but become too large or too small on mobile, breaking visual hierarchy.
**Why it happens:** Using fixed `rem` or `px` values that don't adapt to viewport.
**How to avoid:** The existing fluid type scale using `clamp()` already handles this. The tokens (`--font-size-sm`, `--font-size-base`, `--font-size-md`, `--font-size-lg`, `--font-size-xl`) are viewport-responsive. No changes needed -- just verify the card typography uses these tokens consistently.
**Warning signs:** Headings that feel oversized on mobile or undersized on desktop.

## Code Examples

Verified patterns for the implementation:

### Complete Grid Responsive CSS (Mobile-First)
```css
/* Source: CSS Grid Level 1 spec + project requirements */

/* Mobile-first: single column */
.project-cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-lg);
  padding: var(--space-md) 0;
}

/* Tablet: 2 columns */
@media (min-width: 481px) {
  .project-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop: 3 columns */
@media (min-width: 1025px) {
  .project-cards {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Border-Radius Token Addition
```css
/* Source: Design system pattern */
:root {
  /* Add to existing token block */
  --radius-card: 0.75rem;  /* 12px -- bento card rounding */
}

.project-card {
  /* Update existing border-radius from hardcoded 0.5rem */
  border-radius: var(--radius-card);
}
```

### Typography Verification (Already Implemented)
```css
/* These tokens already exist in :root and are used by card elements */
/* No changes needed -- documenting for verification */

:root {
  --font-size-sm: clamp(0.83rem, 0.44vw + 0.72rem, 1rem);
  --font-size-base: clamp(1rem, 0.34vw + 0.91rem, 1.2rem);
  --font-size-md: clamp(1.2rem, 0.41vw + 1.09rem, 1.44rem);
  --font-size-lg: clamp(1.44rem, 0.49vw + 1.31rem, 1.73rem);
  --font-size-xl: clamp(1.73rem, 0.59vw + 1.57rem, 2.07rem);
}

/* Card typography already uses tokens */
.project-card h2 { font-size: var(--font-size-md); }
.project-card p { font-size: var(--font-size-sm); }
.card-link { font-size: var(--font-size-sm); }
```

### Current Card CSS (For Reference)
```css
/* Existing card styles that will be preserved */
.project-card {
  position: relative;
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;           /* → will become var(--radius-card) */
  padding: var(--space-lg);
  background: var(--color-card-bg);
  transition: border-color 0.15s ease;
}
```

## Typography Analysis

The existing type scale follows a ~1.2 (minor third) modular ratio:

| Token | Min (mobile) | Max (desktop) | Ratio Step | Used By |
|-------|-------------|---------------|------------|---------|
| `--font-size-sm` | 0.83rem (13.3px) | 1rem (16px) | -1 | Card description, card link, muted text |
| `--font-size-base` | 1rem (16px) | 1.2rem (19.2px) | 0 (base) | Body text, card paragraphs |
| `--font-size-md` | 1.2rem (19.2px) | 1.44rem (23px) | +1 | Card headings (h2), toggle button |
| `--font-size-lg` | 1.44rem (23px) | 1.73rem (27.7px) | +2 | Section headings (unused currently) |
| `--font-size-xl` | 1.73rem (27.7px) | 2.07rem (33.1px) | +3 | Page title (h1) |

**Assessment:** The scale is well-implemented with fluid `clamp()` values. Each step multiplies by ~1.2. Card typography (`h2` at `--font-size-md`, `p` at `--font-size-sm`, link at `--font-size-sm`) uses the correct scale steps. STYLE-04 is already satisfied -- verification only needed, no implementation.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Flexbox card grids with `calc()` widths | CSS Grid `repeat(N, 1fr)` | 2017+ (Grid Level 1) | Equal columns without manual width math |
| `auto-fit`/`auto-fill` for all grids | Explicit columns when item count is known | Best practice clarification ~2023 | Deterministic layouts; `auto-fit` only for unknown item counts |
| Fixed `px` breakpoints | `rem`-based or content-based breakpoints | Ongoing trend | Better adaptation to user preferences; this project uses `px` per requirements |
| Media query range syntax `@media (width >= 481px)` | Replaces `@media (min-width: 481px)` | CSS Media Queries Level 4 (~2023) | Cleaner syntax, ~96% browser support |

**Deprecated/outdated:**
- Float-based grid systems: completely superseded by CSS Grid
- `auto-fit`/`minmax()` as universal responsive solution: only appropriate when item count varies; explicit columns preferred for fixed item counts

**Note on range syntax:** The modern `@media (width >= 481px)` syntax has ~96% browser support (caniuse). However, the traditional `@media (min-width: 481px)` works everywhere and is what the codebase currently uses for `prefers-color-scheme`. For consistency, stick with `min-width` syntax.

## Open Questions

1. **Gap size on tablet (2-column) vs desktop (3-column)**
   - What we know: Current gap is `var(--space-lg)` (2.5rem). On 3-column desktop at 1080px max-width, each card is roughly (1080 - 2*40) / 3 = 333px. With 2.5rem gap, cards are ~300px each. This is comfortable.
   - What's unclear: Whether the gap should reduce on tablet to give cards more breathing room in 2-column layout.
   - Recommendation: Keep uniform gap (`--space-lg`) at all breakpoints. The 2-column layout on a 768px screen gives ~340px per card (768 - 24*2 padding - 40 gap) / 2 = ~340px. Adequate. Reduce complexity by not varying gap. If cards feel cramped on smaller tablets (481px), the gap can be tuned in Phase 6.

2. **Whether to reduce container padding on mobile**
   - What we know: Container padding is `var(--space-xl) var(--space-md)` = 4rem vertical, 1.5rem horizontal. On 320px screen, 1.5rem*2 = 3rem = 48px leaves 272px for content.
   - What's unclear: Whether 272px is sufficient for card content on the narrowest phones.
   - Recommendation: 272px is fine for the card content (name, 1-line description, link). No padding change needed. This can be adjusted in Phase 6 if visual testing reveals issues.

## Sources

### Primary (HIGH confidence)
- MDN Web Docs: `repeat()` CSS function - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/repeat - Grid syntax, `auto-fit` vs `auto-fill` behavior, browser compatibility (Baseline: widely available since July 2020)
- Current codebase: `assets/css/main.css` (304 lines), `layouts/index.html`, `layouts/_default/baseof.html`, `data/projects.toml`
- Phase 4 VERIFICATION.md and SUMMARY.md: confirmed WCAG tokens, container structure, toggle mechanism all in place

### Secondary (MEDIUM confidence)
- CSS-Tricks: "An Auto-Filling CSS Grid With Max Columns" - https://css-tricks.com/an-auto-filling-css-grid-with-max-columns/ - `auto-fill` + `max()` technique for limiting columns; confirmed that explicit queries are simpler for fixed item counts
- WeAreDevelopers: "Building a Bento Grid Layout with Modern CSS Grid" - https://www.wearedevelopers.com/en/magazine/682/building-a-bento-grid-layout-with-modern-css-grid-682 - Bento patterns: 12-column base, span classes, dense packing
- iamsteve.me: "Build a bento layout with CSS grid" - https://iamsteve.me/blog/bento-layout-css-grid - Grid setup, responsive patterns, card styling
- Smashing Magazine: "Modern Fluid Typography Using CSS Clamp" - https://www.smashingmagazine.com/2022/01/modern-fluid-typography-css-clamp/ - clamp() fluid type scale best practices, WCAG 1.4.4 compliance

### Tertiary (LOW confidence)
- None -- all findings verified with primary or secondary sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- CSS Grid is mature (since 2017), `repeat(N, 1fr)` is the most basic grid pattern
- Architecture: HIGH -- no HTML changes needed, CSS-only additive changes to existing `.project-cards` element
- Pitfalls: HIGH -- all pitfalls are well-documented CSS Grid behaviors; the main risk (toggle breakage) is avoided by not touching HTML
- Typography: HIGH -- existing fluid scale is already implemented and verified; STYLE-04 is verification-only

**Research date:** 2026-02-19
**Valid until:** 2026-08-19 (stable domain -- CSS Grid spec is mature, breakpoints are project-specific)
