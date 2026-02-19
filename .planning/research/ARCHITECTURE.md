# Architecture: Bento Grid Redesign

**Domain:** Hugo landing page redesign -- bento grid layout, responsive behavior, dark/light theming
**Researched:** 2026-02-19
**Confidence:** HIGH

## Current Architecture Snapshot

The existing site is 501 LOC across 11 files. The key integration points for the redesign are:

```
layouts/_default/baseof.html    HTML shell with checkbox dark mode toggle
layouts/index.html              Home page template (h1 + tagline + project cards loop)
assets/css/main.css             All styles (301 lines, plain CSS, design tokens as custom properties)
data/projects.toml              3 projects with per-language descriptions
```

**Critical constraint:** The `#dark-toggle` checkbox is a **sibling** of `.page-wrapper` in the DOM. The CSS selector `#dark-toggle:checked + .page-wrapper` requires this adjacency. Any bento grid restructuring must preserve this relationship.

## Recommended Architecture for Bento Grid

### System Overview

```
BEFORE (v1.0)                          AFTER (v1.1)
=============                          ============

<body>                                 <body>
  <input#dark-toggle>                    <input#dark-toggle>
  <div.page-wrapper>                     <div.page-wrapper>
    <header>                               <header.site-header>
      [toggle] [lang-switch]                 [toggle] [lang-switch]
    </header>                              </header>
    <main>                                 <main.bento-grid>
      <section>                              <section.bento-cell.bento-hero>
        <h1> + <p.tagline>                     <h1> + <p.tagline>
      </section>                             </section>
      <section.project-cards>                <article.bento-cell.bento-project>
        <article.project-card> x3              [logo] [name] [desc] [link]
      </section>                             </article>  (x3)
    </main>                                  <section.bento-cell.bento-extra>
  </div>                                       [optional: tech stack, GitHub, etc.]
</body>                                      </section>
                                           </main>
                                         </div>
                                       </body>
```

### Component Boundaries

| Component | Responsibility | Communicates With | Status |
|-----------|---------------|-------------------|--------|
| `baseof.html` | HTML shell, dark mode checkbox, header, main block | All partials, CSS | MODIFY -- add `bento-grid` class to `<main>`, keep checkbox/page-wrapper relationship |
| `index.html` | Home page content: hero section + project cards | `baseof.html` via block, `data/projects.toml` | MODIFY -- restructure into bento cells |
| `head.html` | Meta tags, CSS link, asset pipeline | Hugo Pipes | MODIFY -- change `resources.Get` path if converting to SCSS |
| `main.css` | All visual styles | Referenced by `head.html` | REWRITE -- new grid system, revised design tokens, bento layout |
| `projects.toml` | Project metadata | Read by `index.html` template | NO CHANGE |
| `language-switcher.html` | EN/IT toggle | Called by `baseof.html` | NO CHANGE |
| `seo.html` | SEO meta tags | Called by `head.html` | NO CHANGE |

### New vs Modified Files

| File | Action | Rationale |
|------|--------|-----------|
| `assets/css/main.css` | REWRITE | New grid system, new design tokens, new bento component styles |
| `layouts/_default/baseof.html` | MODIFY | Add class to `<main>`, keep dark mode structure intact |
| `layouts/index.html` | REWRITE | Restructure content into bento grid cells |
| `layouts/partials/project-card.html` | NEW (optional) | Extract project card as a reusable partial for cleaner `index.html` |
| `data/projects.toml` | MODIFY (optional) | Add `color` or `icon` field per project for bento cell differentiation |

**No new infrastructure files needed.** The redesign is purely template + CSS. No changes to Hugo config, i18n files, CI/CD pipeline, or deployment.

## Data Flow

### Current Flow (unchanged)

```
data/projects.toml  -->  {{ range .Site.Data.projects.project }}  -->  HTML articles
i18n/*.toml         -->  {{ T "key" }}                            -->  UI strings
                         {{ index .description site.Language.Lang }}  -->  Per-language text
```

### Bento Grid Data Flow (new)

```
Project data flows into bento cells via the same range loop.
No new data sources needed.

Optional: Add per-project visual properties to data/projects.toml:

  [[project]]
  name = "Docora"
  accent = "#2563eb"      <-- Optional: per-card accent color for bento visual interest
  gridSpan = "wide"       <-- Optional: hint for CSS class (bento-cell--wide)
```

The `gridSpan` approach is NOT recommended for 3 cards. Use CSS `nth-child` or explicit grid-template-areas instead. Data files should store content, not layout concerns.

