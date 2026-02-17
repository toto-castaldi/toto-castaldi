# Feature Research

**Domain:** Minimal multilingual developer project showcase landing page
**Researched:** 2026-02-17
**Confidence:** HIGH

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Project cards with name, description, and link | This IS the product. Visitors arrive to see projects and click through. Without clear cards, there is no landing page. | LOW | Each card: project name (h2), one-sentence description, link to subdomain. Three cards total. Hugo `data/` or front matter. |
| Language switcher (IT/EN) | Bilingual visitors expect to toggle language. A multilingual site without a visible switcher is broken UX. | LOW | Text-based, not flags. Flags represent countries, not languages (Smashing Magazine best practice). Place in header/nav corner. Use Hugo's built-in i18n with `{{ .Translations }}`. |
| Responsive / mobile-friendly layout | ~83% of web traffic is mobile. A landing page that breaks on phones is unusable. | LOW | CSS Grid or Flexbox. Three cards stack vertically on mobile. Hugo generates static HTML; responsiveness is pure CSS. |
| Semantic HTML with proper heading hierarchy | Screen readers, SEO crawlers, and browsers all depend on semantic structure. Broken semantics = invisible to many users. | LOW | `<main>`, `<section>`, `<h1>` for page title, `<h2>` per project. `lang` attribute on `<html>` matching the current language. |
| Favicon | Missing favicon = broken icon in browser tabs, bookmarks, and link previews. Looks unfinished. | LOW | SVG favicon with dark mode support via `<style>` inside SVG `<defs>` and `@media (prefers-color-scheme: dark)`. Single file, works in 97%+ of browsers. |
| Fast page load (< 1s) | Single-page static site with no images should be near-instant. Slow load on a minimal page signals incompetence for a developer showcase. | LOW | Hugo generates static HTML. No JS framework needed. Inline critical CSS or use a single small stylesheet. No external dependencies at runtime. |
| HTTPS | Browsers mark HTTP as "not secure." GitHub Pages provides HTTPS for custom domains for free. | LOW | Configured at GitHub Pages level, not in Hugo. Add CNAME file for custom domain. |
| Proper `<title>` and meta description per language | Search engines and browser tabs need meaningful titles. Each language version needs its own metadata. | LOW | Hugo i18n strings or per-language front matter. `<title>Toto Castaldi - Progetti Software</title>` (IT) / `<title>Toto Castaldi - Software Projects</title>` (EN). |

### Differentiators (Competitive Advantage)

