# Project Research Summary

**Project:** toto-castaldi.com
**Domain:** Multilingual static landing page (Hugo + GitHub Pages)
**Researched:** 2026-02-17
**Confidence:** HIGH

## Executive Summary

This is a minimal multilingual developer project showcase: a single-page static site listing three software projects (Docora, Lumio, Helix) in Italian and English, hosted on GitHub Pages at a custom domain. The expert approach for this type of site is Hugo (extended edition) with its built-in i18n system, deployed via GitHub Actions. No Node.js, no JavaScript frameworks, no external theme — Hugo handles SCSS transpilation, minification, and fingerprinting natively through Hugo Pipes, making the entire build pipeline dependency-free except for Hugo itself and Dart Sass.

The recommended approach is a lean, inline-layout structure: write ~150 lines of HTML templates directly in `layouts/`, define projects as structured data in `data/projects.toml`, translate UI strings via `i18n/en.toml` and `i18n/it.toml`, and translate page content via filename-suffixed Markdown files (`_index.en.md`, `_index.it.md`). This eliminates theme overhead, git submodule fragility, and Go module complexity. The entire site builds in milliseconds and delivers sub-1-second page loads with zero JavaScript at runtime.

The key risks are infrastructure-level and must be addressed in Phase 1 before any content work begins: `baseURL` must include a trailing slash in `hugo.toml`, `defaultContentLanguageInSubdir` must be set intentionally before content is created (changing it later rewrites all URLs), the `static/CNAME` file must exist in source (not just GitHub UI), and DNS records must be configured in the correct order to avoid domain takeover risk and HTTPS certificate failure. Multilingual SEO (hreflang tags, canonical URLs, Open Graph) is well-understood and easy to implement once the foundation is correct.

## Key Findings

### Recommended Stack

Hugo extended edition v0.155.3 with Dart Sass 1.97.3 is the correct stack, deployed via the official GitHub Actions workflow. LibSass was deprecated in Hugo v0.153.0 — extended edition with Dart Sass is the only supported path for SCSS. No npm, no Webpack, no Tailwind: Hugo Pipes handles everything. The config format is TOML (`hugo.toml`), which is Hugo's default and has the most community documentation.

See `.planning/research/STACK.md` for full details.

**Core technologies:**
- **Hugo extended v0.155.3**: Static site generator — built-in multilingual i18n, asset pipeline (Hugo Pipes), millisecond build times, no external dependencies
- **Dart Sass 1.97.3**: SCSS transpilation — official replacement for deprecated LibSass; required for SCSS support in Hugo extended
- **GitHub Actions**: CI/CD — official Hugo starter workflow handles build + deploy with zero configuration after setup
- **GitHub Pages**: Hosting — free, custom domain support, automatic HTTPS via Let's Encrypt
- **Custom SCSS (no framework)**: Styling — hand-written SCSS under 100 lines is the right fit; Tailwind/Bootstrap are overkill for 3 cards
- **System font stack**: Typography — zero network requests, GDPR-compliant, native feel on every device

### Expected Features

Research distinguishes clearly between P1 (launch blockers), P2 (post-launch polish), and P3 (future consideration). The boundary is clean: everything needed for a functional bilingual site with visible projects is P1; everything that improves SEO and social sharing is P2; everything else is P3 or anti-feature.

See `.planning/research/FEATURES.md` for full details.

**Must have (table stakes / P1 — the site is broken without these):**
- Project cards (name, description, link) for Docora, Lumio, Helix — this IS the product
- Hugo multilingual setup (IT/EN) with i18n string files — foundational, all other features depend on it
- Language switcher in header (text-based, not flags) — bilingual visitors expect it
- Responsive layout — ~83% of traffic is mobile
- Semantic HTML with proper heading hierarchy and `lang` attribute — screen readers, SEO, correctness
- Favicon (SVG with dark mode support) — missing favicon looks unfinished
- Proper `<title>` and meta description per language — search engine baseline
- HTTPS via GitHub Pages — non-negotiable

**Should have (competitive differentiators / P2 — add after initial deploy):**
- hreflang tags — multilingual SEO; Google needs these to show the right language version
- Open Graph meta tags — rich previews when URL is shared on LinkedIn, Slack, etc.
- Canonical URLs per language — prevents duplicate content SEO penalty
- Dark mode (CSS `prefers-color-scheme` only, no JS toggle) — respects OS preference, signals craft
- WCAG AA contrast audit — verify after final color palette
- Intentional typography scale via CSS custom properties

**Defer (v2+ / P3 — only if specific need arises):**
- OG image (requires graphic design effort)
- Structured data / JSON-LD
- Goatcounter analytics (only if traffic measurement needed)
- Additional projects beyond initial three

