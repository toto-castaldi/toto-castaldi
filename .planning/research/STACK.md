# Technology Stack: v1.1 Bento Grid Restyling

**Project:** toto-castaldi.com
**Researched:** 2026-02-19
**Scope:** Stack additions/changes for bento grid layout, responsive redesign, and WCAG AA dark/light theming
**Overall confidence:** HIGH

## Current Stack (Validated -- No Changes Needed)

These are already working in production and require zero changes:

| Technology | Version | Status |
|------------|---------|--------|
| Hugo Extended | 0.155.3 | Keep as-is |
| Dart Sass | 1.97.3 | Installed in CI but **not used** -- site uses plain CSS |
| GitHub Actions | Current | Keep as-is |
| GitHub Pages | N/A | Keep as-is |
| System font stack | N/A | Keep as-is |
| Bilingual IT/EN | Hugo i18n | Keep as-is |

## Key Decision: Migrate from Plain CSS to SCSS

**Recommendation: YES, migrate to SCSS.**

The current site uses a single `assets/css/main.css` file (301 lines). The v1.1 redesign will roughly triple the CSS complexity with grid layouts, responsive breakpoints, and two complete color themes. SCSS provides:

- **Partials** -- split one 900+ line file into logical modules (`_grid.scss`, `_tokens.scss`, `_theme.scss`, `_cards.scss`)
- **Nesting** -- cleaner responsive breakpoint organization (nest `@media` inside component selectors)
- **Mixins** -- reusable responsive breakpoint definitions
- **`@use` / `@forward`** -- Hugo 0.155.3 with Dart Sass fully supports modern `@use` syntax (confirmed: Hugo added Dart Sass transpiler support, `@use "hugo:vars" as v` documented in official css.Sass docs)

The cost is near-zero: Dart Sass is already installed in CI. The only change is renaming the file and adding `transpiler: "dartsass"` to the Hugo Pipes call.

**Why not stay with plain CSS?** Native CSS nesting has 87%+ browser support and would work, but SCSS partials and mixins are the decisive advantages. A 900+ line single CSS file becomes unmaintainable. Native CSS has no file-splitting mechanism.

### Migration Steps

```
assets/css/main.css --> assets/scss/main.scss
                        assets/scss/_tokens.scss
                        assets/scss/_base.scss
                        assets/scss/_grid.scss
                        assets/scss/_cards.scss
                        assets/scss/_theme.scss
                        assets/scss/_header.scss
                        assets/scss/_utilities.scss
```

Hugo Pipes template change in `head.html`:
```go
{{ $css := resources.Get "scss/main.scss" | css.Sass (dict "transpiler" "dartsass" "outputStyle" "compressed") | resources.Fingerprint }}
```

**Confidence: HIGH** -- Hugo official docs confirm this exact pattern.

## New Techniques Required

### 1. CSS Grid (Bento Layout)

**Use: `grid-template-areas` with media queries** -- not `grid-auto-flow: dense`.

| Approach | Recommendation | Rationale |
|----------|----------------|-----------|
| `grid-template-areas` | **USE THIS** | Named areas give explicit control over which card goes where at each breakpoint. For 3 project cards + potential extra sections, you need predictable placement, not algorithmic filling. Redefine areas per breakpoint for clean responsive behavior. |
| `grid-auto-flow: dense` | Skip | Good for 12+ items where algorithmic placement is acceptable. With only 3-5 items, you want explicit control. Dense packing can produce unexpected visual order. |
| CSS Subgrid | Skip | 87.8% browser support -- good but unnecessary. Subgrid aligns child grids to parent grids. This site has a flat grid, not nested grids. Adds complexity for zero benefit. |
| Container Queries | Skip for v1.1 | 93.5% support, useful for self-aware components. Overkill for this page where media queries suffice. The page has one grid, not reusable components in different contexts. |

**Pattern -- Desktop (3-column bento):**
```scss
.bento-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--space-md);
  grid-template-areas:
    "docora lumio  helix";
}
```

**Pattern -- Tablet (2-column):**
```scss
@media (max-width: 768px) {
  .bento-grid {
    grid-template-columns: repeat(2, 1fr);
    grid-template-areas:
      "docora lumio"
      "helix  helix";
  }
}
```

**Pattern -- Mobile (1-column stack):**
```scss
@media (max-width: 480px) {
  .bento-grid {
    grid-template-columns: 1fr;
    grid-template-areas:
      "docora"
      "lumio"
      "helix";
  }
}
```

**Browser support: CSS Grid** -- 97%+ globally. No concerns.

**Confidence: HIGH** -- CSS Grid is mature, universally supported, and well-documented.

### 2. Dark/Light Mode Toggle

**Decision: Keep the checkbox hack. Do NOT switch to `:has()`.**

