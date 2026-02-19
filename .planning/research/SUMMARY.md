# Project Research Summary

**Project:** toto-castaldi.com v1.1 Bento Grid Restyling
**Domain:** Hugo static landing page — CSS layout redesign and accessibility overhaul
**Researched:** 2026-02-19
**Confidence:** HIGH

## Executive Summary

This milestone is a pure CSS/template redesign of an existing production Hugo site. No infrastructure changes are needed: Hugo Extended 0.155.3, GitHub Actions, and GitHub Pages all stay as-is. The redesign has two inseparable goals — replace the single-column project card list with a 3-column bento grid, and fix the existing WCAG AA contrast failures (border contrast fails at 1.24:1 in light mode and 1.72:1 in dark mode; both require a minimum of 3:1 for UI components per WCAG 2.1 SC 1.4.11). These two goals must be addressed together because fixing color tokens is a prerequisite for all visual polish work.

The recommended approach is conservative and deliberate: widen the page container from 680px to 1080px, implement CSS Grid with `grid-template-columns: repeat(3, 1fr)` and media-query-driven breakpoints at 768px (2-col tablet) and 480px (1-col mobile), and fix design tokens before touching templates. The existing CSS-only checkbox-hack dark mode toggle is kept intact (100% browser support, already working). The critical architectural risk is that any DOM restructuring can silently break the `#dark-toggle:checked + .page-wrapper` adjacent-sibling selector. One notable disagreement between research files: ARCHITECTURE.md recommends staying with plain CSS while STACK.md recommends migrating to SCSS. This summary resolves the disagreement in favor of plain CSS — the CSS file will be under 500 lines after the rewrite, CSS custom properties provide the token system, and adding Dart Sass processing is premature at this scale.

The main risks are the dark mode toggle breaking during HTML restructuring (silent failure, easy to miss in testing), contrast failures being introduced by new bento color tokens if not audited in all 4 theme states (OS-light/dark x toggle-checked/unchecked), and mobile collapse producing a boring identical-card stack that defeats the bento aesthetic. All three risks have clear preventive patterns documented in the research. No external dependencies or new infrastructure are required — the entire redesign is HTML and CSS.

## Key Findings

### Recommended Stack

The current stack requires no changes at the infrastructure level. Hugo Extended 0.155.3 with Dart Sass already installed in CI is the complete build environment. Dart Sass is present but currently unused — both STACK.md and ARCHITECTURE.md address whether to activate it, reaching opposite conclusions. This summary resolves the disagreement: stay with plain CSS.

**Core technologies:**
- Hugo Extended 0.155.3: static site generation and Hugo Pipes asset processing — keep as-is, zero changes needed
- Plain CSS (`assets/css/main.css`): CSS Grid, custom properties, checkbox-hack dark mode — rewrite in-place, no SCSS migration
- CSS Grid Level 1 (97%+ browser support): bento layout via `grid-template-columns` — primary new technique
- `color-scheme: light dark` CSS property (96%+ support): browser-level dark/light awareness — add to `:root`, single line of significant visual polish
- GitHub Actions + GitHub Pages: CI/CD and hosting — no changes

**Rejected technologies (with rationale):**
- SCSS migration: premature at under 500 lines; CSS custom properties already provide token system; native CSS nesting at 91% support handles any nesting needs
- CSS Subgrid: no nested grids in this layout; 87.8% support is below threshold
- Container Queries: single-page with one grid context; media queries are sufficient and simpler
- `light-dark()` CSS function: cannot combine with checkbox toggle (only responds to OS preference); 85% support, not yet Baseline Widely Available
- JavaScript dark mode: violates zero-JS project constraint

### Expected Features

The redesign has a clear must-have/should-have/defer split. All must-have features are pure CSS changes with no HTML rewrites beyond class additions.

