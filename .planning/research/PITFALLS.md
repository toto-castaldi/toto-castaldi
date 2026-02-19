# Domain Pitfalls

**Domain:** Bento grid redesign + dark/light theme overhaul for existing Hugo landing page
**Researched:** 2026-02-19
**Confidence:** HIGH (codebase analysis + verified community patterns + official CSS/Hugo docs)

## Critical Pitfalls

Mistakes that cause visual breakage, accessibility failure, or require rewriting completed work.

### Pitfall 1: Checkbox Hack Dark Mode Breaks When Grid Structure Changes

**What goes wrong:** The current dark mode uses `#dark-toggle:checked + .page-wrapper` -- the adjacent sibling combinator (`+`). This means `.page-wrapper` MUST be the immediate next sibling of the checkbox input. If the bento grid redesign introduces any wrapper element between `#dark-toggle` and `.page-wrapper`, or moves the checkbox inside `.page-wrapper`, the entire dark mode toggle silently stops working. No error, no warning -- just a toggle that does nothing.

**Why it happens:** During a layout overhaul, developers naturally restructure the DOM to accommodate the new grid. Adding a grid container, moving elements around, or wrapping sections in new divs is reflexive. The sibling selector constraint is invisible -- it's not documented in any template, only implicit in the CSS.

**Consequences:** Dark mode toggle appears functional (checkbox still toggles) but no visual change occurs. All 4 CSS variable override blocks (lines 78-101 of current `main.css`) plus 4 icon-visibility blocks (lines 280-287) stop matching. The site is locked to whatever the OS preference is.

**Prevention:**
1. **Before touching HTML:** Add a comment in `baseof.html` on the checkbox line: `<!-- CRITICAL: #dark-toggle must be immediately before .page-wrapper (CSS + sibling selector) -->`
2. **After every HTML change:** Test dark mode toggle in both OS light and OS dark mode
3. **Consider upgrading to `:has()` selector:** `body:has(#dark-toggle:checked)` eliminates the sibling constraint entirely. `:has()` is Baseline Widely Available (Chrome 105+, Safari 15.4+, Firefox 121+, Edge 105+) -- safe for 2026. This decouples the toggle from DOM position.

**Detection:** Toggle button click produces no visual change. Easy to miss if only testing in one OS color scheme.

**Phase to address:** Phase 1 (before any HTML restructuring). Either lock down the DOM constraint with comments and tests, or migrate to `:has()` first.

---

### Pitfall 2: Bento Grid Source Order vs. Visual Order Breaks Accessibility

**What goes wrong:** CSS Grid's `grid-column`, `grid-row`, and `grid-template-areas` let you place items anywhere visually. The bento layout's appeal is asymmetric, varied-size tiles. But screen readers and keyboard tab order follow DOM source order, not visual order. A visually logical layout can produce a nonsensical reading sequence for assistive technology users.

**Why it happens:** Developers design the grid visually first (placing the "hero" card spanning 2 columns in the center, smaller cards around it) then discover the HTML source order doesn't match. The temptation is to use CSS placement to "fix" the visual order without changing HTML, creating a mismatch.

**Consequences:** WCAG 1.3.2 (Meaningful Sequence) violation. Screen readers announce cards in a random-seeming order. Tab key jumps unpredictably across the page. For a 3-project site this is manageable, but it's still a failure point for accessibility audits.

**Prevention:**
1. **Design HTML source order FIRST.** Write the HTML in the order you want it read aloud. For 3 project cards + a hero section, the natural order is: hero/title, then Docora, Lumio, Helix.
2. **Use `grid-template-areas` to map visual layout.** Areas let you name positions and rearrange visually without reordering HTML -- but only if the source order was logical to begin with.
3. **Test with keyboard-only navigation.** Tab through the page; the focus order should flow logically top-to-bottom, left-to-right.
4. **Avoid `order` property on grid items.** It changes visual order but not tab/reading order, creating exactly the mismatch you want to avoid.

**Detection:** Tab through the deployed page with keyboard only. If focus jumps visually (e.g., from top-left to bottom-right, then back to top-right), source order is wrong.

**Phase to address:** Phase 1 (HTML structure design). Must be decided before writing any CSS grid rules.

---

### Pitfall 3: Dark Mode Contrast Fails WCAG AA in One Theme But Not the Other

**What goes wrong:** Colors that pass WCAG AA 4.5:1 contrast ratio in light mode fail in dark mode (or vice versa). The current site has `--color-text-muted: #6b7280` on `--color-bg: #ffffff` (light) = 5.0:1 ratio (passes AA). Dark mode has `--color-text-muted: #9ca3af` on `--color-bg: #111827` = 4.2:1 (FAILS AA for normal text). This existing bug will compound when adding more color tokens for bento card backgrounds, borders, and accent colors.