## CSS Architecture

### Approach: Keep Plain CSS (Do Not Convert to SCSS)

**Rationale:**
- The current CSS is 301 lines. Even after the bento rewrite it will be under 500 lines.
- Hugo Pipes already handles minification and fingerprinting on plain CSS.
- CSS custom properties provide the variable/token system that SCSS variables would offer.
- Native CSS nesting has 91% browser support (Chrome 120+, Firefox 117+, Safari 17.2+). If nesting is needed, native CSS nesting is sufficient.
- Adding SCSS adds a build dependency (Dart Sass is already installed in CI, but it is an unnecessary processing step for this codebase size).
- Converting to SCSS provides no benefit that plain CSS does not already cover at this scale.

**Decision:** Stay with `assets/css/main.css` as plain CSS. Use CSS custom properties for design tokens. Use native CSS nesting sparingly if it improves readability.

### CSS File Structure (within main.css)

```css
/* ============================
   1. Reset
   ============================ */

/* ============================
   2. Design Tokens (Light Mode default)
   ============================ */
:root { ... }

/* ============================
   3. Design Tokens (Dark Mode -- OS preference)
   ============================ */
@media (prefers-color-scheme: dark) { :root { ... } }

/* ============================
   4. Dark Mode Toggle Overrides
   ============================ */
#dark-toggle:checked + .page-wrapper { ... }
@media (prefers-color-scheme: dark) {
  #dark-toggle:checked + .page-wrapper { ... }
}

/* ============================
   5. Base Typography & Elements
   ============================ */

/* ============================
   6. Page Layout (page-wrapper, site-header)
   ============================ */

/* ============================
   7. Bento Grid System
   ============================ */
.bento-grid { ... }
.bento-cell { ... }

/* ============================
   8. Bento Cell Variants
   ============================ */
.bento-hero { ... }
.bento-project { ... }
.bento-extra { ... }

/* ============================
   9. Project Card Component
   ============================ */

/* ============================
   10. Dark Mode Toggle Button
   ============================ */

/* ============================
   11. Utilities (sr-only, etc.)
   ============================ */
```

### Bento Grid CSS Pattern

Use CSS Grid with `grid-template-areas` for explicit, readable layout control. This is the best approach for a fixed number of known cells (hero + 3 project cards + optional extras).

```css
/* ============================
   Bento Grid System
   ============================ */

/* Desktop: 3 columns */
.bento-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
  gap: var(--space-md);
  padding: var(--space-md) 0;
}

/* Hero spans full width */
.bento-hero {
  grid-column: 1 / -1;
}

/* Each project card occupies one column */
.bento-project {
  /* No explicit grid placement needed -- auto-placement fills columns */
}

/* Optional extra section spans full width or partial */
.bento-extra {
  grid-column: 1 / -1;
}

/* Tablet: 2 columns */
@media (max-width: 768px) {
  .bento-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  /* Third project card spans full width on its own row */
  .bento-project:nth-child(5) {
    grid-column: 1 / -1;
  }
}

/* Mobile: 1 column */
@media (max-width: 480px) {
  .bento-grid {
    grid-template-columns: 1fr;
  }
}
```

**Why `grid-template-columns` over `grid-template-areas`:** For 3 equal project cards with a full-width hero, `repeat(3, 1fr)` with span overrides is simpler and more maintainable than named areas. Named areas become worthwhile only when cells have asymmetric sizes (e.g., one card spans 2 columns while another spans 1).

**Alternative for visual variety:** If the design calls for mixed-size bento cells (e.g., one large featured project + two smaller ones), use:

```css
.bento-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-areas:
    "hero    hero    hero"
    "proj-1  proj-1  proj-2"
    "proj-3  extra   extra";
  gap: var(--space-md);
}

.bento-hero    { grid-area: hero; }
.bento-project:nth-child(2) { grid-area: proj-1; }
.bento-project:nth-child(3) { grid-area: proj-2; }
.bento-project:nth-child(4) { grid-area: proj-3; }
.bento-extra   { grid-area: extra; }
```

This is more "bento" in feel (uneven cells create visual interest) but requires careful responsive handling.

## Dark/Light Mode Architecture

### Current Problem

The existing dark mode toggle works correctly but has architectural issues:

