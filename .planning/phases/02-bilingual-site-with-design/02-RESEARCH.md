# Phase 2: Bilingual Site with Design - Research

**Researched:** 2026-02-18
**Domain:** Hugo multilingual templating, CSS design systems, accessibility
**Confidence:** HIGH

## Summary

Phase 2 transforms the placeholder Hugo site from Phase 1 into a complete bilingual landing page with project cards, language switching, dark mode, and a modular typography system. The existing Phase 1 codebase provides a solid foundation: Hugo 0.155.3 with multilingual config (`defaultContentLanguageInSubdir = true`), CSS asset pipeline with fingerprinting, and a `baseof.html` / `index.html` template structure.

The implementation requires work across four domains: (1) Hugo data templates for project cards with per-language content, (2) Hugo i18n for UI string translation and a language switcher, (3) a CSS design system with custom properties for dark mode and modular typography, and (4) an SVG favicon with embedded dark mode support. All of this must be achieved with zero JavaScript, pure CSS, and WCAG AA contrast compliance.

**Primary recommendation:** Build the entire design system in a single `assets/css/main.css` file using CSS custom properties for colors and type scale, with `prefers-color-scheme` media query for dark mode. Use Hugo's `data/projects.toml` with nested per-language description keys accessed via `index` function. Use `.Translations` for the language switcher partial.

<user_constraints>

## User Constraints (from CONTEXT.md)

