# Phase 4: Design Tokens and Container - Research

**Researched:** 2026-02-19
**Domain:** CSS custom properties, WCAG AA contrast, layout architecture
**Confidence:** HIGH

## Summary

Phase 4 fixes WCAG AA contrast failures in both themes and widens the page container from 680px to ~1080px. The current CSS has two critical contrast failures: light mode borders at 1.24:1 (need 3:1) and dark mode borders at 1.72:1 (need 3:1). Additionally, card backgrounds are identical to page backgrounds in light mode (both `#ffffff`), making cards indistinguishable.

The fix requires updating 7 CSS custom properties per theme (14 total across light/dark), changing the page background to off-white in light mode, and restructuring the `.page-wrapper` element to be full-width so the checkbox-hack toggle controls the background across the entire viewport.

**Primary recommendation:** Replace color tokens with WCAG-verified values, restructure `.page-wrapper` to full-width with an inner `.container` for max-width, and update all 4 toggle states (OS-light/dark x checked/unchecked) with the new tokens.

<phase_requirements>

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| LAYOUT-04 | Container max-width increases from 680px to ~1080px | New `.container` element with `max-width: 1080px` inside full-width `.page-wrapper`. See Architecture Patterns. |
| THEME-01 | Light mode legible with WCAG AA contrast on all elements | All proposed light tokens verified: text 16.65-17.40:1, muted 4.63-4.83:1, link 4.95-5.17:1, border 3.04-3.18:1. See Code Examples. |
| THEME-02 | Dark mode legible with WCAG AA contrast on all elements | All proposed dark tokens verified: text 13.35-16.30:1, muted 5.71-6.96:1, link 5.75-7.02:1, border 3.07-3.75:1. See Code Examples. |
| THEME-03 | Dark/light toggle functional (checkbox hack preserved) | Toggle mechanism unchanged -- same `#dark-toggle:checked + .page-wrapper` selector. All 4 states need token updates. See Architecture Patterns. |
| THEME-04 | Cards visually distinct from background in both themes | Light: white cards (`#ffffff`) on off-white page (`#f9fafb`) + 3:1 border. Dark: slate-800 cards (`#1e293b`) on slate-900 page (`#0f172a`) + 3:1 border. |
| THEME-05 | Border contrast >= 3:1 for UI components (WCAG SC 1.4.11) | Light border `#8891a0`: 3.04-3.18:1. Dark border `#64748b`: 3.07-3.75:1. Both pass against all adjacent surfaces. |
| STYLE-05 | Off-white page background in light mode to differentiate cards | Page bg `#f9fafb` (Tailwind gray-50). Subtle enough that muted text still passes 4.5:1 (4.63:1). Cards pop with white bg + visible border. |

</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| CSS Custom Properties | CSS3 | Design token system | Already in use, no build step needed, cascades through DOM |
| Hugo Asset Pipeline | 0.155.3 | CSS minification + fingerprinting | Already configured in `head.html` partial |

### Supporting
No additional libraries needed. Plain CSS handles all requirements.

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Manual contrast verification | Automated CI contrast checker (e.g., Pa11y) | Overkill for 14 tokens; manual verification is faster and documented in this research |
| CSS Custom Properties | SCSS variables | Decided against in v1.1 -- under 500 lines, no build complexity needed |
| `color-mix()` for derived tokens | Hardcoded hex values | `color-mix()` has ~95% browser support but adds cognitive load for 14 straightforward tokens |

**Installation:** None needed -- plain CSS.

## Architecture Patterns

### Current Structure (Problem)
```
body
  input#dark-toggle (checkbox, sr-only)
  div.page-wrapper (max-width: 680px, margin: 0 auto)
    header.site-header
    main
```

**Problem:** `.page-wrapper` has both the max-width constraint AND the background color. When max-width is 680px, the body's background (browser default white) shows on either side. This is invisible now (both white), but will be visible when page bg becomes off-white and when max-width increases to 1080px.

The checkbox hack uses `#dark-toggle:checked + .page-wrapper` (adjacent sibling selector). The toggle can only affect `.page-wrapper` and its descendants -- NOT `body`. So body background cannot be controlled by the toggle.