**Anti-features (deliberately excluded):**
- About/bio section, contact form, blog, JS animations, dark mode toggle, social media links, cookie banner, auto-language-detect, project detail pages

### Architecture Approach

The architecture is a pure static site generator pattern: Hugo compiles templates + content + data into flat HTML/CSS files; GitHub Pages serves those files. There is no server, no database, no API. The "backend" is the build step. Six components handle all concerns with clean separation: config (TOML files), content (Markdown with language suffixes), i18n strings (TOML key-value files), layouts (Hugo templates), assets (SCSS through Hugo Pipes), and structured data (`data/projects.toml`).

See `.planning/research/ARCHITECTURE.md` for full directory structure, component diagrams, and pattern examples.

**Major components:**
1. **`config/_default/`** — site metadata, language definitions, menu structures (split TOML files by concern)
2. **`content/`** — page content (`_index.en.md`, `_index.it.md`); filename-suffix translation method
3. **`i18n/`** — UI string translations (`en.toml`, `it.toml`); all button labels, nav text, headings
4. **`data/projects.toml`** — structured project data iterated by template; separates data from presentation
5. **`layouts/`** — HTML templates: `baseof.html` (shell), `index.html` (home), 5 partials (head, header, project-card, language-switcher, footer)
6. **`assets/css/main.css`** — SCSS processed through Hugo Pipes (minify + fingerprint for cache busting)
7. **`.github/workflows/hugo.yaml`** — CI/CD; official Hugo starter workflow

### Critical Pitfalls

Research identified 5 critical pitfalls, all verifiable against official docs. Four are Day 0 decisions that cannot be corrected cheaply later.

See `.planning/research/PITFALLS.md` for full recovery strategies and verification checklists.

1. **CNAME file wiped on every deployment** — place `static/CNAME` containing `toto-castaldi.com` in the Hugo source repo; never rely solely on the GitHub UI setting
2. **`baseURL` missing trailing slash breaks all asset paths** — set `baseURL = "https://toto-castaldi.com/"` (with trailing slash) before first deployment; the most-reported Hugo + GitHub Pages problem
3. **`defaultContentLanguageInSubdir` decision not made before content creation** — decide before writing any content; use `true` for symmetric `/it/` and `/en/` URL structure; changing later rewrites all URLs and breaks search indexes
4. **DNS configuration order causes HTTPS certificate failure or domain takeover risk** — add custom domain in GitHub Settings first, then create DNS records; never use wildcard DNS (`*.toto-castaldi.com`)
5. **Missing or incorrect hreflang tags** — use `.AllTranslations` in a partial template; always use `.Permalink` (absolute URLs), always include `x-default`, always make references reciprocal; ~75% of hreflang implementations have errors

## Implications for Roadmap

Based on the dependency graph from architecture research and the pitfall-to-phase mapping from pitfalls research, a 4-phase structure is recommended. Architecture research explicitly defines a build order with the same 5 steps; phases below consolidate these into shipping increments.

### Phase 1: Foundation and Infrastructure

**Rationale:** All other work depends on correct Hugo multilingual configuration. The four most critical pitfalls are Day 0 infrastructure decisions that cannot be changed cheaply later (`baseURL`, `defaultContentLanguageInSubdir`, `static/CNAME`, DNS order). This phase must be complete and deployed before any content or design work begins.

**Delivers:** A deployable blank site at `toto-castaldi.com` with correct multilingual URL structure, HTTPS enforced, and custom domain stable across deployments.

**Addresses (from FEATURES.md):** HTTPS, proper URL structure for language switcher

**Implements (from ARCHITECTURE.md):** `config/_default/` (hugo.toml, languages.toml), `baseof.html`, `.github/workflows/hugo.yaml`, `static/CNAME`

**Avoids (from PITFALLS.md):** CNAME wiped on deployment, baseURL mismatch, defaultContentLanguageInSubdir wrong, DNS order-of-operations

**Research flag:** Standard patterns, well-documented in official Hugo docs — no additional research phase needed.

### Phase 2: Multilingual Content and Templates

**Rationale:** With infrastructure in place, build the content layer and all language-aware templates. Hugo i18n is the foundation that language switcher, hreflang, per-language meta, and canonical URLs all depend on — research explicitly notes this dependency chain. The language switcher linking correctly to translated equivalent pages (not just the homepage) is a distinct UX pitfall that must be tested in this phase.

**Delivers:** A functional bilingual site with correct content in both IT and EN, language switcher working, all UI strings translated, and multilingual SEO foundations in place.