1. **Design tokens are duplicated 4 times** -- light defaults in `:root`, dark in `@media (prefers-color-scheme: dark) :root`, dark in `#dark-toggle:checked + .page-wrapper`, and light-override in `@media (prefers-color-scheme: dark) { #dark-toggle:checked + .page-wrapper }`. The `.page-wrapper` selector inside the media query is also redundant (it duplicates `:root` values already set in the same block).

2. **Muted text contrast is borderline** -- `#6b7280` on `#ffffff` gives a contrast ratio of 4.83:1. While this technically passes WCAG AA (minimum 4.5:1), it offers no safety margin. The PROJECT.md mentions "Fix light mode contrast" as an active requirement.

3. **No CSS `color-scheme` declaration** -- The browser does not know the page is dark-aware, so native form elements, scrollbars, and user-agent styles do not adapt.

### Recommended Dark/Light Architecture

**Strategy:** Keep the CSS-only checkbox toggle (no JavaScript). Improve the token system to reduce duplication. Add `color-scheme` declaration. Strengthen contrast ratios.

#### Step 1: Add `color-scheme` to `:root`

```css
:root {
  color-scheme: light dark;
}
```

This tells the browser the page supports both schemes. Native controls (scrollbars, form inputs, `<select>` elements) will adapt to match. This single line provides significant visual polish for free.

#### Step 2: Consolidate Design Tokens (eliminate duplication)

The current CSS defines the same dark palette in two places: `@media (prefers-color-scheme: dark) :root` AND `@media (prefers-color-scheme: dark) .page-wrapper`. The `.page-wrapper` duplication exists because the checkbox toggle targets `.page-wrapper` not `:root`.

**Root cause:** The checkbox selector `#dark-toggle:checked + .page-wrapper` can only set properties on `.page-wrapper` and its descendants. It cannot modify `:root`. Therefore, all color references must resolve on `.page-wrapper`, not `:root`.

**Revised approach:**

```css
/* Light tokens (default) */
:root {
  color-scheme: light dark;
  /* Non-theme tokens (spacing, typography) */
  --space-xs: 0.5rem;
  /* ... */
}

/* Light theme colors on page-wrapper */
.page-wrapper {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-text-muted: #4b5563;  /* Darker than current #6b7280 for better contrast */
  --color-border: #e5e7eb;
  --color-link: #2563eb;
  --color-link-hover: #1d4ed8;
  --color-card-bg: #f9fafb;     /* Subtle card lift vs page bg */
}

/* Dark theme: OS preference */
@media (prefers-color-scheme: dark) {
  .page-wrapper {
    --color-bg: #111827;
    --color-text: #f3f4f6;
    --color-text-muted: #d1d5db;  /* Brighter than current #9ca3af */
    --color-border: #374151;
    --color-link: #60a5fa;
    --color-link-hover: #93bbfd;
    --color-card-bg: #1f2937;
  }
}

/* Toggle: override theme (light OS -> dark) */
#dark-toggle:checked + .page-wrapper {
  --color-bg: #111827;
  --color-text: #f3f4f6;
  --color-text-muted: #d1d5db;
  --color-border: #374151;
  --color-link: #60a5fa;
  --color-link-hover: #93bbfd;
  --color-card-bg: #1f2937;
}

/* Toggle: override theme (dark OS -> light) */
@media (prefers-color-scheme: dark) {
  #dark-toggle:checked + .page-wrapper {
    --color-bg: #ffffff;
    --color-text: #1a1a1a;
    --color-text-muted: #4b5563;
    --color-border: #e5e7eb;
    --color-link: #2563eb;
    --color-link-hover: #1d4ed8;
    --color-card-bg: #f9fafb;
  }
}
```

This still has 4 blocks (inherent to the CSS-only checkbox approach), but each block now defines tokens on `.page-wrapper` only, eliminating the redundant `:root` dark block. The `@media (prefers-color-scheme: dark) :root` block is removed entirely.

**Note:** The `light-dark()` CSS function (87% browser support) could reduce this to 2 blocks but CANNOT work with the checkbox toggle. `light-dark()` responds to `color-scheme` property changes, which would require JavaScript to toggle. Since the project constraint is zero JS, the checkbox + 4-block approach remains the correct architecture.

#### Step 3: Strengthen Contrast Ratios

Current vs proposed muted text colors:

| Token | Current | Proposed | Contrast (light bg) | Contrast (dark bg) |
|-------|---------|----------|---------------------|--------------------|
| `--color-text-muted` (light) | `#6b7280` | `#4b5563` | 4.83:1 -> 7.41:1 | n/a |
| `--color-text-muted` (dark) | `#9ca3af` | `#d1d5db` | n/a | 6.99:1 -> 11.55:1 |

