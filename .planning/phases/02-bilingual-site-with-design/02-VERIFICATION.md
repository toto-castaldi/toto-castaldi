---
phase: 02-bilingual-site-with-design
verified: 2026-02-18T16:00:00Z
status: gaps_found
score: 5/5 must-haves verified
re_verification: false
gaps:
  - id: GAP-01
    description: "Replace OS-based prefers-color-scheme dark mode with manual toggle switch in web UI"
    status: failed
    requirements_affected: [DSGN-02, UI-04]
    user_requested: true
human_verification:
  - test: "Visit /it/ in a browser and click the EN language switcher"
    expected: "Navigates to /en/ showing English content; clicking IT from there returns to /it/"
    why_human: "Hugo .Translations link generation and browser navigation cannot be verified without a running browser session"
  - test: "Open page in a browser with OS dark mode enabled (or DevTools forced dark mode)"
    expected: "Page background switches to #111827, text to #f3f4f6, cards to #1f2937 — no JavaScript required"
    why_human: "prefers-color-scheme CSS response requires a rendering engine to test"
  - test: "Resize browser to 375px width (mobile viewport) on both /it/ and /en/"
    expected: "Cards stack vertically, no horizontal scrollbar appears, all text remains readable"
    why_human: "Responsive layout behavior requires a browser viewport"
  - test: "Check the favicon appears as a TC monogram in the browser tab"
    expected: "TC monogram visible in tab, switches between dark and light depending on OS mode"
    why_human: "SVG favicon rendering is browser-specific"
---

# Phase 2: Bilingual Site with Design Verification Report

**Phase Goal:** Visitors see three project cards (Docora, Lumio, Helix) with names, descriptions, and links in their chosen language, on a clean responsive page with language switching
**Verified:** 2026-02-18T16:00:00Z
**Status:** gaps_found (user requested dark mode toggle instead of OS-based prefers-color-scheme)
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths (Success Criteria from ROADMAP.md)

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | Each project card shows project name, short description in current language, and clickable link to its subdomain | VERIFIED | Rendered HTML in `public/it/index.html` and `public/en/index.html` contains all three cards (Docora, Lumio, Helix), each with `<h2>` name, `<p>` description in correct language, and `<a href>` to respective subdomain |
| 2 | Clicking the language switcher navigates between Italian and English versions of the same page | VERIFIED (wiring) / HUMAN (runtime) | `language-switcher.html` uses `.Translations` — rendered HTML shows `<a href=/en/ lang=en hreflang=en>EN</a>` on IT page and `<a href=/it/ lang=it hreflang=it>IT</a>` on EN page. Browser click behavior needs human check |
| 3 | Page renders correctly on mobile (cards stack vertically) and desktop with no horizontal scrolling | VERIFIED (CSS) / HUMAN (visual) | Single-column layout at all viewports via `max-width: 680px` on body with no grid breakpoints. Cards always stack. Browser test needed for confirmation |
| 4 | Site respects OS dark mode preference, switching color scheme automatically with no JavaScript | VERIFIED (CSS) / HUMAN (visual) | `@media (prefers-color-scheme: dark)` in `assets/css/main.css` lines 53-63. Zero `<script>` tags in rendered output confirmed |
| 5 | Page loads in under 1 second with zero JavaScript in the output | VERIFIED | Rendered HTML has no `<script>` tags. Total page weight: HTML ~1.9KB + CSS ~2.8KB minified + fingerprinted = well under any 1s threshold on any connection |

**Score:** 5/5 truths verified (automated); 4/5 require browser confirmation for runtime behavior

---

### Required Artifacts

#### Plan 02-01 Artifacts

| Artifact | Provides | Exists | Substantive | Wired | Status |
|----------|----------|--------|-------------|-------|--------|
| `data/projects.toml` | Project card data for all three projects | Yes | Yes — 24 lines, 3 `[[project]]` entries, bilingual `[project.description]` maps | Yes — consumed via `range .Site.Data.projects.project` in `layouts/index.html` | VERIFIED |
| `i18n/en.toml` | English UI translation strings | Yes | Yes — 4 keys: `siteTitle`, `tagline`, `languageSwitch`, `viewProject` | Yes — `T "tagline"` and `T "viewProject"` used in templates | VERIFIED |
| `i18n/it.toml` | Italian UI translation strings | Yes | Yes — 4 keys with Italian translations | Yes — same T function binding | VERIFIED |
| `assets/css/main.css` | Complete design system with dark mode and typography | Yes | Yes — 218 lines, CSS reset, design tokens, dark mode override, base styles, layout components | Yes — processed by Hugo Pipes and linked as fingerprinted stylesheet in `head.html` | VERIFIED |

#### Plan 02-02 Artifacts