**Addresses (from FEATURES.md):** Hugo i18n setup, language switcher, semantic HTML, `<title>` + meta per language, hreflang tags, canonical URLs

**Implements (from ARCHITECTURE.md):** `content/_index.en.md` + `_index.it.md`, `i18n/en.toml` + `i18n/it.toml`, `index.html` template, `head.html` partial (meta, hreflang, canonical), `header.html` + `language-switcher.html` partials

**Avoids (from PITFALLS.md):** Hardcoded strings instead of i18n, language switcher linking to homepage instead of translated page, missing hreflang tags, missing canonical URLs, no `lang` attribute on `<html>`

**Research flag:** Standard Hugo patterns, well-documented — no additional research phase needed.

### Phase 3: Project Showcase and Visual Design

**Rationale:** With bilingual infrastructure and templates established, add the actual content (project cards) and visual design. Architecture research recommends keeping the project data structure simple (`data/projects.toml` with per-language description fields) for 3 projects — richer content page structure would be overengineering at this scale.

**Delivers:** The complete functional landing page: three project cards (Docora, Lumio, Helix) rendered correctly in both languages, fully styled, responsive, with favicon and dark mode support.

**Addresses (from FEATURES.md):** Project cards (name, description, link), responsive layout, favicon, dark mode (CSS only), typography scale, WCAG AA contrast, accessible contrast ratios

**Implements (from ARCHITECTURE.md):** `data/projects.toml`, `project-card.html` partial, `assets/css/main.css` (SCSS via Hugo Pipes), `footer.html` partial, `static/favicon.svg`

**Avoids (from PITFALLS.md):** CSS in `static/` instead of `assets/` (no cache busting), large unoptimized images, no HTML/CSS minification

**Research flag:** CSS architecture and modular type scale are standard patterns — no additional research phase needed. If a specific web font is needed beyond the system stack, self-hosting WOFF2 requires checking Google Fonts license for the chosen typeface.

### Phase 4: Polish and SEO Verification

**Rationale:** After the core site is live and visually complete, verify everything that "looks done but isn't" per the PITFALLS.md checklist. Add P2 features (Open Graph, print stylesheet) and run Lighthouse. This phase is verification-heavy — most work is checking deployed behavior, not writing new code.

**Delivers:** A production-ready site with full multilingual SEO, social sharing previews, accessibility verification, and clean Lighthouse scores.

**Addresses (from FEATURES.md):** Open Graph meta tags, print-friendly stylesheet, WCAG AA contrast audit (verified against deployed site)

**Implements (from ARCHITECTURE.md):** OG meta tags in `head.html` partial, `@media print` CSS block, multilingual 404 page

**Avoids (from PITFALLS.md):** Missing OG locale tags, single-language 404 page, sitemap missing language versions

**Research flag:** Standard patterns — no additional research phase needed. The multilingual 404 page (GitHub Pages only serves one root `/404.html`) requires a small JS-based language detection snippet; this is documented in pitfalls research and is a known workaround.

### Phase Ordering Rationale

- **Infrastructure before content:** The `defaultContentLanguageInSubdir` setting and `baseURL` must be final before any content files are created. Changing URL structure after content exists is a full-project rewrite with SEO consequences.
- **i18n before features:** Every feature (language switcher, hreflang, per-language meta) depends on Hugo's multilingual configuration being correct. This is the most explicit dependency chain in feature research.
- **Data structure before templates:** `data/projects.toml` schema should be defined before writing `project-card.html` since the template references specific field names.
- **Core before polish:** Open Graph and print stylesheet have zero dependencies on each other or on the project cards — they are straightforward additions once the `<head>` partial structure exists.

### Research Flags

Phases with standard patterns (no research-phase needed):
- **Phase 1:** Official Hugo + GitHub Pages documentation is comprehensive and current (Feb 2026 versions confirmed)
- **Phase 2:** Hugo i18n is well-documented in official docs with concrete examples
- **Phase 3:** CSS architecture for a minimal landing page has no novel decisions
- **Phase 4:** OG tags and multilingual 404 are documented patterns with known solutions

No phases require `/gsd:research-phase` during planning. All decisions are covered by research already completed at HIGH confidence from official sources.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All recommendations sourced from official Hugo docs (Feb 2026). Version numbers verified from GitHub releases. No community inference needed. |
| Features | HIGH | MVP definition is clear and bounded. Table stakes vs. anti-features distinction is well-reasoned. Source mix of official docs + UX research (Smashing Magazine, Google Search) is credible. |
| Architecture | HIGH | Architecture is Hugo's standard pattern, documented comprehensively in official Hugo docs. All code examples are verified template syntax. |
| Pitfalls | HIGH | Every critical pitfall verified against official docs or GitHub issue tracker. The most common bugs (baseURL, CNAME, hreflang errors) have documented community evidence of frequency. |