Both proposed values exceed WCAG AA (4.5:1) and approach AAA (7:1).

#### Step 4: Bento Card Surfaces

For bento grid layouts, cards need visual distinction from the background. The current design uses identical card-bg and page-bg in light mode (`#ffffff` both). This works for bordered cards but fails for a bento aesthetic where cards should feel like distinct surfaces.

**Recommendation:**
- Light mode: `--color-card-bg: #f9fafb` (very subtle gray lift) or keep `#ffffff` with border emphasis
- Dark mode: `--color-card-bg: #1f2937` (already distinct from `#111827` background) -- this is good
- Add `--color-card-border: var(--color-border)` for explicit border control
- Consider `border-radius: 1rem` for bento cells (larger radius = more bento feel)

## Template Architecture

### baseof.html Changes

```html
<!-- BEFORE -->
<main>
  {{ block "main" . }}{{ end }}
</main>

<!-- AFTER -->
<main class="bento-grid">
  {{ block "main" . }}{{ end }}
</main>
```

Minimal change. The `bento-grid` class on `<main>` activates the CSS grid. The block structure remains identical.

**Alternative:** Keep `<main>` clean and add a wrapper `<div class="bento-grid">` inside the block in `index.html`. This is better if other page types (404, future pages) should NOT use the bento grid. Since this is a single-page site, putting it on `<main>` is simpler.

### index.html Restructure

```html
{{ define "main" }}
<section class="bento-cell bento-hero">
  <h1 class="page-title">{{ .Title }}</h1>
  <p class="tagline">{{ T "tagline" }}</p>
</section>

{{ range .Site.Data.projects.project }}
<article class="bento-cell bento-project">
  {{ if .logo }}
  <img src="{{ .logo }}" alt="" class="project-logo" loading="lazy" width="48" height="48">
  {{ end }}
  <h2>{{ .name }}</h2>
  <p>{{ index .description site.Language.Lang }}</p>
  <a href="{{ .url }}" class="card-link">
    <span class="sr-only">{{ T "viewProject" }}: </span>{{ .name }}
  </a>
</article>
{{ end }}
{{ end }}
```

**Key changes:**
1. Removed the wrapping `<section class="project-cards">` -- bento cells are now direct children of the grid container (`<main class="bento-grid">`).
2. Each `<article>` gets `bento-cell bento-project` classes.
3. Hero section gets `bento-cell bento-hero` classes.
4. The `project-card` class is renamed to `bento-project` for consistency.

### Optional: Extract Project Card Partial

For a 3-project site, extracting a partial is optional but improves readability if project cards become complex (e.g., adding status badges, tech tags):

```html
<!-- layouts/partials/project-card.html -->
<!-- Receives a project data object as context -->
<article class="bento-cell bento-project">
  {{ if .logo }}
  <img src="{{ .logo }}" alt="" class="project-logo" loading="lazy" width="48" height="48">
  {{ end }}
  <h2>{{ .name }}</h2>
  <p>{{ index .description site.Language.Lang }}</p>
  <a href="{{ .url }}" class="card-link">
    <span class="sr-only">{{ T "viewProject" }}: </span>{{ .name }}
  </a>
</article>
```

Called as:
```html
{{ range .Site.Data.projects.project }}
  {{ partial "project-card.html" . }}
{{ end }}
```

**Recommendation:** Do NOT extract a partial for v1.1. The template is 10 lines. Extract only if card complexity grows in a future milestone.

## Responsive Breakpoint Strategy

### Recommended Breakpoints

| Breakpoint | Columns | Layout Description |
|------------|---------|-------------------|
| >= 769px (desktop) | 3 columns | Hero spans 3. Each project card in its own column. |
| 481px - 768px (tablet) | 2 columns | Hero spans 2. Two projects side by side, third spans full width. |
| <= 480px (mobile) | 1 column | Everything stacks vertically. |

### Why These Breakpoints

- **768px** is the standard tablet breakpoint (iPad portrait). 3 project cards at 768px would be ~230px each after gap, which is too narrow for readable card content.
- **480px** captures most phones in portrait. Below this, 2 columns would mean ~220px per card, which is too cramped.
- The `--max-width: 680px` constraint in the current CSS MUST be widened. A 3-column bento grid at 680px gives ~210px per column, which is too narrow. **Change `--max-width` to `960px` or `1080px`** for the bento layout to breathe.