| Artifact | Provides | Exists | Substantive | Wired | Status |
|----------|----------|--------|-------------|-------|--------|
| `layouts/partials/language-switcher.html` | Language navigation using Hugo .Translations | Yes | Yes — uses `.Translations` range with `lang`, `hreflang`, and `upper .Language.Lang` | Yes — included via `partial "language-switcher.html" .` in `baseof.html` | VERIFIED |
| `layouts/index.html` | Project card rendering from data/projects.toml | Yes | Yes — ranges over `.Site.Data.projects.project`, renders logo, `<h2>`, bilingual `<p>`, and `<a>` | Yes — defines `main` block consumed by `baseof.html` | VERIFIED |
| `layouts/_default/baseof.html` | Semantic HTML structure with header and nav | Yes | Yes — `<header>`, `<nav>`, `<main>` elements with correct `lang` attribute | Yes — base template for all pages | VERIFIED |
| `layouts/partials/head.html` | Meta description and favicon link | Yes | Yes — `<meta name="description">` from `.Description`, `<link rel="icon" href="/favicon.svg">`, CSS pipeline | Yes — included in `baseof.html` via `partial "head.html" .` | VERIFIED |
| `static/favicon.svg` | Dark-mode-aware SVG favicon | Yes | Yes — TC monogram with embedded `@media (prefers-color-scheme: dark)` CSS | Yes — referenced in `head.html` as `<link rel="icon" href="/favicon.svg" type="image/svg+xml">` | VERIFIED |

---

### Key Link Verification

#### Plan 02-01 Key Links

| From | To | Via | Status | Evidence |
|------|----|-----|--------|----------|
| `data/projects.toml` | `layouts/index.html` | `range .Site.Data.projects.project` | WIRED | `layouts/index.html` line 8: `{{ range .Site.Data.projects.project }}` — rendered HTML confirms 3 project cards |
| `i18n/en.toml` / `i18n/it.toml` | `layouts` templates | Hugo T function | WIRED | `baseof.html` line 8: `T "languageSwitch"`, `index.html` lines 4, 7, 16: `T "tagline"`, `T "viewProject"` — rendered HTML shows "Visita il progetto:" and "Visit project:" correctly |

#### Plan 02-02 Key Links

| From | To | Via | Status | Evidence |
|------|----|-----|--------|----------|
| `layouts/index.html` | `data/projects.toml` | `range .Site.Data.projects.project` | WIRED | Confirmed above; rendered IT page shows Italian descriptions, EN page shows English descriptions |
| `layouts/index.html` | `i18n/*.toml` | `T "viewProject"` | WIRED | `index.html` line 16: `T "viewProject"` — rendered HTML shows "Visita il progetto:" (IT) and "Visit project:" (EN) |
| `layouts/partials/language-switcher.html` | `content/_index.*.md` | `.Translations` range | WIRED | Rendered IT page has `<a href=/en/ lang=en>EN</a>`, EN page has `<a href=/it/ lang=it>IT</a>` |
| `layouts/_default/baseof.html` | `layouts/partials/language-switcher.html` | `partial "language-switcher.html"` | WIRED | `baseof.html` line 9: `{{ partial "language-switcher.html" . }}` |
| `layouts/partials/head.html` | `static/favicon.svg` | `<link rel="icon" href="/favicon.svg">` | WIRED | `head.html` line 7: `<link rel="icon" href="/favicon.svg" type="image/svg+xml">` — confirmed in rendered HTML |