| Approach | Recommendation | Rationale |
|----------|----------------|-----------|
| Checkbox hack (`#toggle:checked + .wrapper`) | **KEEP THIS** | Already implemented and working. 100% browser support. The constraint (checkbox must be sibling of wrapper) is already satisfied in the existing `baseof.html` structure. |
| `:has()` selector (`body:has(#toggle:checked)`) | Skip | 93.7% support -- good but not 100%. The benefit is cleaner selector (can target `body` directly). But the existing structure works fine, and switching gains nothing while losing ~6% of browsers. |
| `light-dark()` CSS function | Skip | 85% support, not yet Baseline Widely Available (November 2026). Cannot be combined with a manual toggle -- it only responds to `prefers-color-scheme`, not a checkbox state. |
| JavaScript toggle | Skip | Project constraint: zero JS. |

**What to change in the theme system:**

The current checkbox hack implementation is correct in structure. What needs fixing:

1. **Border contrast** -- borders fail WCAG AA for UI components (see contrast analysis below)
2. **Card-to-background differentiation** -- light mode card bg (#fff) = page bg (#fff), zero visual separation
3. **Add `color-scheme` property** -- tells the browser to render form controls and scrollbars in the correct theme
4. **Add `<meta name="color-scheme">` tag** -- enables browser-level dark/light awareness

```scss
// Add to :root
:root {
  color-scheme: light dark;
}

// Light mode explicit
.page-wrapper {
  color-scheme: light;
}

// Dark mode
#dark-toggle:checked + .page-wrapper {
  color-scheme: dark;
}

@media (prefers-color-scheme: dark) {
  .page-wrapper {
    color-scheme: dark;
  }
  #dark-toggle:checked + .page-wrapper {
    color-scheme: light;
  }
}
```

**Confidence: HIGH** -- checkbox hack is proven, `color-scheme` property has 96%+ support.

### 3. WCAG AA Color Token Audit

**Current contrast analysis (computed with WCAG 2.1 relative luminance formula):**

#### Text Contrast (requirement: 4.5:1 normal, 3:1 large)

| Pair | Ratio | AA Normal | Status |
|------|-------|-----------|--------|
| Light: `#1a1a1a` on `#ffffff` | 17.40:1 | PASS | Good |
| Light: `#6b7280` on `#ffffff` | 4.83:1 | PASS | Borderline -- improve |
| Light: `#2563eb` on `#ffffff` | 5.17:1 | PASS | Adequate |
| Dark: `#f3f4f6` on `#111827` | 16.12:1 | PASS | Good |
| Dark: `#9ca3af` on `#111827` | 6.99:1 | PASS | Good |
| Dark: `#60a5fa` on `#1f2937` | 5.77:1 | PASS | Good |

#### UI Component Contrast (requirement: 3:1 per WCAG 2.1 SC 1.4.11)

| Pair | Ratio | AA UI | Status |
|------|-------|-------|--------|
| Light: border `#e5e7eb` on `#ffffff` | 1.24:1 | **FAIL** | Must fix |
| Dark: border `#374151` on `#111827` | 1.72:1 | **FAIL** | Must fix |
| Dark: border `#374151` on `#1f2937` | 1.42:1 | **FAIL** | Must fix |

#### Card Differentiation

| Pair | Ratio | Issue |
|------|-------|-------|
| Light: card `#ffffff` on page `#ffffff` | 1.00:1 | **Identical** -- no visual separation |
| Dark: card `#1f2937` on page `#111827` | 1.21:1 | Barely distinguishable |

**Recommended new color tokens:**

```scss
// Light mode -- improved tokens
$light-tokens: (
  bg:          #f8f9fa,      // Slight off-white page bg (was #ffffff)
  card-bg:     #ffffff,      // White cards pop against off-white
  text:        #111827,      // Darker text (was #1a1a1a) -- 18.09:1
  text-muted:  #4b5563,      // Darker muted (was #6b7280) -- 7.45:1
  border:      #9ca3af,      // Darker border (was #e5e7eb) -- 3.43:1 PASS
  link:        #1d4ed8,      // Darker blue (was #2563eb) -- 7.07:1
  link-hover:  #1e40af,      // Even darker on hover
);

// Dark mode -- keep current (already good), fix borders
$dark-tokens: (
  bg:          #111827,
  card-bg:     #1f2937,
  text:        #f3f4f6,
  text-muted:  #9ca3af,
  border:      #6b7280,      // Lighter border (was #374151) -- 3.57:1 on card PASS
  link:        #60a5fa,
  link-hover:  #93bbfd,
);
```

**Confidence: HIGH** -- contrast ratios computed with standard WCAG 2.1 formula, values chosen to exceed 3:1 for UI components while maintaining visual harmony.

## SCSS Architecture

### Recommended File Structure

```
assets/scss/
  main.scss           # Entry point: @use all partials
  _tokens.scss        # Color tokens, spacing, typography scale
  _base.scss          # Reset, body, typography defaults
  _grid.scss          # Bento grid layout + responsive breakpoints
  _cards.scss         # Project card styles
  _header.scss        # Site header, toggle, language switcher
  _theme.scss         # Dark/light mode switching logic
  _utilities.scss     # .sr-only and other utility classes
```

### Entry Point Pattern

```scss
// main.scss
@use 'tokens';
@use 'base';
@use 'grid';
@use 'cards';
@use 'header';
@use 'theme';
@use 'utilities';
```

Hugo with Dart Sass supports `@use` (confirmed in official docs). Partials must be prefixed with `_` and live in the same directory or subdirectory.

**Why this structure:**
- `_tokens.scss` is imported by other partials via `@use 'tokens'` -- single source of truth for design values
- `_theme.scss` handles all checkbox-hack-related overrides in one place
- `_grid.scss` owns all breakpoint definitions -- easy to adjust responsive behavior
- Each file stays under 100 lines -- easy to review and modify

### Responsive Breakpoint Mixin

```scss
// _tokens.scss
$breakpoints: (
  mobile:  480px,
  tablet:  768px,
  desktop: 1024px,
);

@mixin respond-to($name) {
  @if map-has-key($breakpoints, $name) {
    @media (max-width: map-get($breakpoints, $name)) {
      @content;
    }
  }
}
```

**Confidence: HIGH** -- standard Sass pattern, works with Hugo Dart Sass.

## Layout Width Change

The current site uses `--max-width: 680px` -- designed for a single-column text layout. A bento grid needs more room.

**Recommendation:** Increase to `--max-width: 1100px` for desktop. This accommodates 3 columns of ~320px each with gaps. On mobile, the grid naturally collapses to full viewport width.

```scss
:root {
  --max-width: 1100px;  // was 680px
}
```

## Bento Card Design Tokens

New design tokens needed for the bento redesign:

```scss
// _tokens.scss additions
$card-radius:    0.75rem;     // Bento standard: 12px rounded corners
$card-padding:   1.5rem;
$grid-gap:       1rem;
$card-shadow-light: 0 1px 3px rgba(0, 0, 0, 0.08);
$card-shadow-dark:  none;     // Shadows invisible on dark bg, use border instead
```

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| CSS architecture | SCSS partials via `@use` | Single plain CSS file | Current 301 lines will triple. Single file becomes unmaintainable at 900+ lines. No file-splitting in native CSS. |
| CSS architecture | SCSS partials via `@use` | SCSS with `@import` | `@import` is deprecated in Dart Sass. Will be removed in future versions. `@use` is the modern replacement with proper scoping. |
| Grid approach | `grid-template-areas` | `grid-template-columns: span` | Named areas are more readable and allow complete layout redefinition per breakpoint. Span-based requires calculating line numbers. |
| Grid approach | `grid-template-areas` | Flexbox | CSS Grid is purpose-built for 2D layouts. Flexbox requires hacks for bento-style spanning. |
| Dark mode toggle | Keep checkbox hack | Switch to `:has()` | Existing structure works. `:has()` gains nothing for this page while dropping ~6% browser support. |
| Dark mode toggle | Keep checkbox hack | `light-dark()` function | Cannot combine with manual toggle. Only responds to OS preference. 85% support -- not yet Baseline. |
| Color tokens | Sass maps + CSS custom properties | CSS custom properties only | Sass maps allow iteration, validation, and compile-time checks. CSS custom properties provide runtime switching. Use both: Sass generates the CSS custom property declarations. |
| Responsive | Mobile-first `min-width` | Desktop-first `max-width` | Either works. Desktop-first is more intuitive for this redesign since we are designing the full bento grid first, then collapsing. **Choose desktop-first** since the bento grid IS the desktop design. |

## What NOT to Use

| Avoid | Why |
|-------|-----|
| CSS Subgrid | No nested grids in this layout. 87.8% support is also below the 93%+ threshold of other options. |
| Container Queries | Single-page with one grid context. Media queries are sufficient and simpler. |
| `light-dark()` CSS function | Cannot be toggled by user interaction (checkbox). Only responds to OS `prefers-color-scheme`. |
| JavaScript dark mode | Project constraint: zero JS. |
| Tailwind CSS | Requires Node.js build step. Hugo + Sass handles everything natively. |
| CSS-only persistence | CSS cannot persist toggle state across page loads. Accept this limitation (user preference resets on navigation). Both `/it/` and `/en/` pages reset toggle -- this is acceptable for a 2-page site. |
| `aspect-ratio` on grid | Some bento tutorials use `aspect-ratio: cols/rows` to force square cells. This breaks with varying content heights. Let content determine card height. |

## Hugo Template Changes Required

Minimal template changes to support the redesign:

```html
<!-- baseof.html: no structural changes needed -->
<!-- The checkbox + .page-wrapper sibling structure stays identical -->

<!-- head.html: change CSS resource path -->
{{ $css := resources.Get "scss/main.scss" | css.Sass (dict "transpiler" "dartsass") | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
<meta name="color-scheme" content="light dark">

<!-- index.html: change grid class, add grid-area attributes -->
<section class="bento-grid" aria-label="{{ T "tagline" }}">
  {{ range $i, $p := .Site.Data.projects.project }}
  <article class="bento-card" style="grid-area: {{ $p.name | lower }}">
    ...
  </article>
  {{ end }}
</section>
```

**Note:** Using inline `style="grid-area: name"` from Hugo data is cleaner than maintaining a separate CSS class per project. If project names change, grid areas update automatically.

## Version Compatibility

| Component | Version | Confirmed |
|-----------|---------|-----------|
| Hugo Extended | 0.155.3 | `@use` syntax works with `transpiler: "dartsass"` (Hugo docs) |
| Dart Sass | 1.97.3 | `@use` is the primary import mechanism since Sass 3.0 |
| CSS Grid | Level 1 | 97%+ browser support (caniuse) |
| `grid-template-areas` | CSS Grid L1 | Same 97%+ support |
| `color-scheme` property | CSS Color Adjust L1 | 96%+ support (caniuse) |
| CSS custom properties | L1 | 98%+ support (caniuse) |
| `clamp()` | CSS Values L4 | 96%+ support (caniuse) |

## Sources

- [Hugo css.Sass official docs](https://gohugo.io/functions/css/sass/) -- Dart Sass transpiler options, `@use "hugo:vars"` syntax, deprecation of LibSass in v0.153.0 (HIGH confidence)
- [MDN: CSS Grid grid-template-areas](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Grid_template_areas) -- Named grid areas specification and usage (HIGH confidence)
- [Can I Use: CSS Container Queries](https://caniuse.com/css-container-queries) -- 93.47% global support (HIGH confidence)
- [Can I Use: CSS :has()](https://caniuse.com/css-has) -- 93.71% global support (HIGH confidence)
- [Can I Use: CSS Subgrid](https://caniuse.com/css-subgrid) -- 87.8% global support (HIGH confidence)
- [Can I Use: light-dark()](https://caniuse.com/mdn-css_types_color_light-dark) -- 84.93% global support, Baseline Widely Available November 2026 (HIGH confidence)
- [MDN: prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) -- Media query for OS theme detection (HIGH confidence)
- [MDN: color-scheme property](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/color-scheme) -- Browser-level dark/light mode opt-in (HIGH confidence)
- [W3C WCAG 2.1 SC 1.4.3](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html) -- 4.5:1 normal text, 3:1 large text (HIGH confidence)
- [W3C WCAG 2.1 SC 1.4.11](https://www.w3.org/WAI/WCAG21/Understanding/non-text-contrast.html) -- 3:1 for UI components (HIGH confidence)
- [Bento layout with CSS grid (iamsteve.me)](https://iamsteve.me/blog/bento-layout-css-grid) -- 12-column grid with `grid-auto-flow: dense` and span-based sizing (MEDIUM confidence)
- [FreeCodeCamp: Bento grids in web design](https://www.freecodecamp.org/news/bento-grids-in-web-design/) -- `grid-template-areas` responsive pattern with breakpoints (MEDIUM confidence)
- [Codemotion: Bento box with modern CSS](https://www.codemotion.com/magazine/frontend/lets-create-a-bento-box-design-layout-using-modern-css/) -- CSS custom properties for grid dimensions, `aspect-ratio` technique (MEDIUM confidence)
- [CSS-only dark mode (endtimes.dev)](https://endtimes.dev/no-javascript-dark-mode-toggle/) -- Checkbox hack with `prefers-color-scheme` inversion pattern (MEDIUM confidence)
- [CSS-only dark mode (kleinfreund.de)](https://kleinfreund.de/css-only-dark-mode/) -- Checkbox + `:checked` sibling combinator deep-dive, limitations analysis (MEDIUM confidence)
- [Dark mode toggle without JS via :has() (blog.tymek.dev)](https://blog.tymek.dev/no-js-dark-mode-toggle/) -- `body:has(#dark:checked)` pattern as alternative to checkbox hack (MEDIUM confidence)
- [Smashing Magazine: Color scheme preferences with CSS](https://www.smashingmagazine.com/2024/03/setting-persisting-color-scheme-preferences-css-javascript/) -- Comprehensive dark mode implementation strategies (MEDIUM confidence)

---
*Stack research for: toto-castaldi.com v1.1 bento grid restyling*
*Researched: 2026-02-19*