**Why it happens:** Designers pick colors that "look right" in one mode and assume inverting them works for the other. Dark mode is not just light mode with swapped background/foreground -- the contrast math is asymmetric. Adding card-specific background colors (e.g., for bento tiles with colored headers) multiplies the number of foreground/background pairs that must each individually pass contrast checks.

**Consequences:** WCAG AA failure. Legal risk in some jurisdictions. More practically: muted text becomes genuinely unreadable for users with low vision, especially on mobile screens in sunlight (light mode) or in dark rooms (dark mode).

**Prevention:**
1. **Audit every color pair in both themes BEFORE coding.** Use WebAIM Contrast Checker for each `--color-*` token against its expected background.
2. **Current dark mode `--color-text-muted` needs fixing.** Change from `#9ca3af` to `#a1a9b8` or lighter to achieve 4.5:1 against `#111827`.
3. **If adding bento tile accent colors,** create a contrast matrix: every foreground color x every background color it could appear on x both themes. For 3 projects with distinct accent colors, that's ~18 pairs minimum.
4. **Use `color-contrast()` CSS function** if targeting modern browsers (currently limited support -- verify before relying on it).
5. **WCAG AA requirements:** 4.5:1 for normal text, 3:1 for large text (18px bold or 24px regular), 3:1 for UI components and graphical objects.

**Detection:** Run axe-core or Lighthouse accessibility audit on both themes. Check the "Color Contrast" section specifically.

**Phase to address:** Phase 1 (design tokens). Fix existing contrast bug and validate all new tokens before building any grid components.

---

### Pitfall 4: Bento Grid Collapses to Identical-Width Stacked Blocks on Mobile

**What goes wrong:** The carefully designed desktop bento layout with varied tile sizes (1x1, 2x1, 1x2 spanning cards) collapses to a single column on mobile where every card looks identical. The visual hierarchy -- the entire point of bento -- is lost. The page becomes a boring vertical stack indistinguishable from the current non-bento design.

**Why it happens:** The default responsive approach is `grid-template-columns: 1fr` on mobile. This makes every card full-width, destroying size variation. Developers add breakpoints for desktop but treat mobile as an afterthought single-column layout.

**Consequences:** Mobile users (typically 60%+ of traffic) never see the bento design. The redesign effort is invisible to the majority of visitors. Worse: if cards have unequal content lengths, the stacked mobile layout has awkward height variations with no visual logic.

**Prevention:**
1. **Design mobile-first with a 2-column minimum.** Even on 320px viewports, a 2-column grid is possible. Use `grid-template-columns: repeat(2, 1fr)` as the mobile base, letting one "hero" card span both columns while others stay 1-column.
2. **Define 3 breakpoints minimum:** mobile (2-col), tablet (3-col), desktop (4-col). Each needs its own `grid-template-areas` definition.
3. **Use `aspect-ratio` on grid items** to maintain proportional sizing even when columns change. A card that's 2:1 on desktop should still feel wider-than-tall on mobile.
4. **Hide decorative-only bento tiles on mobile.** If any tiles are purely visual (no content), hide them below a breakpoint rather than letting them eat vertical space.
5. **Test on real 320px viewport.** Chrome DevTools' responsive mode is not enough -- test on actual small phones.

**Detection:** View on a 375px-wide viewport. If all cards are identical rectangles stacked vertically, the bento design is lost.

**Phase to address:** Phase 2 (responsive implementation). But the grid area names and HTML structure must support this from Phase 1.

---

## Moderate Pitfalls

### Pitfall 5: Hugo CSS Fingerprint Caching Masks Theme Changes During Development

**What goes wrong:** Hugo's `resources.Fingerprint` (used in the current `head.html`: `resources.Get "css/main.css" | resources.Minify | resources.Fingerprint`) generates a content-hash filename. During rapid CSS iteration, the browser cache aggressively holds the old hashed file. Developers think their CSS changes aren't working when they actually are -- the browser is just serving the cached old version.

**Why it happens:** The fingerprint changes with every content change, but the browser may cache the old file aggressively if using `hugo server` (which serves from memory and may not bust browser cache between reloads). Additionally, some CDN or service worker caches at the GitHub Pages level can hold stale CSS for minutes.

**Prevention:**
1. **Use `hugo server --disableFastRender`** during CSS-heavy development to ensure full rebuilds.
2. **Hard-refresh (Ctrl+Shift+R) after CSS changes** during development.
3. **After deploying,** verify with a private/incognito window that the new CSS hash is being served.
4. **Do NOT switch to unhashed CSS filenames** -- the fingerprinting is correct production behavior. Just be aware of its development friction.