Features that set this product apart. Not required, but add polish that signals craft.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| hreflang tags for multilingual SEO | Tells Google which language version to serve to which audience. Without hreflang, Google may show the Italian page to English searchers or flag pages as duplicate content. Most developer portfolio sites skip this entirely. | LOW | Hugo can generate these in the `<head>` partial. Add `<link rel="alternate" hreflang="it" href="...">` and `hreflang="en"` plus `hreflang="x-default"`. Reciprocal linking is required (Google's rule). |
| Open Graph meta tags | When someone shares the URL on LinkedIn, Twitter, or Slack, a rich preview (title, description, image) appears instead of a bare URL. Signals professionalism. | LOW | `og:title`, `og:description`, `og:url`, `og:type` (website), `og:locale` (per language). Optional: `og:image` if a simple brand image is added later. Hugo template in `<head>`. |
| Dark mode support (CSS `prefers-color-scheme`) | Respects user's OS-level preference. A developer showcase that handles dark mode shows attention to detail. Most minimal Hugo sites do not bother. | MEDIUM | CSS custom properties for colors, `@media (prefers-color-scheme: dark)` block. No JS toggle needed -- just respect the OS preference. Keep it simple: swap background/text colors and link colors. |
| Print-friendly stylesheet | Visitors who want to save/print the project list get a clean output. Uncommon on developer sites but trivially implementable. | LOW | `@media print` CSS block hiding nav, language switcher; ensuring text is black-on-white. |
| Accessible contrast ratios (WCAG AA) | 4.5:1 minimum contrast for body text, 3:1 for large text. Goes beyond "works" to "works for everyone." | LOW | Choose color palette deliberately. Test with browser DevTools contrast checker. Strong typography + high contrast = the design ethos already. |
| Smooth, intentional typography scale | Using a modular type scale (e.g., 1.25 ratio) with deliberate sizing for h1/h2/body creates visual authority. Most dev portfolios use arbitrary sizes. | LOW | Define `--font-size-base`, `--font-size-lg`, `--font-size-xl`, `--font-size-2xl` as CSS custom properties. Two font families maximum: one sans-serif for headings (e.g., Inter), one for body (system font stack or same). |
| Canonical URLs per language version | Prevents SEO duplicate content penalties. Each language page declares itself as canonical. | LOW | `<link rel="canonical" href="...">` in `<head>`, generated from Hugo's `.Permalink`. |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems for this specific project.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| About/bio section | "People want to know who you are" | Explicitly out of scope per PROJECT.md. Adds content that dilutes the single purpose (showcase projects). Requires maintaining another block of translated content. | The projects themselves communicate capability. If needed later, add a single sentence above the project cards, not a section. |
| Contact form | "How will people reach you?" | Requires a form backend (Formspree, Netlify Forms, etc.) on a static site. Adds JavaScript, a third-party dependency, and spam management. Overkill for a minimal showcase. | Not needed. This is a project directory, not a freelance portfolio. If contact is ever needed, a simple `mailto:` link suffices. |
| Blog / dynamic content | "Show thought leadership" | Fundamentally changes the site from a landing page to a content site. Requires ongoing maintenance, RSS, pagination, taxonomy. Hugo supports it, but it is scope creep. | Keep the site as a single-page showcase. If blogging is desired, host it on a separate subdomain (e.g., blog.toto-castaldi.com). |
| JavaScript animations / Three.js / scroll effects | "Modern portfolios have animations" | Adds bundle size, runtime dependencies, and accessibility issues (motion sensitivity). Contradicts "minimal clean design with strong typography." Animations on 3 project cards is gratuitous. | CSS transitions only where meaningful (e.g., subtle hover state on project links). `prefers-reduced-motion` media query to disable even those. |
| Dark mode toggle button | "Let users pick their theme" | Requires JavaScript for state management and localStorage persistence. Adds complexity to a zero-JS site. OS-level preference is sufficient for a landing page. | Use `prefers-color-scheme` CSS media query. Respects user choice without any JS. |
| Social media links / icons | "Connect with visitors" | Adds visual clutter to a minimal design. Icon fonts or SVG icon libraries add weight. Social links are expected on portfolios seeking clients -- this is a project directory. | Omit entirely per PROJECT.md out-of-scope. |
| Analytics / tracking scripts | "Know your visitors" | Third-party JS (Google Analytics, Plausible, etc.) adds load time, privacy concerns, and cookie banner requirements (especially for EU/Italian visitors under GDPR). | If analytics is ever needed, use privacy-respecting server-side analytics (e.g., GoatCounter, which is a single `<script>` tag with no cookies). But for now, omit. |
| Cookie consent banner | "GDPR compliance" | Only needed if you use cookies or tracking. A static HTML page with no JS, no analytics, no third-party requests sets zero cookies. Adding a cookie banner to a cookieless site is absurd. | Do not add cookies. No cookies = no banner needed. |
| Auto-detect language from browser | "Show the right language automatically" | Creates confusion when users land on unexpected language. Makes bookmarking/sharing unpredictable (same URL, different content). Complicates caching. Hugo's approach of separate URLs per language is cleaner. | Default language (Italian, since .com domain targets Italian audience first) with visible language switcher. Let the user choose. |
| Project detail pages | "Give each project its own page" | Each project already has its own full site on its subdomain. A detail page on the landing page would duplicate content and create maintenance burden for zero value. | Link directly to each project's subdomain. The landing page is a directory, not a CMS. |

## Feature Dependencies

```
[Semantic HTML]
    |
    +--enhances--> [Accessible contrast ratios]
    +--enhances--> [hreflang tags]
    +--enhances--> [Open Graph meta tags]
    +--requires--> [Proper <title> and meta per language]

[Hugo i18n setup]
    |
    +--requires--> [Language switcher]
    +--requires--> [hreflang tags]
    +--requires--> [Proper <title> and meta per language]
    +--requires--> [Canonical URLs per language]

[CSS custom properties]
    |
    +--enables--> [Dark mode support]
    +--enables--> [Typography scale]

[Responsive layout]
    +--independent (pure CSS, no dependencies)

[Favicon]
    +--independent (static asset)
```

### Dependency Notes

- **Hugo i18n setup is foundational:** Language switcher, hreflang, per-language meta, and canonical URLs all depend on Hugo's multilingual configuration being set up first. This must be the first thing built.
- **Semantic HTML enables SEO and accessibility:** Proper heading hierarchy and `lang` attributes make hreflang, Open Graph, and screen reader support work correctly.
- **CSS custom properties enable theming:** Dark mode and typography scale both benefit from CSS variables. Define them once, reference everywhere.
- **No JS dependencies exist:** Every feature listed can be implemented with HTML and CSS only. This is a deliberate architectural constraint.

## MVP Definition

### Launch With (v1)

Minimum viable product -- what is needed to validate the concept.

- [x] Project cards (name, description, link) for Docora, Lumio, Helix -- the core purpose
- [x] Hugo multilingual setup (IT/EN) with i18n string files -- foundational requirement
- [x] Language switcher in header -- bilingual visitors need to toggle
- [x] Responsive layout -- mobile visitors must be served
- [x] Semantic HTML structure -- baseline quality
- [x] Favicon -- prevents "broken" appearance in browser tabs
- [x] Proper `<title>` and meta description per language -- search engine baseline
- [x] HTTPS via GitHub Pages -- non-negotiable security baseline

### Add After Validation (v1.x)

Features to add once core is working and deployed.

- [ ] hreflang tags -- add when SEO matters, after Google indexes the site
- [ ] Open Graph meta tags -- add when the URL is being shared publicly
- [ ] Canonical URLs -- add alongside hreflang
- [ ] Dark mode CSS -- add when the base design is finalized and stable
- [ ] WCAG AA contrast audit -- verify after final color palette is chosen
- [ ] Print stylesheet -- add after visual design is locked

### Future Consideration (v2+)

Features to defer until the product proves its value.

- [ ] OG image (custom preview image for social sharing) -- requires graphic design effort
- [ ] Structured data / JSON-LD -- only if SEO becomes a priority
- [ ] Goatcounter analytics -- only if traffic measurement becomes needed
- [ ] Additional projects beyond the initial three -- add when new projects exist

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Project cards (name, desc, link) | HIGH | LOW | P1 |
| Hugo i18n setup (IT/EN) | HIGH | LOW | P1 |
| Language switcher | HIGH | LOW | P1 |
| Responsive layout | HIGH | LOW | P1 |
| Semantic HTML | HIGH | LOW | P1 |
| Favicon | MEDIUM | LOW | P1 |
| `<title>` + meta per language | MEDIUM | LOW | P1 |
| HTTPS (GitHub Pages) | HIGH | LOW | P1 |
| hreflang tags | MEDIUM | LOW | P2 |
| Open Graph meta tags | MEDIUM | LOW | P2 |
| Canonical URLs | MEDIUM | LOW | P2 |
| Dark mode (CSS only) | LOW | MEDIUM | P2 |
| WCAG AA contrast | MEDIUM | LOW | P2 |
| Typography scale | MEDIUM | LOW | P2 |
| Print stylesheet | LOW | LOW | P3 |
| OG image | LOW | MEDIUM | P3 |
| Structured data | LOW | MEDIUM | P3 |

**Priority key:**
- P1: Must have for launch -- the site does not function without these
- P2: Should have, add after initial deploy -- polish and professionalism
- P3: Nice to have, future consideration -- only if specific need arises

## Competitor Feature Analysis

| Feature | Simplefolio (GitHub template) | HugoBlox Landing Page | Introduction Theme | Our Approach |
|---------|-------------------------------|----------------------|-------------------|--------------|
| Project showcase | Cards with image, title, description, links to live site and source | Markdown-driven sections with customizable blocks | Full single-page with multiple sections | Three clean text-only cards. No images. Name + description + link. Maximally minimal. |
| Multilingual | Not supported | Not built-in (requires manual Hugo i18n) | Built-in multilingual support | First-class IT/EN with Hugo i18n. Language switcher in header. hreflang tags. |
| Dark mode | Not supported | Automatic dark mode | Light and dark themes | CSS `prefers-color-scheme` only. No toggle. Respect OS preference. |
| Typography | Standard Bootstrap typography | Theme-dependent | Theme-dependent | Intentional modular type scale. Typography IS the design. Two font families max. |
| JavaScript | jQuery required | Hugo Blox JS bundle | Minimal JS | Zero JavaScript. Pure HTML + CSS. |
| About section | Included by default | Included by default | Included by default | Deliberately excluded. Projects only. |
| Contact | Included (email link) | Contact form widget | Contact section | Deliberately excluded. |
| Animations | Fade-in on scroll | Theme animations | Smooth scrolling | None. Static content. CSS hover transitions only. |
| Performance | Moderate (jQuery + images) | Moderate (JS bundle) | Good (minimal JS) | Excellent (<1s load, no JS, no images, inline CSS possible) |

## Sources

- [Hostinger: 25 web developer portfolio examples](https://www.hostinger.com/tutorials/web-developer-portfolio) -- common portfolio features analysis
- [Smashing Magazine: Designing a Better Language Selector](https://www.smashingmagazine.com/2022/05/designing-better-language-selector/) -- flags vs text-based switcher best practices
- [Usersnap: Designing a Language Switch](https://usersnap.com/blog/design-language-switch/) -- language switcher UX patterns
- [Google: x-default hreflang for international pages](https://developers.google.com/search/blog/2013/04/x-default-hreflang-for-international-pages) -- hreflang implementation requirements
- [Google: Managing Multi-Regional Sites](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites) -- multilingual SEO rules
- [CSS-Tricks: Dark Mode Favicons](https://css-tricks.com/dark-mode-favicons/) -- SVG favicon dark mode technique
- [Joyofcode: Dark Mode Favicon](https://joyofcode.xyz/dark-mode-favicon) -- HTML-only favicon approach
- [Hugo: Multilingual mode](https://gohugo.io/content-management/multilingual/) -- official Hugo i18n documentation
- [victoriadrake/hugo-theme-introduction](https://github.com/victoriadrake/hugo-theme-introduction) -- minimal multilingual Hugo theme reference
- [HugoBlox Landing Page Template](https://hugoblox.com/templates/details/landing-page/) -- Hugo landing page feature set

---
*Feature research for: Minimal multilingual developer project showcase landing page*
*Researched: 2026-02-17*
