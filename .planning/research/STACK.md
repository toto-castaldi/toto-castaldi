# Stack Research

**Domain:** Multilingual static landing page (Hugo + GitHub Pages)
**Researched:** 2026-02-17
**Confidence:** HIGH

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Hugo (extended edition) | v0.155.3 | Static site generator | The fastest SSG available. Built-in multilingual support (i18n, content translation, language switcher). Built-in asset pipeline (Hugo Pipes) for CSS minification, fingerprinting, Sass transpilation -- no Webpack/Vite needed. Official docs recommend extended edition for Sass support. |
| Dart Sass | 1.97.3 | SCSS to CSS transpilation | Hugo's embedded LibSass was deprecated in v0.153.0. Dart Sass is the officially recommended replacement. Allows clean SCSS organization without any npm build step. |
| GitHub Actions | N/A | CI/CD pipeline | Hugo's official docs provide a complete workflow YAML for GitHub Pages deployment. Zero-config once the workflow file is committed. |
| GitHub Pages | N/A | Static hosting | Free hosting with custom domain support, automatic HTTPS via Let's Encrypt, and native GitHub Actions integration. Already chosen per project constraints. |

### CSS and Design

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Custom SCSS (no framework) | N/A | Styling | For a minimal landing page with 3 project cards, CSS frameworks like Bootstrap or Tailwind are overkill. Hand-written SCSS keeps the output under 5KB and gives full control over typography and whitespace. Hugo Pipes handles transpilation and minification natively. |
| System font stack | N/A | Typography | System fonts (-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, etc.) load instantly with zero network requests. For a "clean minimal design with strong typography," system fonts are the right default -- they feel native on every device and eliminate all font-loading performance issues. |
| CSS custom properties | N/A | Theming/variables | Modern CSS variables for colors, spacing, and typography scale. Supported by all browsers. Works alongside SCSS for compile-time and runtime flexibility. |

### Content and i18n

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Hugo i18n (filename suffix method) | Built-in | Content translation | Two methods exist: filename suffixes (index.it.md / index.en.md) or separate content directories. Filename suffix is simpler for a small site -- all translations live next to each other, no extra directory config needed. |
| Hugo i18n strings | Built-in | UI string translation | i18n/en.yaml and i18n/it.yaml files provide translated UI strings (nav labels, language switcher text). Built into Hugo, no external library needed. |

### Infrastructure

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Git | latest | Version control | Required by Hugo and GitHub Pages. No submodules needed since we build a custom theme inline (not importing an external theme). |
| CNAME file (in static/) | N/A | Custom domain | A static/CNAME file containing "toto-castaldi.com" is copied to the build output root. GitHub Pages uses this to serve the custom domain. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Hugo dev server (`hugo server`) | Local development with live reload | Rebuilds in milliseconds. Run with `--navigateToChanged` for automatic navigation to changed pages. |
| Hugo CLI (`hugo new`) | Scaffold content files | Use `hugo new content` to create properly templated content files. |

## Configuration Approach

### Config format: TOML (`hugo.toml`)

TOML is Hugo's default and most widely documented format. Community examples, themes, and tutorials overwhelmingly use TOML. Use `hugo.toml` for the main config.

### Multilingual config structure

```toml
baseURL = 'https://toto-castaldi.com/'
defaultContentLanguage = 'it'
defaultContentLanguageInSubdir = false

[languages]
  [languages.it]
    languageName = 'Italiano'
    weight = 1
    title = 'Toto Castaldi'
  [languages.en]
    languageName = 'English'
    weight = 2
    title = 'Toto Castaldi'
```

Italian is the default language (no URL prefix). English content lives under `/en/`. This is correct because the domain owner is Italian and Italian visitors are the primary audience.

### Theme approach: Custom inline theme (no external dependency)

For a 1-page landing site, create layouts directly in the project's `layouts/` directory rather than using an external theme. This avoids git submodule complexity, Hugo Module/Go dependency, and theme update breakage. The entire site needs only 3-4 template files.

## Installation

```bash
# Install Hugo extended on Linux (snap)
sudo snap install hugo

# Or download specific version (more reproducible)
curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v0.155.3/hugo_extended_0.155.3_linux-amd64.tar.gz"

# Create new site
hugo new site toto-castaldi
cd toto-castaldi

# Initialize git
git init

# Create the CNAME for custom domain
echo "toto-castaldi.com" > static/CNAME

# Start dev server
hugo server
```

No `npm install` required. Hugo handles everything natively.

## GitHub Actions Workflow

The official Hugo workflow (`.github/workflows/hugo.yaml`) handles:
- Installing Hugo extended v0.155.3, Dart Sass 1.97.3, Go 1.25.6
- Building with `hugo --gc --minify`
- Deploying to GitHub Pages via `actions/deploy-pages@v4`
- Caching the Hugo build cache between runs

Key actions versions:
- `actions/checkout@v6`
- `actions/setup-go@v6`
- `actions/configure-pages@v5`
- `actions/upload-pages-artifact@v3`
- `actions/deploy-pages@v4`

Prerequisites: Set repository Settings > Pages > Source to "GitHub Actions".

## Alternatives Considered

| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| Custom inline layouts | External Hugo theme (Ananke, PaperMod, etc.) | For 1 page with 3 cards, a full theme adds unnecessary complexity, overrides to fight, and a git submodule or Hugo Module dependency. Write ~100 lines of templates instead. |
| System font stack | Google Fonts (self-hosted WOFF2) | System fonts are faster (zero network requests), GDPR-compliant by default, and provide excellent typography. If the design later demands a specific typeface, self-host WOFF2 files in static/fonts/ -- never use the Google Fonts CDN (privacy, GDPR, extra DNS lookup). |
| SCSS via Dart Sass | Plain CSS | SCSS provides variables, nesting, and partials that make even small stylesheets more maintainable. Hugo transpiles it natively -- no build tooling overhead. |
| SCSS via Dart Sass | Tailwind CSS | Tailwind requires Node.js, PostCSS config, and a build step. For 3 cards and a language switcher, it adds ~10x more tooling complexity than writing 80 lines of SCSS. |
| TOML config | YAML config | Both work. TOML is Hugo's default, has more community examples, and avoids YAML's indentation pitfalls. |
| Filename suffix i18n | Content directory i18n | Filename suffix (index.it.md, index.en.md) keeps translations visually paired. Content directories are better for large sites with 50+ pages; overkill for a single landing page. |
| No external theme | Hugo Module (Go-based theme management) | Hugo Modules require Go installed locally and in CI. For a project with zero external theme dependencies, this adds unnecessary complexity. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Hugo standard edition | Cannot transpile SCSS without Dart Sass installed separately. Extended edition bundles the capability, and the official GitHub Actions workflow installs extended. | Hugo extended edition |
| LibSass (embedded) | Deprecated in Hugo v0.153.0, will be removed in a future release. | Dart Sass 1.97.3 |
| Git submodules for themes | Fragile, confusing clone commands, traces left in multiple repo files on removal. Hugo community consensus: "sort of anti-pattern." | Inline layouts or Hugo Modules |
| Google Fonts CDN | Extra DNS lookup, potential GDPR issues (EU court rulings), render-blocking resource. | System font stack, or self-hosted WOFF2 if a specific typeface is needed |
| Tailwind CSS | Requires Node.js, PostCSS, purge config. Massive tooling overhead for a page with 3 cards. | Custom SCSS (< 100 lines) |
| Bootstrap | 200KB+ framework for 3 cards and a nav. Absurd overhead. Forces you to fight framework defaults for "minimal" design. | Custom SCSS |
| npm/Node.js build step | Hugo handles SCSS, minification, and fingerprinting natively. Adding Node.js to the build doubles CI complexity for zero benefit in this project. | Hugo Pipes |
| `config.toml` (old name) | Hugo renamed the config file to `hugo.toml` (with hugo prefix). Old name still works but new projects should use the current convention. | `hugo.toml` |

## Stack Patterns by Variant

**If the design later needs a custom web font:**
- Download WOFF2 files from Google Fonts
- Place in `static/fonts/`
- Reference via `@font-face` in SCSS
- Use `font-display: swap` to prevent invisible text during load
- Because: self-hosting avoids GDPR issues and external dependencies

**If the project grows beyond a single landing page:**
- Consider migrating inline layouts to a proper theme directory (`themes/toto/`)
- Consider Hugo Modules if importing external components
- Because: separation of concerns becomes valuable at 5+ template files

**If JavaScript is needed later (unlikely for this project):**
- Hugo's built-in `js.Build` uses esbuild for bundling, tree-shaking, and minification
- No Webpack/Vite needed
- Because: Hugo Pipes handles JS natively, same as CSS

## Version Compatibility

| Component | Version | Compatible With | Notes |
|-----------|---------|-----------------|-------|
| Hugo extended | v0.155.3 | Dart Sass 1.97.3 | Official docs confirm this pairing |
| Hugo extended | v0.155.3 | Go 1.25.6 | Required only if using Hugo Modules (we are not) |
| GitHub Actions workflow | Current | actions/checkout@v6, actions/deploy-pages@v4 | Versions from official Hugo docs (Feb 2026) |
| Hugo | v0.146.0+ | Multilingual filename suffix i18n | Minimum version per quick-start docs; we exceed this |

## DNS Configuration for Custom Domain

GitHub Pages requires these DNS records for apex domain:

| Type | Name | Value |
|------|------|-------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | toto-castaldi.github.io |

GitHub auto-provisions Let's Encrypt SSL once DNS resolves.

## Sources

- [Hugo official docs: Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/) -- Complete GitHub Actions workflow, version recommendations (HIGH confidence)
- [Hugo official docs: Multilingual mode](https://gohugo.io/content-management/multilingual/) -- i18n configuration, translation methods, language switcher (HIGH confidence)
- [Hugo official docs: Configuration introduction](https://gohugo.io/configuration/introduction/) -- Config format, TOML default (HIGH confidence)
- [Hugo official docs: Hugo Pipes](https://gohugo.io/hugo-pipes/) -- Asset pipeline capabilities: minify, fingerprint, SCSS, JS (HIGH confidence)
- [Hugo GitHub releases](https://github.com/gohugoio/hugo/releases) -- v0.155.3 released Feb 8, 2026 (HIGH confidence)
- [Hugo official docs: Installation (Linux)](https://gohugo.io/installation/linux/) -- Editions: standard, extended, extended/deploy (HIGH confidence)
- [Hugo official docs: css.Sass](https://gohugo.io/functions/css/sass/) -- LibSass deprecated in v0.153.0, Dart Sass recommended (HIGH confidence)
- [Hugo discourse: Modules vs submodules](https://discourse.gohugo.io/t/theme-as-a-hugo-module-or-theme-as-a-git-submodule/54388) -- Community consensus on submodules as anti-pattern (MEDIUM confidence)
- [GitHub docs: Custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) -- DNS A records, CNAME setup (HIGH confidence)

---
*Stack research for: toto-castaldi.com multilingual Hugo landing page*
*Researched: 2026-02-17*