### Locked Decisions
- Each card shows: official project logo (from each product's site), project name, bilingual description, link to subdomain
- Short tagline above the project cards (e.g., "Software projects by Toto Castaldi" -- bilingual)
- Tagline feels like a quiet subtitle -- smaller, lighter weight, understated; cards are the focus
- No footer -- page ends after the project cards
- Airy and spacious density -- generous line-height, large margins, content breathes
- Project names on cards use the same font as body text, just bold and larger -- hierarchy through size and weight, not a different typeface
- Modular typography scale with CSS custom properties (per DSGN-03 requirement)
- Language switcher positioned in top-right corner of the page

### Claude's Discretion
- Card visual containment style (bordered, elevated, borderless)
- Card link interaction pattern (whole card clickable vs explicit link)
- Page structure (centered column vs card grid)
- Color palette for light and dark modes
- Font choice (system stack vs specific web font)
- Language switcher appearance and behavior details
- Favicon design
- Dark mode color strategy (CSS prefers-color-scheme, no JS)

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope

</user_constraints>

<phase_requirements>

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| CONT-01 | Home page with project cards showing name, short description, and link for each project | Hugo data templates with `range .Site.Data.projects`, card HTML structure in `index.html` |
| CONT-02 | Three projects displayed: Docora, Lumio, Helix with subdomain URLs | `data/projects.toml` with `[[project]]` array entries containing name, url, and per-language descriptions |
| CONT-03 | Project data stored in `data/projects.toml` with per-language fields | TOML array-of-tables syntax with nested `[project.description]` map keyed by language code; accessed via `index .description site.Language.Lang` |
| CONT-04 | Proper `<title>` and meta description per language | Front matter `description` field in `_index.it.md` / `_index.en.md`; template renders `<meta name="description">` |
| CONT-05 | i18n string files for UI labels in Italian and English | `i18n/it.toml` and `i18n/en.toml` with key-value pairs; accessed via `{{ T "key" }}` or `{{ i18n "key" }}` in templates |
| CONT-06 | Favicon (SVG with dark mode support) | Inline `<style>` block inside SVG with `@media (prefers-color-scheme: dark)` to swap fill colors |
| UI-01 | Text-based language switcher (IT/EN) in header | Hugo `.Translations` method on Page; partial template in `layouts/partials/language-switcher.html` |
| UI-02 | Responsive layout that stacks project cards vertically on mobile | CSS Grid or Flexbox with media query breakpoint; cards stack by default (mobile-first) |
| UI-03 | Semantic HTML with proper heading hierarchy and `lang` attribute | `<html lang="{{ .Site.Language.Lang }}">` already in baseof.html; add `<header>`, `<nav>`, `<main>`, `<section>` elements |
| UI-04 | Zero JavaScript -- pure HTML + CSS | All interactivity via CSS only; dark mode via `prefers-color-scheme`; language switch is plain `<a>` link |
| UI-05 | Page load under 1 second | System font stack (no web font downloads), single minified+fingerprinted CSS file, SVG favicon inline, no JS |
| DSGN-01 | Minimal clean design with whitespace-heavy layout and strong typography | Large `padding`/`margin` values, generous `line-height` (1.6+), restrained color palette |
| DSGN-02 | Dark mode via CSS `prefers-color-scheme` (no JS toggle) | CSS custom properties on `:root` overridden inside `@media (prefers-color-scheme: dark)` block |
| DSGN-03 | Modular typography scale with CSS custom properties | `--font-size-sm` through `--font-size-xxxl` custom properties using `clamp()` for fluid scaling |
| DSGN-04 | WCAG AA contrast ratios (4.5:1 body text, 3:1 large text) | Verified color pairs for both light and dark modes; use WebAIM contrast checker tool |

</phase_requirements>

## Standard Stack

### Core (already in place from Phase 1)
| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| Hugo | 0.155.3 (extended) | Static site generator | Installed, configured |
| Hugo Asset Pipeline | built-in | CSS minification + fingerprinting | Already wired in `head.html` |
| Hugo i18n | built-in | Translation strings | Files exist but minimal |
| GitHub Actions | v5 actions | CI/CD to GitHub Pages | Working |

### New for Phase 2 (no new dependencies)
| Feature | Implementation | Purpose |
|---------|---------------|---------|
| Data templates | `data/projects.toml` | Project card content |
| CSS custom properties | `assets/css/main.css` | Design tokens (colors, type scale) |
| `prefers-color-scheme` | CSS media query | Automatic dark mode |
| SVG favicon | `static/favicon.svg` | Dark-mode-aware site icon |

### Alternatives Considered
| Instead of | Could Use | Why Not |
|------------|-----------|---------|
| Single `data/projects.toml` | Separate `data/en/projects.toml` + `data/it/projects.toml` | CONT-03 specifies single file; nested language keys are cleaner for 3 items |
| `@media (prefers-color-scheme)` | CSS `light-dark()` function | `light-dark()` has 85% browser support (Baseline 2024); media query approach has near-universal support and is more explicit |
| System font stack | Google Fonts / web font | System fonts = zero network requests, matches UI-05 (sub-1s load), fits minimal aesthetic |
| `clamp()` fluid type | Utopia full fluid scale | Utopia is complex; simple `clamp()` values meet DSGN-03 and are easier to maintain |

**Installation:** No new packages needed. Everything is built into Hugo and CSS.

## Architecture Patterns

### Project Structure (files to create/modify)
```
toto-castaldi/
├── assets/
│   └── css/
│       └── main.css              # MODIFY: full design system
├── config/
│   └── _default/
│       ├── hugo.toml             # MODIFY: add params (description)
│       └── languages.toml        # MODIFY: add per-language params
├── content/
│   ├── _index.en.md              # MODIFY: add description front matter
│   └── _index.it.md              # MODIFY: add description front matter
├── data/
│   └── projects.toml             # CREATE: project data
├── i18n/
│   ├── en.toml                   # MODIFY: add UI string translations
│   └── it.toml                   # MODIFY: add UI string translations
├── layouts/
│   ├── _default/
│   │   └── baseof.html           # MODIFY: add header, nav, favicon link
│   ├── index.html                # MODIFY: project cards template
│   └── partials/
│       ├── head.html             # MODIFY: meta description, favicon
│       └── language-switcher.html # CREATE: language nav partial
└── static/
    └── favicon.svg               # CREATE: SVG with dark mode
```

### Pattern 1: Hugo Data Template with Per-Language Fields

**What:** Store project data in a single TOML file with language-keyed description maps.
**When to use:** When content is small, structured, and shared across languages (only descriptions differ).

```toml
# data/projects.toml
[[project]]
name = "Docora"
url = "https://docora.toto-castaldi.com"
logo = "https://docora.toto-castaldi.com/images/logo.svg"
[project.description]
it = "Descrizione del progetto Docora"
en = "Docora project description"

[[project]]
name = "Lumio"
url = "https://lumio.toto-castaldi.com"
logo = ""
[project.description]
it = "Descrizione del progetto Lumio"
en = "Lumio project description"
```

**Template access pattern:**
```html
<!-- Source: Hugo docs - data templates + index function -->
{{ range .Site.Data.projects.project }}
<article class="project-card">
  {{ if .logo }}
  <img src="{{ .logo }}" alt="{{ .name }}" class="project-logo">
  {{ end }}
  <h2>{{ .name }}</h2>
  <p>{{ index .description site.Language.Lang }}</p>
  <a href="{{ .url }}">{{ .url }}</a>
</article>
{{ end }}
```

### Pattern 2: Hugo Language Switcher Partial

**What:** A template partial that renders links to all translations of the current page.
**When to use:** Any page that needs language navigation.

```html
<!-- layouts/partials/language-switcher.html -->
<!-- Source: Hugo docs - multilingual mode -->
<nav aria-label="{{ T "languageSwitch" }}">
  {{ range .Translations }}
  <a href="{{ .RelPermalink }}" lang="{{ .Language.Lang }}"
     hreflang="{{ .Language.Lang }}">
    {{ .Language.LanguageName }}
  </a>
  {{ end }}
</nav>
```

Note: `.Translations` returns only the OTHER languages (excludes current). For a switcher that shows only the alternate language, this is ideal. For showing all languages including current, use `.AllTranslations` with a conditional.

### Pattern 3: CSS Design Tokens with Dark Mode

**What:** CSS custom properties for all colors and type sizes, overridden for dark mode.
**When to use:** Any design system that needs theming or dark mode.

```css
/* Source: MDN prefers-color-scheme + modular type scale best practices */
:root {
  /* Colors - Light mode (default) */
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-text-muted: #6b7280;
  --color-border: #e5e7eb;
  --color-link: #2563eb;
  --color-link-hover: #1d4ed8;

  /* Type scale using clamp() for fluid sizing */
  --font-size-sm: clamp(0.83rem, 0.44vw + 0.72rem, 1rem);
  --font-size-base: clamp(1rem, 0.34vw + 0.91rem, 1.2rem);
  --font-size-md: clamp(1.2rem, 0.41vw + 1.09rem, 1.44rem);
  --font-size-lg: clamp(1.44rem, 0.49vw + 1.31rem, 1.73rem);
  --font-size-xl: clamp(1.73rem, 0.59vw + 1.57rem, 2.07rem);

  /* Spacing */
  --space-xs: 0.5rem;
  --space-sm: 1rem;
  --space-md: 1.5rem;
  --space-lg: 2.5rem;
  --space-xl: 4rem;
  --space-xxl: 6rem;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #111827;
    --color-text: #f3f4f6;
    --color-text-muted: #9ca3af;
    --color-border: #374151;
    --color-link: #60a5fa;
    --color-link-hover: #93bbfd;
  }
}
```

### Pattern 4: SVG Favicon with Dark Mode

**What:** SVG file with embedded CSS media query for dark mode color swap.
**When to use:** Any site with dark mode support.

```svg
<!-- static/favicon.svg -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <style>
    .fg { fill: #1a1a1a; }
    @media (prefers-color-scheme: dark) {
      .fg { fill: #f3f4f6; }
    }
  </style>
  <text class="fg" x="50%" y="50%" dominant-baseline="central"
        text-anchor="middle" font-size="24" font-weight="bold">T</text>
</svg>
```

HTML reference in `head.html`:
```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
```

### Anti-Patterns to Avoid
- **JavaScript for dark mode:** Requirement UI-04 mandates zero JS. Use only `prefers-color-scheme` CSS media query.
- **Separate CSS files per mode:** Use custom properties on `:root` and override in media query -- single file, single request.
- **Hardcoded colors in templates:** All colors should reference CSS custom properties, never inline styles or hardcoded hex values.
- **Skipping heading levels:** Go h1 -> h2 -> h3 sequentially. The page title is h1, project names are h2.
- **Using `relURL` instead of `relLangURL`:** In multilingual Hugo, always use `.RelPermalink` or `relLangURL` to ensure language prefix is included.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Translation strings | Custom lookup maps in templates | Hugo i18n (`{{ T "key" }}`) | Built-in, handles plurals, RFC 5646 compliant |
| Language switcher URLs | Manual URL construction | `.Translations` / `.AllTranslations` | Hugo manages language path prefixes automatically |
| CSS minification | External build tool | `resources.Minify` (already in head.html) | Built into Hugo pipes, zero config |
| Asset cache busting | Manual version strings | `resources.Fingerprint` (already in head.html) | Built into Hugo pipes, generates content hash |
| Responsive images | Custom media query sets for logos | `<img>` with `max-width: 100%` | External logos from subdomains; no Hugo image processing available |

**Key insight:** Hugo's built-in i18n, data templates, and asset pipeline handle every requirement without external dependencies. The entire build produces static HTML + CSS with zero JavaScript.

## Common Pitfalls

### Pitfall 1: TOML Array-of-Tables Syntax
**What goes wrong:** Using `[projects]` (single bracket) instead of `[[project]]` (double bracket) for arrays in TOML. Single brackets create a map, double brackets create an array of maps.
**Why it happens:** TOML's array-of-tables syntax is unusual compared to JSON/YAML arrays.
**How to avoid:** Always use `[[project]]` for each project entry. Test with `hugo server` and check for template errors.
**Warning signs:** Template renders nothing or shows raw map structure instead of iterating.

### Pitfall 2: Data Template Key Casing
**What goes wrong:** Hugo lowercases TOML keys when loading data files. A field `logoURL` in TOML becomes `logourl` in the template.
**Why it happens:** Hugo normalizes data keys to lowercase.
**How to avoid:** Use lowercase or snake_case keys in TOML data files (`logo_url` not `logoURL`). Access them the same way in templates.
**Warning signs:** Template values are empty despite data file being correct.

### Pitfall 3: Language Switcher Links to Home Instead of Current Page
**What goes wrong:** Using `.Site.Home.AllTranslations` links to the homepage of each language, not the current page's translation.
**Why it happens:** Copying the basic Hugo docs example which uses `$.Site.Home.AllTranslations`.
**How to avoid:** Use `.Translations` (on the current page context) not `.Site.Home.AllTranslations`.
**Warning signs:** Clicking language switcher always navigates to homepage.

### Pitfall 4: Dark Mode Contrast Failures
**What goes wrong:** Colors that pass WCAG AA in light mode fail in dark mode (or vice versa).
**Why it happens:** Dark mode colors are often an afterthought. Inverting colors does not guarantee equivalent contrast ratios.
**How to avoid:** Check contrast ratios independently for BOTH color schemes. Use WebAIM contrast checker (https://webaim.org/resources/contrastchecker/) for each color pair.
**Warning signs:** Text becomes hard to read in one mode.

### Pitfall 5: Missing `lang` Attribute on `<html>` Element
**What goes wrong:** Screen readers use wrong pronunciation rules for the content language.
**Why it happens:** Not using Hugo's language data in the template.
**How to avoid:** Already correct in Phase 1's `baseof.html`: `<html lang="{{ .Site.Language.Lang }}">`. Verify it stays during modifications.
**Warning signs:** Screen reader mispronounces Italian/English content.

### Pitfall 6: External Logo Images Blocking Page Load
**What goes wrong:** Loading logos from external subdomains (docora.toto-castaldi.com, etc.) adds DNS lookups and network requests, potentially exceeding the 1-second load target.
**Why it happens:** External resources are outside our control.
**How to avoid:** Consider: (a) copy logos into `static/images/` to serve from same domain, or (b) use `loading="lazy"` attribute, or (c) set explicit `width`/`height` to prevent layout shift. Option (a) is most reliable for load time.
**Warning signs:** Lighthouse shows external resource blocking render.

### Pitfall 7: Hugo Data Template Access Path
**What goes wrong:** Accessing `site.Data.projects` when the file is `data/projects.toml` and it contains `[[project]]` arrays -- the top-level key is `project` not `projects`.
**Why it happens:** Confusion between filename (projects.toml) and TOML key name ([[project]]).
**How to avoid:** The access path is `.Site.Data.projects.project` -- `projects` is the filename (without extension), `project` is the TOML key.
**Warning signs:** Range produces no output or errors.

## Code Examples

Verified patterns from official sources:

### Hugo i18n String Translation
```toml
# i18n/en.toml
# Source: https://gohugo.io/functions/lang/translate/
[siteTitle]
other = "Antonio Toto Castaldi"

[tagline]
other = "Software projects by Toto Castaldi"

[languageSwitch]
other = "Language"

[viewProject]
other = "Visit project"
```

```toml
# i18n/it.toml
[siteTitle]
other = "Antonio Toto Castaldi"

[tagline]
other = "Progetti software di Toto Castaldi"

[languageSwitch]
other = "Lingua"

[viewProject]
other = "Visita il progetto"
```

### Hugo Template: Meta Description
```html
<!-- layouts/partials/head.html -->
<!-- Source: Hugo docs - front matter + multilingual -->
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{ .Title }}</title>
{{ with .Description }}
<meta name="description" content="{{ . }}">
{{ end }}
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```

### Content Front Matter with Description
```markdown
---
# content/_index.en.md
title: "Antonio Toto Castaldi"
description: "Software projects by Toto Castaldi: Docora, Lumio, and Helix"
---
```

```markdown
---
# content/_index.it.md
title: "Antonio Toto Castaldi"
description: "Progetti software di Toto Castaldi: Docora, Lumio e Helix"
---
```

### Complete baseof.html Structure
```html
<!-- layouts/_default/baseof.html -->
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}">
<head>
  {{ partial "head.html" . }}
</head>
<body>
  <header>
    <nav>
      {{ partial "language-switcher.html" . }}
    </nav>
  </header>
  <main>
    {{ block "main" . }}{{ end }}
  </main>
</body>
</html>
```

### Responsive Card Layout (mobile-first)
```css
/* Mobile-first: cards stack vertically by default */
.project-cards {
  display: grid;
  gap: var(--space-lg);
  padding: var(--space-md) 0;
}

/* Optional: side-by-side on wide screens if grid layout chosen */
@media (min-width: 768px) {
  .project-cards {
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  }
}
```

### Whole-Card Clickable Pattern (CSS only, no JS)
```html
<article class="project-card">
  <img src="..." alt="..." class="project-logo">
  <h2>{{ .name }}</h2>
  <p>{{ index .description site.Language.Lang }}</p>
  <a href="{{ .url }}" class="card-link">
    <span class="sr-only">{{ T "viewProject" }}: </span>{{ .name }}
  </a>
</article>
```

```css
.project-card {
  position: relative;
}

.card-link::after {
  content: "";
  position: absolute;
  inset: 0;
}
```

This uses the "pseudo-element overlay" pattern: the `<a>` element's `::after` pseudo-element stretches to cover the entire card, making the whole card clickable without JavaScript.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Separate CSS files for dark/light | CSS custom properties + `prefers-color-scheme` | ~2020 | Single file, auto-switching |
| Media queries for responsive type | `clamp()` for fluid typography | ~2022, now Baseline | No breakpoints needed for type |
| Web fonts for "nice" typography | System font stacks | Trend since ~2023 | Zero font requests, faster loads |
| `light-dark()` CSS function | Available but only Baseline 2024 (85%) | May 2024 | Simpler syntax but still too new for full compatibility; `prefers-color-scheme` is safer |
| `-apple-system, BlinkMacSystemFont` | `system-ui` as primary | ~2024 | Single value works everywhere modern |
| Manual SVG + PNG favicon | SVG favicon with embedded dark mode CSS | ~2022 | One file, automatic dark mode |

**Deprecated/outdated:**
- `-apple-system` and `BlinkMacSystemFont` font-family prefixes: `system-ui` is sufficient in 2025+, though legacy prefixes don't hurt.
- `<link rel="shortcut icon">`: Use `<link rel="icon">` (without "shortcut"). "shortcut" was never standard.

## Discretion Recommendations

Based on research, here are recommendations for areas marked as Claude's discretion:

### Card Visual Containment: Subtle border with rounded corners
**Rationale:** Bordered cards provide clear visual separation on a whitespace-heavy page without the visual heaviness of drop shadows. In dark mode, borders remain visible (just swap color). Borderless cards risk visual ambiguity; elevated (shadow) cards conflict with the minimal aesthetic.

### Card Link Interaction: Whole card clickable
**Rationale:** Better UX -- larger click target. The pseudo-element overlay pattern works with zero JS. Combine with a visible text link inside the card for accessibility (screen readers need a discoverable link element).

### Page Structure: Centered column, cards stacked vertically
**Rationale:** With only 3 cards, a single-column layout gives each card breathing room and avoids awkward grid math (3 cards in 2 columns = orphan). Centered column matches the "airy and spacious" requirement. Max-width around 640-720px.

### Color Palette: Warm neutrals
**Rationale:** Light mode: near-white background (#fafafa or #ffffff), near-black text (#1a1a1a), muted gray for secondary text (#6b7280). Dark mode: deep gray background (#111827), light gray text (#f3f4f6). Blue for links (#2563eb light / #60a5fa dark). All pairs exceed WCAG AA 4.5:1 ratio.

### Font Choice: System font stack
**Rationale:** `system-ui, -apple-system, sans-serif` -- zero network requests, meets UI-05 (sub-1s load), looks native on every OS. The "same font for headings and body, hierarchy through size and weight" decision makes system fonts ideal.

### Language Switcher: Show only the alternate language as a text link
**Rationale:** With only two languages, showing both is redundant (user already knows which one they are on). Show "EN" when on Italian, "IT" when on English. Use `.Translations` which naturally excludes the current language. Simple, minimal, fits the design.

### Favicon: Simple "TC" monogram or single letter
**Rationale:** A text-based SVG is trivially small, works at all sizes, and the dark mode color swap via embedded CSS is straightforward. More complex icon designs risk looking blurry at 16x16.

## Open Questions

1. **External logo availability**
   - What we know: Docora has `/images/logo.svg` on its subdomain. Lumio and Helix appear to have no visible logo/favicon in their HTML.
   - What's unclear: Whether all three products have usable logo images, and if they are served with appropriate CORS headers for cross-origin use.
   - Recommendation: During implementation, attempt to fetch each logo. If logos are missing or inaccessible, fall back to a styled text-only card (project name as the visual anchor). Consider copying available logos into `static/images/` for reliability and performance.

2. **Project descriptions content**
   - What we know: The TOML structure supports per-language descriptions.
   - What's unclear: The actual Italian and English descriptions for Docora, Lumio, and Helix have not been provided.
   - Recommendation: Use placeholder descriptions during implementation, to be replaced with final copy. Or infer from each product's own landing page.

3. **TOML array key naming**
   - What we know: `[[project]]` creates array at key `project`; file `projects.toml` creates namespace `projects`.
   - What's unclear: Whether Hugo 0.155.3 handles `[[project]]` within `data/projects.toml` exactly as expected (access via `.Site.Data.projects.project`).
   - Recommendation: Verify during first implementation task with `hugo server`. This is LOW risk but worth an early smoke test.

## Sources

### Primary (HIGH confidence)
- [Hugo Data Sources](https://gohugo.io/content-management/data-sources/) - data template file structure and access patterns
- [Hugo .Site.Data method](https://gohugo.io/methods/site/data/) - TOML array-of-tables access, `index` function
- [Hugo lang.Translate](https://gohugo.io/functions/lang/translate/) - i18n function syntax, TOML file format, pluralization
- [Hugo Multilingual Mode](https://gohugo.io/content-management/multilingual/) - `.Translations`, `.AllTranslations`, language switcher pattern
- [MDN prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-color-scheme) - dark mode media query syntax
- [MDN light-dark()](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/color_value/light-dark) - browser support data (Baseline 2024, 85%)
- [Can I Use light-dark()](https://caniuse.com/mdn-css_types_color_light-dark) - 84.93% global support

### Secondary (MEDIUM confidence)
- [Owen Conti - SVG Favicon Dark Mode](https://owenconti.com/posts/supporting-dark-mode-with-svg-favicons) - SVG favicon with embedded prefers-color-scheme
- [Aleksandr Hovhannisyan - Fluid Type Scale](https://www.aleksandrhovhannisyan.com/blog/fluid-type-scale-with-css-clamp/) - clamp() type scale formula and values
- [Utopia - CSS Modular Scales](https://utopia.fyi/blog/css-modular-scales/) - advanced fluid scale approach (referenced but simpler approach recommended)
- [System Font Stack](https://systemfontstack.com/) - modern system-ui font stack reference
- [Hugo Discourse - Multilingual Data Files](https://discourse.gohugo.io/t/multilingual-data-files/4684) - community patterns for per-language data
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) - WCAG AA ratio verification tool

### Tertiary (LOW confidence)
- Product site inspection: Docora has `/images/logo.svg`; Lumio and Helix logos not confirmed from web fetch (may exist but weren't in fetched HTML)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All built-in Hugo features, verified against official docs
- Architecture: HIGH - Patterns verified against Hugo docs and established CSS practices
- Pitfalls: HIGH - Based on documented Hugo behaviors and common multilingual issues
- Design recommendations: MEDIUM - Based on research and best practices but involve subjective choices
- External logos: LOW - Only Docora logo confirmed; Lumio and Helix need verification during implementation

**Research date:** 2026-02-18
**Valid until:** 2026-03-18 (Hugo and CSS standards are stable)