### Page Wrapper Width Change

```css
/* BEFORE */
:root {
  --max-width: 680px;
}

/* AFTER */
:root {
  --max-width: 1080px;  /* or 960px for tighter feel */
}
```

This is a critical layout change. The current 680px max-width was designed for a single-column reading experience. The bento grid needs horizontal space for 3 columns. 1080px provides ~340px per column at desktop, which is comfortable for card content.

## Patterns to Follow

### Pattern 1: BEM-lite Class Naming for Bento

**What:** Use `bento-` prefix for all grid-related classes. Avoid deep nesting.

**Classes:**
```
.bento-grid      (the grid container)
.bento-cell      (shared cell styles: border-radius, padding, background)
.bento-hero      (hero cell variant)
.bento-project   (project card cell variant)
.bento-extra     (optional extra sections)
```

**Why:** Flat selectors are fast, readable, and easy to override. Avoid `.bento-grid > .bento-cell > .bento-cell__content` type nesting -- this is a small site, not a design system.

### Pattern 2: Full-Bleed Page Wrapper with Grid Content

**What:** The `.page-wrapper` provides full-width background color while `.bento-grid` constrains content width.

```css
.page-wrapper {
  /* Remove max-width here -- let it fill the viewport */
  background: var(--color-bg);
  color: var(--color-text);
  min-height: 100vh;
  padding: var(--space-lg) var(--space-md);
}

.bento-grid {
  max-width: var(--max-width);
  margin: 0 auto;
  display: grid;
  /* ... */
}
```

**Why:** On wide screens, the background color should extend edge-to-edge while the bento content stays centered. Moving `max-width` from `.page-wrapper` to `.bento-grid` (or keeping it on `.page-wrapper` but widening it) achieves this.

### Pattern 3: Clickable Card with Overlay Link

**What:** The current `card-link::after` with `position: absolute; inset: 0;` overlay is already implemented. Keep it for the bento redesign.

**Why:** Makes the entire card clickable without JavaScript. Requires `position: relative` on the card container.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Using JavaScript for the Grid Layout

**What:** Adding a JS library (Masonry, Isotope) for the bento grid.

**Why bad:** Violates the zero-JS constraint. CSS Grid handles this layout natively. JS-based grids are for dynamic, unknown-size content (photo galleries). This site has exactly 3 known project cards.

**Instead:** Pure CSS Grid with `grid-template-columns` and media queries.

### Anti-Pattern 2: Container Queries for This Use Case

**What:** Using `@container` queries instead of `@media` queries for responsive bento cells.

**Why bad for this project:** Container queries are useful when components appear in different container contexts (sidebar vs main content). This site has one context: the full page. Container queries would add complexity for zero benefit. Browser support is also lower (88%) than media queries (100%).

**Instead:** Standard `@media` breakpoints. Simple, universal, sufficient.

### Anti-Pattern 3: Converting to SCSS for "Organization"

**What:** Converting `main.css` to `main.scss` to use variables, nesting, or partials.

**Why bad for this project:** The CSS is under 500 lines. CSS custom properties already provide variables. Native CSS nesting (91% support) provides nesting. SCSS partials/imports add file management overhead disproportionate to this codebase size. The Dart Sass build step is already installed in CI but would be an unnecessary processing step.

**Instead:** Stay with plain CSS. Use section comments for organization. Use custom properties for tokens.

### Anti-Pattern 4: Animated Grid Transitions

**What:** Adding `transition` on `grid-template-columns` or animating grid item positions on resize.

**Why bad:** CSS grid transitions have inconsistent browser support. On mobile, they cause layout thrashing during scroll/resize. They serve no functional purpose on a landing page.

**Instead:** Instant layout shifts on breakpoint changes. Use subtle transitions only on hover states (border-color, box-shadow).

## Build Order for the Redesign

Dependencies flow top-to-bottom. Each step produces a functional intermediate state.