**Must have (table stakes):**
- 3-column desktop bento grid — the entire point of v1.1; CSS Grid `repeat(3, 1fr)` on the existing project cards container
- Responsive collapse: 3-col desktop → 2-col tablet (≥768px) → 1-col mobile (≤480px)
- Wider page container: `--max-width` from 680px to 1080px — gating change; without this the grid is cramped at ~200px per column
- WCAG AA contrast fix: border contrast (light 1.24:1, dark 1.72:1) must reach 3:1; muted text improved in light mode
- Cards visually distinct from background: card-bg differentiated from page-bg in light mode (currently identical at #ffffff)

**Should have (differentiators):**
- Rounded corners increased to 12-16px — signature bento aesthetic; current 8px is too subtle
- Layered box-shadow for depth — CSS variables, theme-aware values per mode; layered approach avoids dated flat-shadow look
- Card hover elevation (`transform: translateY(-2px)` + shadow increase) — interactive tactile feel without JavaScript
- `prefers-reduced-motion` media query — accessibility; currently missing from the site entirely
- `color-scheme: light dark` on `:root` — native form controls and scrollbars adapt to theme automatically

**Defer to future milestone:**
- Extra bento sections (tech stack, GitHub stats) — scope creep for v1.1; ship 3-card grid first
- Variable-size bento tiles (2x1, 1x2 spans) — creates false hierarchy with 3 equal-weight projects
- Per-project accent color strips on card borders — optional; adds complexity; evaluate after seeing live layout
- CSS-only dark mode persistence across page navigation — requires JS; accept as known limitation

**Anti-features to explicitly avoid:**
- `grid-auto-flow: dense` — reorders DOM vs visual order, WCAG 1.3.2 accessibility violation
- Glassmorphism / `backdrop-filter` — contrast calculation complexity, performance cost, trend that ages poorly
- JavaScript card animations — violates zero-JS constraint
- Icon fonts or external icon libraries — unnecessary dependency

### Architecture Approach

The redesign modifies 3 existing files and adds no new ones. `assets/css/main.css` is rewritten with a new section structure. `layouts/_default/baseof.html` gains a `class="bento-grid"` attribute on `<main>`. `layouts/index.html` restructures the page into bento cells (hero section + 3 project cards as direct grid children, eliminating the now-redundant `.project-cards` wrapper). The dark mode toggle's DOM relationship — `#dark-toggle` as direct sibling of `.page-wrapper` — is the hardest architectural constraint and must be preserved throughout all changes.

**Major components:**
1. Design Tokens (`main.css`, `:root` and `.page-wrapper` blocks) — color, spacing, breakpoints; theme-switching via 4 CSS blocks (light default, dark OS preference, toggle-to-dark override, toggle-back-to-light override)
2. Bento Grid System (`main.css`, `.bento-grid` and `.bento-cell` variants) — CSS Grid layout with `repeat(3, 1fr)` columns and 3 responsive breakpoints
3. Project Card Component (`main.css`, `.bento-project`) — card surface, hover effects, `::after` click overlay (already implemented; must be preserved)
4. Dark Mode Toggle Architecture (checkbox `#dark-toggle` + adjacent sibling `+` combinator to `.page-wrapper`) — 4-block CSS structure handling all theme state combinations
5. Hugo Templates (`baseof.html` class change, `index.html` restructure) — minimal changes; data flow from `projects.toml` unchanged

**CSS section order within `main.css`:**
Reset → Design Tokens light defaults → Design Tokens dark OS preference → Dark Mode Toggle overrides → Base Typography → Page Layout → Bento Grid System → Bento Cell Variants → Project Card → Toggle Button → Utilities

### Critical Pitfalls

1. **Dark mode toggle breaks silently during HTML restructuring** — The `#dark-toggle:checked + .page-wrapper` CSS selector requires the checkbox to be the immediate DOM sibling of `.page-wrapper`. Any element inserted between them kills dark mode with no browser error or warning. Prevention: add a `<!-- CRITICAL: #dark-toggle must be immediately before .page-wrapper (CSS + sibling selector) -->` comment in `baseof.html` before touching HTML; test all 4 theme states after every HTML change.

2. **Border and UI contrast fails WCAG AA in both themes** — Current border contrast: light 1.24:1, dark 1.72:1 (requirement: 3:1 for UI components per WCAG 2.1 SC 1.4.11). This is an existing measured bug, not speculation. Prevention: fix `--color-border` in both themes and add `--color-card-bg` differentiation before writing any grid CSS; validate every new token pair with WCAG relative luminance formula.

3. **`--max-width: 680px` makes the grid cramped before it is even built** — At 680px, 3 grid columns are ~200px each — too narrow for readable card content. Prevention: the very first CSS change must be widening `--max-width` to 1080px; nothing else should proceed until this is confirmed.

4. **Mobile collapse destroys bento visual hierarchy** — Default `grid-template-columns: 1fr` on mobile makes all cards identical rectangles, eliminating the bento aesthetic for 60%+ of visitors. Prevention: implement the 3-breakpoint responsive system from the start; the tablet breakpoint (2-col, with the third project card spanning full width) is the key intermediate step.

5. **New color tokens must be tested in all 4 theme states** — OS-light unchecked, OS-light checked, OS-dark unchecked, OS-dark checked. Adding bento card backgrounds or shadows means adding values to all 4 CSS blocks; missing one causes a feature to break in a specific theme state only, which is easy to miss. Prevention: use `var(--color-*)` tokens exclusively — never hardcoded color values in component styles.

## Implications for Roadmap

The dependency graph across all 4 research files dictates a clear build order: tokens must precede grid CSS, grid CSS must precede template changes, template changes must precede polish. Each step produces a functional intermediate state.

### Phase 1: Foundation — Tokens and Container Width

**Rationale:** Research is unanimous that this must come first. The `--max-width: 680px` constraint is identified in PITFALLS.md (Pitfall 6), FEATURES.md (dependency graph), and ARCHITECTURE.md (Build Order Step 1) as the gating blocker. Contrast fixes must precede all visual work. This phase is non-breaking — the existing single-column layout renders correctly with updated tokens; nothing visually breaks while the tokens are being fixed.

**Delivers:** Correct WCAG AA contrast in both themes (borders pass 3:1, muted text improved to 7.5:1), proper container width for a 3-column grid, and theme architecture cleanup (`color-scheme: light dark` added, design tokens moved to `.page-wrapper` blocks only, no redundant `:root` dark block).

**Addresses:** FEATURES.md table stakes (contrast fixes, card-bg differentiation); Pitfall 3 (contrast failure); Pitfall 6 (680px constraint); Pitfall 12 (4-way theme duplication).

**Specific token changes:**
- `--max-width`: 680px → 1080px
- Add `color-scheme: light dark` to `:root`
- `--color-text-muted` light: #6b7280 → #4b5563 (ratio: 4.83:1 → 7.56:1)
- `--color-border` light: #e5e7eb → #9ca3af (ratio: 1.24:1 → 3.43:1)
- `--color-card-bg` light: #ffffff → #f8f9fa (card surface distinct from page background)
- `--color-border` dark: #374151 → #6b7280 (ratio: 1.72:1 → 3.57:1)

### Phase 2: Bento Grid CSS

**Rationale:** Write and verify grid styles in isolation before touching templates. ARCHITECTURE.md's build order explicitly sequences CSS before templates — grid styles can be validated in browser devtools before being activated in HTML. This separation allows catching CSS bugs without also debugging template changes simultaneously.

**Delivers:** Complete CSS grid system with 3 responsive breakpoints, `.bento-cell` shared card styles (border-radius, padding, background), `.bento-hero` (full-width span) and `.bento-project` variants, plus card hover effects and layered shadows.

**Uses:** CSS Grid `grid-template-columns: repeat(3, 1fr)` with `grid-column: 1/-1` for hero span; `@media (max-width: 768px)` for tablet 2-col; `@media (max-width: 480px)` for mobile 1-col; `@media (prefers-reduced-motion: reduce)` to disable all transitions.

**Avoids:** Pitfall 4 (mobile bento collapse) by implementing 3-breakpoint system from the start; Pitfall 7 (grid-template-areas maintenance burden) by using auto-placement for cards and explicit span only for the hero section.

### Phase 3: Template Restructure

**Rationale:** Templates activate the grid. Per ARCHITECTURE.md, this step follows CSS so the grid is immediately visible when HTML changes take effect. This is the highest-risk step due to Pitfall 1 (dark mode toggle) and must be preceded by two stable phases so the codebase is in a confirmed-good state.

**Delivers:** Live bento layout. `<main class="bento-grid">` in `baseof.html`. `index.html` restructured into `.bento-cell.bento-hero` (hero section) plus 3x `.bento-cell.bento-project` (project cards) as direct grid children, eliminating the `.project-cards` wrapper. Old `.project-cards` and `.project-card` CSS rules removed.

**Addresses:** Pitfall 1 (dark mode sibling selector) — add protective comment, test all 4 theme states immediately after each HTML change; Pitfall 2 (source order vs visual order) — write HTML in reading order: hero, Docora, Lumio, Helix; Pitfall 9 (card link `::after` overlay) — keep `position: relative` on `.bento-project` and test click targets at card edges.

### Phase 4: Visual Polish

**Rationale:** Can only be done after seeing the real layout at scale. Layered shadow values, hover transition amounts, border-radius final tuning, and the decision on per-project accent colors all depend on the grid being live and visible at real viewport sizes.

**Delivers:** Polished bento aesthetic — 12-16px border-radius, layered box-shadow (CSS variables, theme-aware), hover elevation (`translateY(-2px)` + shadow increase), `prefers-reduced-motion` media query. Optionally: per-project accent color on card top border.

**Addresses:** FEATURES.md differentiators — rounded corners, depth, hover tactility, motion sensitivity respect. Pitfall 10 (clamp values tuned for 680px may look undersized in hero tile at 1080px — adjust by eye).

### Phase Ordering Rationale

- Tokens precede grid because color tokens are referenced by grid component styles; fixing them first means no rework of component colors later.
- Grid CSS precedes templates because verifying CSS in isolation is faster than debugging CSS and HTML changes simultaneously; browser devtools can show the grid structure even without HTML applying the classes.
- Templates are the high-risk step (dark mode breakage) and are preceded by two stable phases where the codebase is confirmed working.
- Polish is last because estimated shadow and radius values in planning are always wrong; they must be tuned against the real layout.

### Research Flags

Phases with well-documented patterns (can skip additional research):
- **Phase 1 (Tokens):** Fully specified in STACK.md and ARCHITECTURE.md with exact token values and computed contrast ratios. Implement directly — no research needed.
- **Phase 2 (Grid CSS):** CSS Grid `grid-template-columns` is a mature, universally documented pattern. Specific breakpoints and class names are decided in research. Implement directly.
- **Phase 4 (Polish):** CSS hover transitions and box-shadow layering are established patterns; specific values are tuned by eye after seeing the live layout.

Phases requiring a careful pre-flight checklist (not research, but explicit test planning):
- **Phase 3 (Templates):** Before starting, write the test matrix: all 4 theme states (OS-light unchecked, OS-light checked, OS-dark unchecked, OS-dark checked) x 2 languages x card click targets x keyboard tab order. Execute the full matrix after each HTML change, not just at the end.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Official Hugo docs confirm Hugo Pipes patterns. CSS Grid, custom properties, and `color-scheme` property verified via MDN and caniuse with precise browser support percentages. The SCSS vs. plain CSS disagreement is a scope question, not a technical uncertainty. |
| Features | HIGH | Table stakes features are mechanical CSS changes with no ambiguity. Anti-features are derived directly from project constraints (zero-JS, 3 equal-weight projects). Contrast ratios are computed values against actual current CSS, not estimates. |
| Architecture | HIGH | Build order is derived from hard CSS dependencies (tokens → CSS → templates), not preference. The 4-block dark mode structure is the only viable CSS-only approach given the checkbox constraint. |
| Pitfalls | HIGH | Critical pitfalls verified via direct codebase analysis (lines 78-101 of `main.css`, `baseof.html` DOM structure). The dark mode sibling constraint and border contrast failures are measured facts from the existing codebase, not speculative risks. |

**Overall confidence: HIGH**

### Gaps to Address

- **SCSS vs. plain CSS disagreement:** STACK.md recommends SCSS migration; ARCHITECTURE.md recommends staying with plain CSS. Resolved in this summary in favor of plain CSS. If the CSS file grows beyond ~600 lines during implementation, revisit SCSS partials at that point.
- **Per-project accent colors:** FEATURES.md lists card accent border strips as a should-have differentiator but provides no specific color values. Defer the decision to Phase 4 — evaluate whether layered shadows alone provide sufficient card distinction before adding per-project colors.
- **`:has()` selector migration:** PITFALLS.md recommends migrating from `#dark-toggle:checked + .page-wrapper` to `body:has(#dark-toggle:checked)` to eliminate the DOM adjacency constraint and reduce the 4 CSS blocks to 2. `:has()` is Baseline Widely Available at 93.7% support. Not pursued in v1.1 to keep scope contained; revisit if Pitfall 1 actually occurs during implementation.
- **Fluid typography clamp values:** PITFALLS.md (Pitfall 10) flags that current `clamp()` values were tuned for 680px and may look undersized at 1080px. Minor issue; adjust during Phase 4 by eye.

## Sources

### Primary (HIGH confidence)
- Hugo css.Sass official docs (gohugo.io) — Dart Sass transpiler, `@use` syntax, Hugo Pipes fingerprinting
- MDN: CSS Grid grid-template-areas — named grid areas specification and usage
- W3C WCAG 2.1 SC 1.4.3 — text contrast requirement (4.5:1 normal, 3:1 large)
- W3C WCAG 2.1 SC 1.4.11 — non-text contrast requirement for UI components (3:1)
- MDN: color-scheme property — browser-level dark/light mode opt-in, 96%+ support
- MDN: prefers-color-scheme — OS theme detection
- MDN: CSS Grid Layout and Accessibility — source order vs visual order, WCAG 1.3.2
- MDN: :has() CSS selector — Baseline Widely Available status, 93.7% support
- caniuse.com: CSS Grid (97%+), color-scheme (96%+), CSS nesting (91%+), :has() (93.7%), light-dark() (85%), CSS Subgrid (87.8%)
- WebAIM Contrast Checker — WCAG relative luminance formula and ratio validation
- Codebase analysis of `assets/css/main.css` (lines 78-101, 280-287) and `layouts/_default/baseof.html` — measured contrast ratios from actual production values, DOM structure verification

### Secondary (MEDIUM confidence)
- iamsteve.me: Build a bento layout with CSS grid — responsive bento patterns, 12-column approach
- wearedevelopers.com: Building a Bento Grid Layout with Modern CSS Grid — `grid-template-columns`, responsive spans
- Josh W. Comeau: Designing Beautiful Shadows in CSS — layered shadow technique for realistic depth
- kleinfreund.de: CSS-only Dark Mode — checkbox hack deep-dive and limitations analysis
- css-tricks.com: A Complete Guide to Dark Mode on the Web — 4-block checkbox approach, color-scheme meta tag
- inkbotdesign.com / desinance.com: Bento Grid Design 2026 — 12-24px corner radius trend, gap guidance
- code.mendhak.com: The unpleasant hackiness of CSS dark mode toggles — CSS-only toggle limitations
- johnjago.com: Hugo detecting CSS changes but not rendering — fingerprint caching development friction
- css-tricks.com: Grid Content Re-ordering and Accessibility — WCAG 1.3.2 and visual vs source order

---
*Research completed: 2026-02-19*
*Ready for roadmap: yes*