### Recommended Structure (Solution)
```
body
  input#dark-toggle (checkbox, sr-only)
  div.page-wrapper (FULL WIDTH, min-height: 100vh, background: var(--color-bg))
    div.container (max-width: 1080px, margin: 0 auto, padding)
      header.site-header
      main
```

**Why:** Making `.page-wrapper` full-width means the toggle-controlled background covers the entire viewport. The new `.container` element handles the centering and max-width constraint.

**HTML change in `baseof.html`:**
```html
<body>
  <input type="checkbox" id="dark-toggle" class="sr-only">
  <div class="page-wrapper">
    <div class="container">
      <header class="site-header">...</header>
      <main>{{ block "main" . }}{{ end }}</main>
    </div>
  </div>
</body>
```

**CSS change:**
```css
.page-wrapper {
  /* Remove max-width, keep full-width bg */
  color: var(--color-text);
  background: var(--color-bg);
  min-height: 100vh;
}

.container {
  max-width: 1080px;
  margin: 0 auto;
  padding: var(--space-xl) var(--space-md);
}
```

### Pattern: 4-State Toggle Token Updates

The checkbox hack creates 4 states that each need a complete token set:

| State | CSS Context | Theme Applied |
|-------|-------------|---------------|
| OS-light, unchecked | `:root` (default) | Light tokens |
| OS-light, checked | `#dark-toggle:checked + .page-wrapper` | Dark tokens |
| OS-dark, unchecked | `@media (prefers-color-scheme: dark) :root` + `.page-wrapper` | Dark tokens |
| OS-dark, checked | `@media (prefers-color-scheme: dark) #dark-toggle:checked + .page-wrapper` | Light tokens |

Each state must set ALL 7 tokens: `--color-bg`, `--color-text`, `--color-text-muted`, `--color-border`, `--color-link`, `--color-link-hover`, `--color-card-bg`.

**Current duplication note:** The dark-mode media query currently sets tokens on BOTH `:root` AND `.page-wrapper`. This is because `:root` sets the body-level defaults, while `.page-wrapper` is needed for the checkbox toggle's specificity to override. This pattern must be preserved with the new token values.

### Anti-Patterns to Avoid
- **Changing body background via :root only:** The checkbox toggle cannot affect body because `body` is not a sibling of `#dark-toggle`. If body bg differs from `.page-wrapper` bg during a toggle, the viewport edges flash a different color.
- **Using the same bg for cards and page:** THEME-04 explicitly requires visual distinction. Even subtle off-white vs white matters.
- **Setting border contrast to exactly 3.0:1:** Rounding errors in browsers can cause marginal values to fail. Target 3.04:1+ minimum for safety.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Contrast ratio calculation | Custom formula in CSS | Pre-calculated hex values verified in this research | WCAG luminance formula is non-trivial; pre-verified values eliminate runtime risk |
| Dark mode detection | JS `matchMedia` listener | CSS `prefers-color-scheme` + checkbox hack | Already working, zero-JS constraint, broader browser support |

**Key insight:** All contrast work is done at research/planning time. The implementation is simply replacing hex values in CSS -- no runtime contrast logic needed.

## Common Pitfalls

### Pitfall 1: Muted Text Failing on Off-White Background
**What goes wrong:** Using a darker off-white (e.g., `#f3f4f6`) causes muted text (`#6b7280`) to drop below 4.5:1.
**Why it happens:** Off-white reduces the luminance gap between muted gray text and the background.
**How to avoid:** Use `#f9fafb` (Tailwind gray-50) -- verified at 4.63:1 for muted text.
**Warning signs:** Muted text appearing washed out or hard to read on the page background.

### Pitfall 2: Border Passing Against One Surface But Failing Against Another
**What goes wrong:** A border color passes 3:1 against the page bg but fails against the card bg (or vice versa).
**Why it happens:** Cards sit ON the page bg; borders touch both surfaces.
**How to avoid:** Verify border contrast against BOTH the lighter and darker adjacent surface. The harder constraint (lighter surface) determines the minimum.
**Warning signs:** Borders visible on dark bg cards but invisible on light bg cards.

### Pitfall 3: Toggle State Mismatch
**What goes wrong:** Updating tokens in 3 of 4 toggle states but missing one, causing a state where old and new tokens mix.
**Why it happens:** The 4-state matrix (OS-light/dark x checked/unchecked) is easy to overlook.
**How to avoid:** Update all 4 CSS blocks that define color tokens. Test each state.
**Warning signs:** Toggle works in one OS preference but shows wrong colors in the other.