**Detection:** CSS changes visible in source file but not in browser. Hash in CSS filename unchanged after modification.

**Phase to address:** Ongoing throughout all phases. Document this in developer notes.

---

### Pitfall 6: `max-width: 680px` Constraint Prevents Bento Grid From Working

**What goes wrong:** The current `.page-wrapper` has `max-width: var(--max-width)` where `--max-width: 680px`. A bento grid with 3-4 columns of meaningful tiles needs at least 900-1200px. At 680px, you can fit at most 2 narrow columns, defeating the visual impact of bento. The grid either looks cramped or the constraint forces a boring 1-2 column layout.

**Why it happens:** The 680px max-width was correct for the current text-focused card list layout (optimal reading width). Developers start adding grid CSS without updating the container width, then wonder why tiles are too small.

**Consequences:** Bento tiles are squished. Text overflows or gets too small. The layout looks worse than the original because it's trying to be a grid in too little space.

**Prevention:**
1. **Increase `--max-width` to 1100-1200px** for the bento container, or remove it entirely and use the grid's own column sizing.
2. **Consider a full-bleed bento grid** that uses viewport width with padding, independent of the page-wrapper constraint.
3. **Keep narrow max-width for text-only sections** (hero title, tagline) while allowing the grid to go wider. This means either separate containers or using CSS Grid itself to create narrow-center and wide-grid zones.

**Detection:** Grid tiles narrower than ~200px each. Cards feel cramped compared to reference bento designs.

**Phase to address:** Phase 1 (layout token changes). Must be updated before grid CSS is written.

---

### Pitfall 7: Grid Template Areas Maintenance Burden Across 3+ Breakpoints

**What goes wrong:** `grid-template-areas` is readable and powerful, but each breakpoint needs a complete redefinition of the area map. For 3 breakpoints (mobile, tablet, desktop) with 6+ named areas, that's 3 complete ASCII-art grids to maintain. Adding a new project card means updating all 3 area maps. Forgetting one produces a broken layout at that breakpoint.

**Why it happens:** Developers love `grid-template-areas` for its visual clarity and use it for everything. It works beautifully for static layouts. But a landing page that may add/remove projects needs a grid system that adapts to content count.

**Prevention:**
1. **Use `grid-template-areas` only for the macro layout** (header zone, grid zone, footer zone). Use auto-placement (`grid-auto-flow: dense`) for the project cards themselves.
2. **For project cards specifically,** use `grid-column: span 2` / `span 1` modifiers on individual cards rather than defining named areas for each card. This scales to any number of cards.
3. **If using named areas for cards,** add a comment block above each `@media` rule listing which areas exist and what they map to. Make the 3 grid maps visually aligned in the source code so differences are obvious.

**Detection:** Adding a 4th project requires editing CSS in multiple places. Layout breaks at one specific viewport width.

**Phase to address:** Phase 1 (grid architecture decision). Choose the grid strategy before writing CSS.

---

### Pitfall 8: Dark Mode Toggle State Lost on Language Switch

**What goes wrong:** User toggles dark mode on the Italian page, then clicks the language switcher to English. The page reloads (Hugo generates separate HTML per language). The checkbox resets to unchecked. User is back in the default theme, losing their preference. This is an EXISTING limitation of the checkbox hack but becomes more noticeable with a visual redesign that draws attention to the toggle.

**Why it happens:** CSS-only checkbox state lives in the DOM. Page navigation = new DOM = lost state. This is fundamental to the zero-JS constraint. There is no CSS mechanism to persist state across page loads.

**Consequences:** Users who manually toggle theme must re-toggle on every page navigation. With only 2 pages (IT/EN) this is tolerable but annoying. It contradicts user expectations -- every other dark mode toggle they've used persists.

**Prevention:**
1. **Accept this as a known limitation of zero-JS** and document it. The OS preference still works correctly -- only the manual override is lost.
2. **If this becomes a real user complaint,** the fix requires minimal JS: `localStorage.setItem('theme', 'dark')` on toggle, `localStorage.getItem('theme')` on page load to set checkbox state. This breaks the zero-JS constraint but is ~5 lines of progressive enhancement.
3. **Do NOT try to solve this with CSS-only hacks** (cookie-based tricks, iframe persistence). They all have worse UX than the reload problem.
4. **Consider `:has()` migration.** It doesn't solve persistence but simplifies the CSS structure, making a future JS enhancement easier to add.

**Detection:** Toggle dark mode, click language switch, observe theme resets.