**Overall confidence:** HIGH

### Gaps to Address

- **Exact project descriptions:** `data/projects.toml` requires the actual one-sentence descriptions for Docora, Lumio, and Helix in both IT and EN. This is content, not architecture — can be filled in during Phase 3. Placeholder text is acceptable for earlier phases.
- **Custom font decision:** Stack research recommends system fonts as default with clear guidance on self-hosting WOFF2 if a specific typeface is needed. The final typography decision (system stack vs. specific font) should be made before Phase 3 begins, since it affects whether `static/fonts/` and `@font-face` SCSS are needed.
- **`defaultContentLanguageInSubdir` final decision:** Research recommends `true` (symmetric `/it/` and `/en/` URLs) but notes the trade-off of an HTML meta-refresh redirect at `/`. This is a Day 0 decision that must be confirmed before Phase 1 completes. The meta-refresh latency is minimal for a landing page but the UX pitfall (flash of redirect page) should be mitigated with an instant-redirect template at the root index.

## Sources

### Primary (HIGH confidence)
- [Hugo official docs: Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/) — GitHub Actions workflow, custom domain, CNAME setup
- [Hugo official docs: Multilingual mode](https://gohugo.io/content-management/multilingual/) — i18n configuration, filename-based translation, language switcher
- [Hugo official docs: Configuration introduction](https://gohugo.io/configuration/introduction/) — TOML config format
- [Hugo official docs: Hugo Pipes](https://gohugo.io/hugo-pipes/) — CSS minification, fingerprinting, Sass
- [Hugo official docs: css.Sass](https://gohugo.io/functions/css/sass/) — LibSass deprecation, Dart Sass migration
- [Hugo official docs: Directory Structure](https://gohugo.io/getting-started/directory-structure/) — standard project layout
- [Hugo official docs: Template Types](https://gohugo.io/templates/types/) — baseof, home, partial templates
- [Hugo official docs: Data Templates](https://gohugo.io/templates/data-templates/) — data/ directory, .Site.Data
- [Hugo official docs: lang.Translate](https://gohugo.io/functions/lang/translate/) — i18n T() function
- [Hugo official docs: URL Management](https://gohugo.io/content-management/urls/) — baseURL, relURL, absURL
- [Hugo official docs: Configure Languages](https://gohugo.io/configuration/languages/) — defaultContentLanguageInSubdir
- [Hugo GitHub releases](https://github.com/gohugoio/hugo/releases) — v0.155.3 release date and changelog
- [GitHub docs: Managing Custom Domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) — DNS A records, CNAME, HTTPS provisioning
- [GitHub docs: Securing with HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https) — Enforce HTTPS, certificate provisioning
- [GitHub Actions starter workflow for Hugo](https://github.com/actions/starter-workflows/blob/main/pages/hugo.yml) — official workflow YAML
- [Google: x-default hreflang](https://developers.google.com/search/blog/2013/04/x-default-hreflang-for-international-pages) — hreflang requirements
- [Google: Managing Multi-Regional Sites](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites) — multilingual SEO rules

### Secondary (MEDIUM confidence)
- [Smashing Magazine: Designing a Better Language Selector](https://www.smashingmagazine.com/2022/05/designing-better-language-selector/) — flags vs. text switcher UX
- [Hugo Discourse: Modules vs submodules](https://discourse.gohugo.io/t/theme-as-a-hugo-module-or-theme-as-a-git-submodule/54388) — community consensus on submodules as anti-pattern
- [Hugo Discourse: defaultContentLanguageInSubdir gotchas](https://discourse.gohugo.io/t/defaultcontentlanguageinsubdir/4658) — URL asymmetry issues
- [Hugo Discourse: Custom 404 per language](https://discourse.gohugo.io/t/custom-404-per-language/20239) — multilingual 404 workaround
- [Hugo Issue #5898](https://github.com/gohugoio/hugo/issues/5898) — redirects for defaultContentLanguage sections
- [Regis Philibert: Hugo Multilingual Part 1](https://www.regisphilibert.com/blog/2018/08/hugo-multilingual-part-1-managing-content-translation/) — translation approach comparison
- [hreflang on Hugo sites (Wieckiewicz)](https://dariusz.wieckiewicz.org/en/setting-hreflang-and-x-default-on-multilingual-site-part-2/) — implementation examples

---
*Research completed: 2026-02-17*
*Ready for roadmap: yes*