### Pitfall 4: Viewport Edge Color Flash on Toggle
**What goes wrong:** When toggling dark/light, the body area outside `.page-wrapper` stays the old color for a frame.
**Why it happens:** `body` bg is set by `:root`/media query, not by the checkbox toggle.
**How to avoid:** Make `.page-wrapper` full-width so it covers the entire viewport. The toggle then controls the whole visible background.
**Warning signs:** Flickering white/dark strips on viewport edges during toggle.

### Pitfall 5: Container Padding Collapse on Mobile
**What goes wrong:** With `max-width: 1080px`, horizontal padding gets removed or too small on narrow screens.
**Why it happens:** Fixed padding values don't adapt to viewport.
**How to avoid:** Use `padding: var(--space-xl) var(--space-md)` -- the `--space-md` (1.5rem/24px) provides adequate mobile padding regardless of max-width.
**Warning signs:** Content touching viewport edges on small screens.

## Code Examples

Verified token values from contrast analysis (computed with WCAG 2.0 relative luminance formula):

### Complete Light Mode Token Set
```css
/* Light mode (default) */
:root {
  --color-bg: #f9fafb;        /* off-white page (was #ffffff) */
  --color-text: #1a1a1a;      /* unchanged */
  --color-text-muted: #6b7280;/* unchanged */
  --color-border: #8891a0;    /* was #e5e7eb (1.24:1) -> now 3.18:1 on white */
  --color-link: #2563eb;      /* unchanged */
  --color-link-hover: #1d4ed8;/* unchanged */
  --color-card-bg: #ffffff;   /* white cards on off-white page */
}
```

### Complete Dark Mode Token Set
```css
/* Dark mode */
--color-bg: #0f172a;        /* Tailwind slate-900 (was #111827) */
--color-text: #f1f5f9;      /* Tailwind slate-100 (was #f3f4f6) */
--color-text-muted: #94a3b8;/* Tailwind slate-400 (was #9ca3af) */
--color-border: #64748b;    /* Tailwind slate-500 (was #374151, 1.72:1) -> now 3.07:1 */
--color-link: #60a5fa;      /* unchanged */
--color-link-hover: #93bbfd;/* unchanged */
--color-card-bg: #1e293b;   /* Tailwind slate-800 (was #1f2937) */
```

### Contrast Verification Table (All PASS)

#### Light Mode
| Element | Foreground | Background | Ratio | Threshold | Result |
|---------|-----------|------------|-------|-----------|--------|
| Text on page | #1a1a1a | #f9fafb | 16.65:1 | 4.5:1 | PASS |
| Text on card | #1a1a1a | #ffffff | 17.40:1 | 4.5:1 | PASS |
| Muted on page | #6b7280 | #f9fafb | 4.63:1 | 4.5:1 | PASS |
| Muted on card | #6b7280 | #ffffff | 4.83:1 | 4.5:1 | PASS |
| Link on page | #2563eb | #f9fafb | 4.95:1 | 4.5:1 | PASS |
| Link on card | #2563eb | #ffffff | 5.17:1 | 4.5:1 | PASS |
| Link hover on page | #1d4ed8 | #f9fafb | 6.41:1 | 4.5:1 | PASS |
| Link hover on card | #1d4ed8 | #ffffff | 6.70:1 | 4.5:1 | PASS |
| Border on page | #8891a0 | #f9fafb | 3.04:1 | 3.0:1 | PASS |
| Border on card | #8891a0 | #ffffff | 3.18:1 | 3.0:1 | PASS |