**Phase to address:** Phase 1 (decide whether to accept or fix). If accepting, document. If fixing, add minimal JS as progressive enhancement.

---

### Pitfall 9: Card Link Overlay (`::after` Trick) Conflicts with Bento Grid Item Sizing

**What goes wrong:** The current `.card-link::after { position: absolute; inset: 0; }` makes the entire card clickable by stretching an invisible overlay. This works because `.project-card` has `position: relative`. In a bento grid, if cards use `grid-column: span 2` or `grid-row: span 2`, the `::after` overlay may not cover the full visual card area if the card's `position: relative` context doesn't match the grid cell dimensions (e.g., when padding or gap creates discrepancies).

**Why it happens:** The overlay sizes itself to the card's padding box. If grid gap is outside the card (between cards), this works fine. But if developers add inner grid structures to cards (sub-grids for card content), the `::after` position context can shift.

**Prevention:**
1. **Keep `.project-card { position: relative }` and ensure `::after` uses `inset: 0`** -- this should work as-is if the card is the direct grid item.
2. **Test click targets after grid implementation.** Click the edges and corners of each card, especially spanning cards.
3. **If cards get an inner layout (sub-grid for icon/title/description),** ensure the `::after` is on an element that's a direct child of the `position: relative` container, not nested deeper.

**Detection:** Clicking near the edge of a bento card doesn't navigate. Gap between cards is not clickable (expected) but edge of card should be.

**Phase to address:** Phase 2 (card component implementation). Test after styling each card variant.

---

## Minor Pitfalls

### Pitfall 10: Fluid Typography Clamp Values Don't Scale With Wider Viewport

**What goes wrong:** Current `clamp()` values (e.g., `--font-size-xl: clamp(1.73rem, 0.59vw + 1.57rem, 2.07rem)`) were tuned for a 680px max-width container. At 1200px bento width, text may still be at the clamp maximum (2.07rem) when it should scale larger to fill bigger tiles. Bento hero tiles with large headings look undersized.

**Prevention:** Re-calculate clamp values for the new max viewport target. Use a tool like utopia.fyi to generate a new fluid type scale for the wider layout.

**Phase to address:** Phase 2 (typography refinement).

---

### Pitfall 11: External Logo Images Break Bento Tile Aspect Ratios

**What goes wrong:** Project logos are loaded from external subdomains (e.g., `https://docora.toto-castaldi.com/images/logo.svg`). If these SVGs lack `viewBox` attributes or have fixed `width`/`height`, they won't scale properly within bento tiles that vary in size. A logo that's 48x48 in the current layout may need to be 64x64 or 96x96 in a larger bento tile.

**Prevention:** Ensure SVG logos have `viewBox` and no fixed `width`/`height` attributes. Use CSS `width: 100%; height: auto; max-width: [size]` rather than fixed pixel dimensions.

**Phase to address:** Phase 2 (card component styling).

---

### Pitfall 12: `prefers-color-scheme` Media Query Duplication Across Grid-Specific Styles

**What goes wrong:** The current CSS already duplicates dark mode tokens in 4 places (`:root`, `.page-wrapper`, `#dark-toggle:checked + .page-wrapper`, and `@media dark + #dark-toggle:checked`). Adding bento-grid-specific dark mode styles (e.g., tile accent colors, grid borders) means adding rules to all 4 blocks. Missing one block means the feature works in 3 of 4 theme states but breaks in the 4th.

**Prevention:**
1. **Migrate to `:has()` selector** to reduce the 4 blocks to 2 (light defaults + dark override via `:has(#dark-toggle:checked)` or `prefers-color-scheme`).
2. **If keeping checkbox hack,** use CSS custom properties exclusively. Never use hardcoded color values in component styles. Every color reference must go through `var(--color-*)` tokens so the 4 toggle blocks control everything.
3. **Add a "theme state matrix" comment** at the top of the CSS listing all 4 states and which CSS block handles each.

**Detection:** Feature looks correct in OS-dark mode but wrong when manually toggled to dark, or vice versa. Must test all 4 combinations: OS-light + unchecked, OS-light + checked, OS-dark + unchecked, OS-dark + checked.