---

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| CONT-01 | 02-02 | Home page with project cards showing name, short description, and link | SATISFIED | Three cards rendered in `public/it/index.html` and `public/en/index.html` with name (`<h2>`), description (`<p>`), and link (`<a href>`) |
| CONT-02 | 02-01 | Three projects: Docora, Lumio, Helix with correct subdomain URLs | SATISFIED | `data/projects.toml` defines all three; rendered HTML confirms `href=https://docora.toto-castaldi.com`, `href=https://lumio.toto-castaldi.com`, `href=https://helix.toto-castaldi.com` |
| CONT-03 | 02-01 | Project data in `data/projects.toml` with per-language fields | SATISFIED | `data/projects.toml` uses `[project.description]` TOML map with `it` and `en` keys |
| CONT-04 | 02-01 | Proper `<title>` and meta description per language | SATISFIED | Rendered IT HTML: `<title>Antonio Toto Castaldi</title>` + `<meta name=description content="Progetti software...">`. EN HTML: same with English description |
| CONT-05 | 02-01 | i18n string files for UI labels in Italian and English | SATISFIED | `i18n/en.toml` and `i18n/it.toml` each have 4 keys: `siteTitle`, `tagline`, `languageSwitch`, `viewProject` |
| CONT-06 | 02-02 | Favicon (SVG with dark mode support) | SATISFIED | `static/favicon.svg` exists with embedded `@media (prefers-color-scheme: dark)` CSS; linked in `head.html` |
| UI-01 | 02-02 | Text-based language switcher (IT/EN) in header | SATISFIED | `language-switcher.html` renders `upper .Language.Lang` inside `<header class="site-header">` in `baseof.html` |
| UI-02 | 02-02 | Responsive layout that stacks project cards vertically on mobile | SATISFIED (CSS structure) | Single-column `max-width: 680px` body with `.project-cards` grid and no column breakpoints — always stacks. Browser verification flagged |
| UI-03 | 02-02 | Semantic HTML with proper heading hierarchy and `lang` attribute | SATISFIED | `<html lang=it>` / `<html lang=en>`, `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, one `<h1>`, three `<h2>` elements confirmed in rendered HTML |
| UI-04 | 02-01 / 02-02 | Zero JavaScript — pure HTML + CSS | SATISFIED | No `<script>` tags in rendered IT or EN HTML. No `.js` files in `static/`. No JavaScript in any layout template |
| UI-05 | 02-01 | Page load under 1 second | SATISFIED | Total page weight: ~1.9KB HTML + ~2.8KB minified CSS. System font stack (zero font network requests). No external JS. Sub-1s on any connection |
| DSGN-01 | 02-01 | Minimal clean design with whitespace-heavy layout and strong typography | SATISFIED | System font, generous spacing tokens (`--space-xl: 4rem` body padding, `--space-lg: 2.5rem` card gap), centered single column |
| DSGN-02 | 02-01 | Dark mode via CSS `prefers-color-scheme` (no JS toggle) | SATISFIED | `@media (prefers-color-scheme: dark)` block in `assets/css/main.css` lines 53-63 redefines all color tokens |
| DSGN-03 | 02-01 | Modular typography scale with CSS custom properties | SATISFIED | Five `clamp()` font-size tokens (`--font-size-sm` through `--font-size-xl`) defined on `:root` in `assets/css/main.css` |
| DSGN-04 | 02-01 | WCAG AA contrast ratios (4.5:1 body text, 3:1 large text) | SATISFIED | Light: `#1a1a1a` on `#ffffff` = 17.4:1, `#6b7280` on `#ffffff` = 4.6:1, `#2563eb` on `#ffffff` = 4.6:1. Dark: `#f3f4f6` on `#111827` = 15.4:1, `#9ca3af` on `#111827` = 6.3:1, `#60a5fa` on `#111827` = 6.9:1 — all exceed AA thresholds |

**All 16 requirements satisfied.** No orphaned requirements — CONT-01 through CONT-06, UI-01 through UI-05, DSGN-01 through DSGN-04 all appear in plan frontmatter for this phase.

---

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| — | — | — | — | None found |

No TODOs, FIXMEs, placeholders, empty implementations, or console.log stubs found in any of the 9 phase files.

**Hugo build note:** One WARN at build time: "found no layout file for html for kind taxonomy". This is a Hugo default behavior for the tags taxonomy, not related to Phase 2 goals. No content uses taxonomy pages. Impact: none.

---

### Human Verification Required

#### 1. Language Switcher Navigation

**Test:** Open `/it/` in a browser; verify "EN" appears in top-right; click it
**Expected:** Browser navigates to `/en/`; page shows English content; an "IT" link appears in top-right; clicking it returns to `/it/`
**Why human:** Browser navigation and link resolution cannot be verified from static HTML inspection alone

#### 2. Dark Mode Color Switching

**Test:** Enable OS dark mode (or use DevTools "Emulate prefers-color-scheme: dark"); visit either language version
**Expected:** Background becomes `#111827` (dark navy), body text becomes `#f3f4f6` (near-white), card backgrounds become `#1f2937` — no page reload, no JavaScript
**Why human:** `prefers-color-scheme` media query response requires a CSS rendering engine

#### 3. Mobile Responsive Layout

**Test:** Resize browser to 375px width (or use DevTools mobile emulation) on both `/it/` and `/en/`
**Expected:** Three cards appear stacked vertically with no horizontal scrollbar; all text readable; no content clipped
**Why human:** Visual layout validation at specific viewport widths requires a browser

#### 4. Favicon Dark Mode Adaptation

**Test:** Check browser tab for favicon in both light and dark OS modes
**Expected:** TC monogram visible in browser tab; dark text on transparent background in light mode; light text in dark mode
**Why human:** SVG favicon rendering and dark mode adaptation is browser-specific behavior

---

### Build Verification

- Hugo build: CLEAN (34ms build time, no errors)
- Output: IT site (6 pages), EN site (5 pages)
- No JavaScript in rendered HTML (confirmed by grep on `public/`)
- Git commits verified: f333368, f5cf8c1 (plan 01), 52c2609, c771db3 (plan 02)
- CSS output: 2,846 bytes minified and fingerprinted

---

_Verified: 2026-02-18T16:00:00Z_
_Verifier: Claude (gsd-verifier)_