#### Dark Mode
| Element | Foreground | Background | Ratio | Threshold | Result |
|---------|-----------|------------|-------|-----------|--------|
| Text on page | #f1f5f9 | #0f172a | 16.30:1 | 4.5:1 | PASS |
| Text on card | #f1f5f9 | #1e293b | 13.35:1 | 4.5:1 | PASS |
| Muted on page | #94a3b8 | #0f172a | 6.96:1 | 4.5:1 | PASS |
| Muted on card | #94a3b8 | #1e293b | 5.71:1 | 4.5:1 | PASS |
| Link on page | #60a5fa | #0f172a | 7.02:1 | 4.5:1 | PASS |
| Link on card | #60a5fa | #1e293b | 5.75:1 | 4.5:1 | PASS |
| Link hover on page | #93bbfd | #0f172a | 9.15:1 | 4.5:1 | PASS |
| Link hover on card | #93bbfd | #1e293b | 7.50:1 | 4.5:1 | PASS |
| Border on page | #64748b | #0f172a | 3.75:1 | 3.0:1 | PASS |
| Border on card | #64748b | #1e293b | 3.07:1 | 3.0:1 | PASS |

### Container + Wrapper CSS
```css
.page-wrapper {
  color: var(--color-text);
  background: var(--color-bg);
  min-height: 100vh;
  /* NO max-width -- full viewport width for background coverage */
}

.container {
  max-width: 1080px;
  margin: 0 auto;
  padding: var(--space-xl) var(--space-md);
}
```

### Layout Token Update
```css
:root {
  /* Layout */
  --max-width: 1080px; /* was 680px */
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Hardcoded colors everywhere | CSS custom properties (design tokens) | CSS3 (2015+) | Already in use -- just updating values |
| `color-contrast()` CSS function | Pre-calculated tokens | CSS Color L5 (not yet stable) | Cannot rely on CSS-native contrast; must pre-verify |
| WCAG 2.x relative luminance | APCA (Accessible Perceptual Contrast Algorithm) | Under development (WCAG 3.0 draft) | WCAG 2.x is still the legal standard; APCA is informational only |

**Deprecated/outdated:**
- WCAG 2.0 AAA (7:1 text) is aspirational, not required. AA (4.5:1) is the target.
- `color-contrast()` CSS function would automate contrast checks but has no stable browser support.

## Open Questions

1. **Card distinction via background vs. border**
   - What we know: Card/page bg contrast is subtle (1.05:1 light, 1.22:1 dark). The 3:1+ border provides the primary visual boundary.
   - What's unclear: Whether the subtle bg difference alone is sufficient for THEME-04, or whether the border is doing the heavy lifting.
   - Recommendation: Accept both mechanisms together. The border provides the WCAG-measurable boundary; the bg difference adds a reinforcing visual cue. Phase 5 will add `border-radius` and Phase 6 will add shadows, further distinguishing cards.

2. **body background on toggle**
   - What we know: Making `.page-wrapper` full-width solves the viewport-edge color mismatch during toggle.
   - What's unclear: Whether setting `body { background: var(--color-bg) }` as a fallback (from `:root` only, not toggle-aware) adds value or causes confusion.
   - Recommendation: Set `body { background: var(--color-bg) }` as a belt-and-suspenders fallback. It won't respond to the toggle, but `.page-wrapper` covers 100% of the viewport anyway. The body bg only matters in edge cases (e.g., overscroll bounce on iOS).

## Sources

### Primary (HIGH confidence)
- WCAG 2.1 SC 1.4.3 (Contrast Minimum) and SC 1.4.11 (Non-text Contrast) -- W3C Recommendation
- Current codebase: `assets/css/main.css` (300 lines), `layouts/_default/baseof.html`, `layouts/index.html`
- Contrast ratios computed with WCAG 2.0 relative luminance formula (verified via Node.js script)

### Secondary (MEDIUM confidence)
- Tailwind CSS color palette (gray-50 through gray-900, slate-100 through slate-900) as reference for harmonious values
- WCAG SC 1.4.11 interpretation: borders need 3:1 against adjacent background when they are the sole visual indicator of a UI component (verified via W3C Understanding doc and yatil.net analysis)

### Tertiary (LOW confidence)
- None -- all findings verified with primary sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- plain CSS custom properties, no libraries to evaluate
- Architecture: HIGH -- restructure is minimal (add one wrapper div, move max-width), checkbox hack mechanism unchanged
- Pitfalls: HIGH -- all contrast values mathematically verified, all 4 toggle states documented
- Token values: HIGH -- computed with WCAG 2.0 formula, every pair verified against both page and card backgrounds

**Research date:** 2026-02-19
**Valid until:** 2026-06-19 (stable domain -- WCAG 2.x won't change, CSS custom properties are mature)