**Phase to address:** Phase 1 (theme architecture). Decide on `:has()` migration before adding any new theme tokens.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| HTML restructuring for grid | Dark mode sibling selector breaks (#1) | Comment the constraint or migrate to `:has()` first |
| Grid area design | Source order mismatch (#2) | Write HTML in reading order, then map visually with areas |
| Design tokens | Contrast failure in one theme (#3) | Audit every token pair in both themes before coding |
| Responsive grid | Mobile bento collapse (#4) | Design mobile 2-col minimum, not single-col fallback |
| Container width | 680px too narrow for grid (#6) | Update `--max-width` before writing grid CSS |
| Breakpoint grid areas | Maintenance burden (#7) | Use auto-placement for cards, named areas only for macro layout |
| Language switch | Theme state lost (#8) | Accept as known limitation or add 5 lines of progressive JS |
| Card interactivity | Click overlay mismatch (#9) | Test click targets on all card size variants |
| New color tokens | 4-way theme duplication (#12) | Migrate to `:has()` or strictly use `var()` tokens only |

## Integration Risks (Existing Features That Break During Redesign)

| Existing Feature | How It Could Break | Prevention |
|------------------|-------------------|------------|
| Language switcher position | Moves from `site-header` flex layout to grid layout; may lose right-alignment or overlap with toggle | Keep header as flex container inside the grid, or explicitly place it in a grid area |
| `card-link::after` overlay | Position context shifts in grid items | Verify `.project-card` remains `position: relative` and is the direct grid item |
| Hugo `resources.Fingerprint` | CSS changes appear not to work during development | Hard-refresh; don't remove fingerprinting |
| `.page-wrapper` as dark mode target | Any structural change between checkbox and wrapper kills dark mode | Lock DOM relationship or migrate to `:has()` |
| `hreflang` / SEO partials | No risk from CSS changes; these are in `<head>` and won't be affected by layout changes | No action needed -- but verify after deployment |
| `aria-label` on project cards section | If grid restructuring moves the `<section>` or changes its role, screen readers lose context | Keep `aria-label` on the grid container |

## Testing Matrix (Must-Pass for Each Phase)

Every pull request during the redesign should verify:

| Test | Method | All 4 theme states? |
|------|--------|---------------------|
| Dark mode toggle works | Click toggle, observe theme change | Yes: OS-light+unchecked, OS-light+checked, OS-dark+unchecked, OS-dark+checked |
| Language switch preserves page | Click IT/EN, verify correct URL and content | Both languages |
| Cards are clickable | Click each card, verify navigation | All cards in both themes |
| Keyboard navigation logical | Tab through page, verify focus order | Visual order matches tab order |
| Text contrast passes AA | Lighthouse or axe audit | Both light and dark themes |
| Mobile layout has visual hierarchy | View at 375px viewport width | Cards should NOT all be identical width |
| No horizontal scroll | View at 320px minimum | Both orientations |
| Logos render at correct size | Visual check in each card variant | Spanning and non-spanning cards |

## Sources

- [MDN: CSS Grid Layout and Accessibility](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_and_accessibility) -- HIGH confidence
- [MDN: :has() CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/:has) -- HIGH confidence
- [MDN: prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) -- HIGH confidence
- [WebAIM: Contrast and Color Accessibility](https://webaim.org/articles/contrast/) -- HIGH confidence
- [BOIA: Dark Mode Doesn't Satisfy WCAG Contrast](https://www.boia.org/blog/offering-a-dark-mode-doesnt-satisfy-wcag-color-contrast-requirements) -- MEDIUM confidence
- [CSS-Tricks: Grid Content Re-ordering and Accessibility](https://css-tricks.com/grid-content-re-ordering-and-accessibility/) -- MEDIUM confidence
- [kleinfreund.de: CSS-only Dark Mode](https://kleinfreund.de/css-only-dark-mode/) -- MEDIUM confidence
- [inkbotdesign.com: Bento Grid Design 2026](https://inkbotdesign.com/bento-grid-design/) -- MEDIUM confidence
- [iamsteve.me: Build a Bento Layout with CSS Grid](https://iamsteve.me/blog/bento-layout-css-grid) -- MEDIUM confidence
- [wearedevelopers.com: Building Bento Grid with Modern CSS Grid](https://www.wearedevelopers.com/en/magazine/682/building-a-bento-grid-layout-with-modern-css-grid-682) -- MEDIUM confidence
- [johnjago.com: Hugo Detecting CSS Changes but Not Rendering](https://johnjago.com/hugo-css-mystery/) -- MEDIUM confidence
- [W3C WCAG Issue #2889: Dark Mode and Contrast](https://github.com/w3c/wcag/issues/2889) -- HIGH confidence
- Codebase analysis of `/home/toto/scm-projects/toto-castaldi/assets/css/main.css` (lines 78-101, 280-287) -- HIGH confidence
- Codebase analysis of `/home/toto/scm-projects/toto-castaldi/layouts/_default/baseof.html` -- HIGH confidence

---
*Pitfalls research for: Bento grid redesign + dark/light theme overhaul*
*Researched: 2026-02-19*