```
Step 1: Design Tokens Update
  Modify: assets/css/main.css (tokens section only)
  - Widen --max-width to 1080px
  - Add color-scheme: light dark
  - Strengthen --color-text-muted values
  - Add --color-card-bg differentiation
  - Update dark mode token blocks
  Result: Existing layout still works, just with better contrast

Step 2: Bento Grid CSS
  Modify: assets/css/main.css (add grid section)
  - Add .bento-grid display:grid rules
  - Add .bento-cell shared card styles
  - Add .bento-hero, .bento-project, .bento-extra variants
  - Add responsive breakpoints (768px, 480px)
  Result: Grid styles ready, but not yet applied (no HTML changes)

Step 3: Template Restructure
  Modify: layouts/_default/baseof.html (add class to <main>)
  Modify: layouts/index.html (restructure into bento cells)
  Result: Bento layout is live. Old .project-cards styles can be removed.

Step 4: Polish
  Modify: assets/css/main.css (remove old styles, refine)
  - Remove .project-cards, .project-card class definitions
  - Fine-tune card padding, border-radius, hover effects
  - Verify dark mode toggle still works end-to-end
  Result: Clean, final bento grid with proper theming

Step 5: Optional Extras
  Modify: layouts/index.html (add extra bento sections)
  Modify: assets/css/main.css (style extra sections)
  - Add tech stack section, GitHub link, or other bento cells
  - Only if design warrants -- do not add for the sake of filling space
  Result: Complete bento grid with optional supplementary content
```

**Step ordering rationale:**
- Tokens first because they are non-breaking (the old layout renders fine with updated tokens).
- CSS before templates because you can write and verify grid styles in isolation using browser devtools before committing HTML changes.
- Template changes third because they activate the grid. Doing templates before CSS would break the layout.
- Polish last because cleanup requires seeing the final result.

## Integration Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Dark mode toggle breaks after restructure | HIGH -- toggle is the only interactive element | Test `#dark-toggle:checked + .page-wrapper` selector chain in every step. The checkbox must remain a direct sibling of `.page-wrapper`. |
| Cards too narrow at 3-column desktop | MEDIUM -- readability degrades | Increase `--max-width` from 680px to 1080px. Test at 1024px viewport. |
| Logo images break bento card layout | LOW -- logos are external URLs | Current logos are 48x48 SVGs with `object-fit: contain`. These will work in any card size. |
| Responsive breakpoints feel wrong | LOW -- subjective | Test at common device widths (375, 414, 768, 1024, 1440). Adjust breakpoints based on content, not arbitrary device sizes. |

## Sources

- [CSS Grid bento layout patterns (iamsteve.me)](https://iamsteve.me/blog/bento-layout-css-grid) -- grid-template-columns, auto-flow dense, responsive spans (MEDIUM confidence)
- [Bento box CSS Grid tutorial (codemotion.com)](https://www.codemotion.com/magazine/frontend/lets-create-a-bento-box-design-layout-using-modern-css/) -- grid-template-areas approach, aspect-ratio patterns (MEDIUM confidence)
- [CSS light-dark() function (css-tricks.com)](https://css-tricks.com/come-to-the-light-dark-side/) -- 87% browser support, requires color-scheme property, cannot work with checkbox toggle (MEDIUM confidence)
- [light-dark() browser support (caniuse.com)](https://caniuse.com/mdn-css_types_color_light-dark) -- 84.93% global support, Chrome 123+, Firefox 120+, Safari 17.5+ (HIGH confidence)
- [CSS Nesting browser support (caniuse.com)](https://caniuse.com/css-nesting) -- 91% global support (HIGH confidence)
- [WCAG 2.2 contrast requirements (makethingsaccessible.com)](https://www.makethingsaccessible.com/guides/contrast-requirements-for-wcag-2-2-level-aa/) -- 4.5:1 normal text, 3:1 large text (HIGH confidence)
- [Hugo Pipes asset processing (gohugo.io)](https://gohugo.io/hugo-pipes/introduction/) -- resources.Get, Minify, Fingerprint (HIGH confidence)
- [Hugo SCSS transpilation (gohugo.io)](https://gohugo.io/hugo-pipes/transpile-sass-to-css/) -- Dart Sass integration, toCSS function (HIGH confidence)
- [prefers-color-scheme (MDN)](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-color-scheme) -- OS-level dark mode detection (HIGH confidence)
- [Dark mode complete guide (css-tricks.com)](https://css-tricks.com/a-complete-guide-to-dark-mode-on-the-web/) -- checkbox toggle approach, color-scheme meta tag (MEDIUM confidence)
- [WeAreDevs bento grid tutorial](https://www.wearedevelopers.com/en/magazine/682/building-a-bento-grid-layout-with-modern-css-grid-682) -- Modern CSS grid bento patterns (MEDIUM confidence)

---
*Architecture research for: v1.1 Bento Grid Redesign (toto-castaldi.com)*
*Researched: 2026-02-19*
