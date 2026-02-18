# Phase 1: Foundation and Deploy Pipeline - Research

**Researched:** 2026-02-18
**Domain:** Hugo multilingual site scaffolding, GitHub Pages deployment, custom domain DNS
**Confidence:** HIGH

## Summary

Phase 1 establishes the Hugo project skeleton, bilingual URL structure, GitHub Actions CI/CD pipeline, and custom domain with HTTPS. The critical "Day 0" decision is the `defaultContentLanguageInSubdir` setting, which determines the permanent URL structure for both languages.

Research confirms that Hugo v0.155.3 (latest, Feb 2026) with `defaultContentLanguageInSubdir = true` is the correct approach for this project, giving symmetric `/it/` and `/en/` URL paths. Hugo generates a meta-refresh redirect at `/` pointing to the default language subdirectory. A known Hugo issue (open since 2018) means no `404.html` is generated at root when this setting is true -- a `static/404.html` workaround is needed for GitHub Pages. The GitHub Actions workflow from Hugo's official docs must be modified to remove the `--baseURL` flag (which injects the repo name for project sites with custom domains).

**Primary recommendation:** Set `defaultContentLanguage = 'it'` with `defaultContentLanguageInSubdir = true`, hardcode `baseURL` in `hugo.toml`, place CNAME in `static/`, and use a static 404.html at root as a workaround.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Default language: Claude's discretion (choose based on best practices for multilingual Hugo + GitHub Pages)
- URL structure (defaultContentLanguageInSubdir): Claude's discretion -- pick the approach that works best for this stack and future phases
- This is a Day 0 decision -- cannot be cheaply changed later
- Root URL (`/`) behavior: Claude's discretion -- pick the best approach for Hugo static sites on GitHub Pages
- 404 page: Claude's discretion -- handle appropriately for Phase 1
- Brief downtime during DNS cutover from current host to GitHub Pages is acceptable
- Subdomains (docora/lumio/helix.toto-castaldi.com) are already handled separately -- do not touch them
- Placeholder must be in both Italian and English -- validates bilingual routing works (Phase 1 success criterion)
- Content and styling: Claude's discretion -- whatever minimal content makes sense
- Display name: "Antonio Toto Castaldi"

### Claude's Discretion
- Default language choice (it vs en)
- URL structure (both in subdirectories vs default at root)
- Root URL redirect behavior
- 404 page design and behavior
- Placeholder page content and styling beyond the name
- All technical implementation (Hugo config, GitHub Actions, DNS setup)

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| INFR-01 | Site generated with Hugo and deployed to GitHub Pages via GitHub Actions | Official Hugo docs workflow verified via Context7; must remove `--baseURL` flag for custom domain project sites; use Hugo extended v0.155.3 |
| INFR-02 | Custom domain toto-castaldi.com and www.toto-castaldi.com with HTTPS | DNS A records (4 IPs) + CNAME for www -> toto-castaldi.github.io; set custom domain in GitHub Settings FIRST, then configure DNS |
| INFR-03 | CNAME file in `static/` for custom domain persistence across deploys | Verified critical: GitHub Actions artifact-based deploy wipes CNAME on every push if not in `static/` |
| INFR-04 | Hugo multilingual configuration with Italian and English (filename-based i18n) | Filename-suffix method (`_index.it.md`, `_index.en.md`) confirmed as correct for small sites; Context7 shows exact config pattern |
| INFR-05 | `defaultContentLanguageInSubdir` set correctly as Day 0 decision | Researched thoroughly: recommend `true` for symmetric `/it/` and `/en/` paths; root `/` gets meta-refresh redirect to `/it/`; requires `static/404.html` workaround |
</phase_requirements>

## Discretion Recommendations

These are the areas where the user granted full discretion. Each recommendation includes the reasoning and evidence.

### Default Language: Italian (`it`)

**Recommendation:** `defaultContentLanguage = 'it'`

**Reasoning:**
- The domain owner is Italian ("Antonio Toto Castaldi") and the domain is `.com` (not `.it`, but the primary audience is Italian)
- Italian visitors are the primary audience; English is the secondary/international option
- Hugo uses the default language as the fallback when translations are missing -- Italian content should never be missing
- The `weight` parameter controls menu ordering; Italian (weight 1) appears first
- Prior research in `.planning/research/STACK.md` already recommended Italian as default

**Confidence:** HIGH -- this is a straightforward audience-based decision.

### URL Structure: `defaultContentLanguageInSubdir = true`

**Recommendation:** Set `defaultContentLanguageInSubdir = true` so both languages live in subdirectories.

**Resulting URL structure:**
- `/it/` -- Italian home page
- `/en/` -- English home page
- `/` -- meta-refresh redirect to `/it/`

**Reasoning:**
- **Symmetric URLs:** Both languages get the same URL pattern (`/it/...` and `/en/...`). This makes the language switcher logic uniform -- every page always has a language prefix.
- **Clean hreflang implementation (Phase 3):** Symmetric paths make reciprocal hreflang tags straightforward. No special-casing for the default language at root.
- **Future-proof:** Adding a third language requires zero structural changes. The pattern scales.
- **Avoids the "ghost /it/ 404" problem:** When `defaultContentLanguageInSubdir = false` and Italian is default, there is no `/it/` path at all. Users and bots trying `/it/` get a 404. With `true`, `/it/` exists.
- **Trade-off acknowledged:** Root `/` serves a meta-refresh redirect page instead of content. This adds one redirect hop (~0ms in practice since it's `content="0"`), and the redirect page has no useful content for SEO. This is acceptable because the root URL will not be the canonical URL for either language.

**Alternative considered and rejected:** `defaultContentLanguageInSubdir = false` (Italian at root, English at `/en/`). Rejected because it creates asymmetric URLs, makes the language switcher harder to implement, and causes `/it/` to 404.

**Confidence:** HIGH -- verified against Hugo docs (Context7), community consensus, and prior pitfalls research.

### Root URL Behavior: Meta-refresh redirect to `/it/`

**Recommendation:** Accept Hugo's default behavior: when `defaultContentLanguageInSubdir = true`, Hugo generates a root `index.html` containing a `<meta http-equiv="refresh" content="0; url=/it/">` redirect.

**The redirect HTML Hugo generates:**
```html
<!DOCTYPE html>
<html lang="it">
  <head>
    <title>https://toto-castaldi.com/it/</title>
    <link rel="canonical" href="https://toto-castaldi.com/it/">
    <meta charset="utf-8">
    <meta http-equiv="refresh" content="0; url=https://toto-castaldi.com/it/">
  </head>
</html>
```

**Key properties:**
- Instant redirect (`content="0"` = zero delay)
- Includes canonical link pointing to the Italian homepage
- No JavaScript required
- Works on all browsers including those with JS disabled
- Hugo 0.140.0+ added `disableDefaultLanguageRedirect` to optionally disable this, but we WANT the redirect

**Confidence:** HIGH -- verified via Hugo source template in official docs.

### 404 Page: Static bilingual 404.html

**Recommendation:** Place a hand-crafted `static/404.html` in the Hugo source that contains both Italian and English text.

**Why this is needed:**
- Hugo issue #5161 (open since 2018, still open as of Aug 2024): when `defaultContentLanguageInSubdir = true`, Hugo generates `404.html` only inside language subdirectories (`/it/404.html`, `/en/404.html`), NOT at root level.
- GitHub Pages requires `/404.html` at root to serve custom error pages.
- Without it, GitHub Pages shows its default 404 page.

**Implementation approach:**
- Create `static/404.html` with minimal bilingual content (both Italian and English text, since we cannot know which language the user was trying to reach)
- Keep it simple for Phase 1; Phase 2 can add styling
- Include links to both `/it/` and `/en/` so users can navigate to a valid page

**Alternative considered:** Using `hugo && cp public/it/404.html public/404.html` as a post-build step in GitHub Actions. Rejected because it couples the build script to implementation details and the Italian-only 404 would confuse English users hitting a bad URL.

**Confidence:** HIGH -- verified via GitHub issue #5161 and multiple community workarounds.

### Placeholder Content: Minimal bilingual name card

**Recommendation:** A minimal placeholder page showing "Antonio Toto Castaldi" with a brief line indicating the site is under construction, in the appropriate language.

**Italian (`_index.it.md`):**
- Title: "Antonio Toto Castaldi"
- Body: Brief text indicating the site is being built

**English (`_index.en.md`):**
- Title: "Antonio Toto Castaldi"
- Body: Brief text indicating the site is being built

**Styling:** Minimal inline CSS or a tiny `assets/css/main.css` processed through Hugo Pipes. Center-aligned, clean typography using system font stack. Just enough to confirm the site renders and the CSS pipeline works.

**Confidence:** HIGH -- minimal content, validates bilingual routing.

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| Hugo extended | v0.155.3 | Static site generator | Latest stable (Feb 8, 2026). Extended edition required for SCSS. Official docs workflow uses it. |
| Dart Sass | 1.97.3 | SCSS transpilation | Latest stable (Jan 21, 2026). Hugo deprecated embedded LibSass in v0.153.0. |
| GitHub Actions | N/A | CI/CD pipeline | Official Hugo docs provide complete workflow YAML. Artifact-based deploy via `actions/deploy-pages@v4`. |
| GitHub Pages | N/A | Static hosting | Free, custom domain support, auto HTTPS via Let's Encrypt. |

### Supporting (Phase 1 only)

| Component | Purpose | Notes |
|-----------|---------|-------|
| `static/CNAME` | Custom domain persistence | Contains `toto-castaldi.com` -- copied to build output root on every deploy |
| `static/404.html` | Root-level 404 page | Workaround for Hugo issue #5161 with `defaultContentLanguageInSubdir = true` |

### GitHub Actions Versions (from Hugo official docs via Context7)

| Action | Version | Purpose |
|--------|---------|---------|
| `actions/checkout` | `@v5` | Clone repo with submodules and full history |
| `actions/setup-go` | `@v5` | Install Go (only needed for Hugo Modules, but included in official workflow) |
| `actions/configure-pages` | `@v5` | Configure GitHub Pages deployment |
| `actions/cache/restore` + `actions/cache/save` | `@v4` | Cache Hugo build artifacts between runs |
| `actions/upload-pages-artifact` | `@v3` | Upload built site as deployable artifact |
| `actions/deploy-pages` | `@v4` | Deploy artifact to GitHub Pages |

## Architecture Patterns

### Recommended Phase 1 Project Structure

```
toto-castaldi/
+-- .github/
|   +-- workflows/
|       +-- hugo.yaml              # Build and deploy pipeline
+-- config/
|   +-- _default/
|       +-- hugo.toml              # Site-wide: baseURL, defaultContentLanguage
|       +-- languages.toml         # Language definitions (it, en)
+-- content/
|   +-- _index.it.md              # Italian placeholder page
|   +-- _index.en.md              # English placeholder page
+-- i18n/
|   +-- it.toml                   # Italian UI strings (minimal for Phase 1)
|   +-- en.toml                   # English UI strings (minimal for Phase 1)
+-- layouts/
|   +-- _default/
|   |   +-- baseof.html           # Base HTML shell
|   +-- index.html                # Home page template
|   +-- partials/
|       +-- head.html             # <head> tag (meta, CSS)
+-- assets/
|   +-- css/
|       +-- main.css              # Minimal stylesheet (validates Hugo Pipes)
+-- static/
|   +-- CNAME                     # Custom domain: toto-castaldi.com
|   +-- 404.html                  # Bilingual 404 page (workaround)
+-- .planning/                    # Planning docs (not published)
```

### Pattern: Split Configuration (`config/_default/`)

Hugo supports splitting configuration across multiple files in `config/_default/`. For Phase 1:

- `hugo.toml` -- site-wide settings (`baseURL`, `defaultContentLanguage`, `defaultContentLanguageInSubdir`, `title`)
- `languages.toml` -- language definitions with `languageName`, `languageCode`, `weight`, `title`

This separation keeps concerns isolated and makes Phase 2 additions (menus, params) clean.

### Pattern: Filename-based Translation

Content files use language suffixes: `_index.it.md` and `_index.en.md`. Hugo automatically links them as translations. For Phase 1, only the home page exists.

### Pattern: Hugo Pipes for CSS

CSS in `assets/css/main.css` is processed through Hugo Pipes:
```html
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```

This validates the asset pipeline works from Phase 1, avoiding surprises when Phase 2 adds real styling.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Root-level redirect | Custom JavaScript redirect at `/` | Hugo's built-in alias redirect (auto-generated with `defaultContentLanguageInSubdir = true`) | Hugo handles it automatically with proper canonical link |
| CSS minification | Manual minification or external tools | Hugo Pipes `resources.Minify` | Built into Hugo, zero config |
| Cache-busting filenames | Manual versioning or query strings | Hugo Pipes `resources.Fingerprint` | Content-hash filenames, automatic |
| Deployment pipeline | Custom shell scripts deploying to gh-pages branch | `actions/deploy-pages@v4` (artifact-based) | Official GitHub approach, avoids gh-pages branch management |
| HTTPS certificates | Manual Let's Encrypt setup | GitHub Pages auto-provisioning | Automatic after DNS propagation |

## Common Pitfalls

### Pitfall 1: `--baseURL` Flag Injects Repository Name

**What goes wrong:** The official Hugo workflow includes `--baseURL "${{ steps.pages.outputs.base_url }}/"` in the build step. For project sites (repos NOT named `username.github.io`), `configure-pages` outputs the URL with the repo name appended (e.g., `https://toto-castaldi.github.io/toto-castaldi`). With a custom domain, this breaks all internal links -- CSS 404s, wrong link paths, etc.

**Why it happens:** The `configure-pages` action is unaware of custom domain configuration during the build phase.

**How to avoid:** Remove the `--baseURL` flag from the Hugo build command in the GitHub Actions workflow. Hardcode `baseURL = 'https://toto-castaldi.com/'` in `hugo.toml`. The build uses this value directly.

**Warning signs:** Site works locally but CSS/assets 404 on deployed site. URLs contain repo name in path segments.

**Confidence:** HIGH -- verified via Hugo community report (discourse.gohugo.io/t/56404) and jmooring's (Hugo maintainer) recommendation.

### Pitfall 2: CNAME File Wiped on Deployment

**What goes wrong:** Each artifact-based deployment replaces the entire site. If CNAME was only set via GitHub Settings UI, it disappears on push.

**How to avoid:** `static/CNAME` containing `toto-castaldi.com` (no protocol, no trailing newline). Hugo copies it to build output root.

**Confidence:** HIGH -- verified in prior pitfalls research and GitHub community discussions.

### Pitfall 3: No Root 404.html with `defaultContentLanguageInSubdir = true`

**What goes wrong:** Hugo generates `/it/404.html` and `/en/404.html` but NOT `/404.html`. GitHub Pages requires root-level `404.html`.

**How to avoid:** Place a hand-crafted `static/404.html` that covers both languages.

**Confidence:** HIGH -- verified via Hugo issue #5161 (open, confirmed behavior in v0.155.x).

### Pitfall 4: DNS Order-of-Operations

**What goes wrong:** Setting DNS records BEFORE adding custom domain in GitHub Settings creates a domain takeover window. Setting them in wrong order causes HTTPS certificate failure.

**How to avoid:** Exact order:
1. Add custom domain in GitHub repo Settings > Pages
2. Create DNS A records for apex (4 IPs)
3. Create CNAME record: `www` -> `toto-castaldi.github.io`
4. Wait for propagation (up to 24h)
5. Enable "Enforce HTTPS" once checkbox appears

**Warning signs:** "Enforce HTTPS" greyed out, cert warnings, `dig` shows wrong IPs.

**Confidence:** HIGH -- verified via GitHub official docs.

### Pitfall 5: Existing DNS Records Conflict

**What goes wrong:** The domain is currently hosted elsewhere. Old A records, CNAME records, or other DNS entries may conflict with the new GitHub Pages records.

**How to avoid:** Before adding GitHub Pages DNS records:
1. Remove any existing A records for the apex domain (`@`)
2. Remove any existing CNAME for `www`
3. Do NOT use wildcard DNS (`*.toto-castaldi.com`) -- GitHub docs explicitly warn this creates "immediate risk of domain takeovers"
4. Leave subdomain records (docora, lumio, helix) untouched

**Confidence:** HIGH -- explicitly warned in GitHub docs.

### Pitfall 6: GitHub Pages Source Not Set to "GitHub Actions"

**What goes wrong:** By default, GitHub Pages source may be set to "Deploy from a branch." The artifact-based workflow requires "GitHub Actions" as the source.

**How to avoid:** Before first push, go to repo Settings > Pages > Source and change to "GitHub Actions."

**Confidence:** HIGH -- documented in Hugo official deployment guide.

## Code Examples

### hugo.toml (Phase 1)

```toml
# config/_default/hugo.toml
baseURL = 'https://toto-castaldi.com/'
title = 'Antonio Toto Castaldi'
defaultContentLanguage = 'it'
defaultContentLanguageInSubdir = true
```

Source: Hugo official docs via Context7 (`/gohugoio/hugodocs`), adapted for this project.

### languages.toml (Phase 1)

```toml
# config/_default/languages.toml
[it]
  languageCode = 'it'
  languageName = 'Italiano'
  title = 'Antonio Toto Castaldi'
  weight = 1

[en]
  languageCode = 'en'
  languageName = 'English'
  title = 'Antonio Toto Castaldi'
  weight = 2
```

Source: Hugo official docs via Context7.

### baseof.html (Phase 1 minimal)

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}">
<head>
  {{ partial "head.html" . }}
</head>
<body>
  <main>
    {{ block "main" . }}{{ end }}
  </main>
</body>
</html>
```

Source: Hugo template docs, standard base template pattern.

### head.html partial (Phase 1 minimal)

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{ .Title }}</title>
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```

Source: Hugo Pipes docs, standard asset processing pattern.

### index.html (home page template)

```html
{{ define "main" }}
<section>
  <h1>{{ .Title }}</h1>
  {{ .Content }}
</section>
{{ end }}
```

### GitHub Actions Workflow (modified for custom domain)

```yaml
# .github/workflows/hugo.yaml
name: Build and deploy

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.97.3
      HUGO_VERSION: 0.155.3
    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "$HOME/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "$HOME/.local/dart-sass" >> "$GITHUB_PATH"
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir -p "$HOME/.local/hugo"
          tar -C "$HOME/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "$HOME/.local/hugo" >> "$GITHUB_PATH"
      - name: Build
        run: hugo --gc --minify
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Key differences from official starter workflow:**
1. **No `--baseURL` flag** in the build step (critical for custom domain project sites)
2. **No Go setup** (not using Hugo Modules)
3. **No Node.js setup** (no npm dependencies)
4. **No submodules** (no external theme)
5. **No cache steps** (simplicity for Phase 1; can add later if builds are slow)
6. Pinned Hugo and Dart Sass versions for reproducibility

### static/CNAME

```
toto-castaldi.com
```

One line, no protocol prefix, no trailing newline.

### static/404.html (bilingual)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>404</title>
  <style>
    body { font-family: system-ui, -apple-system, sans-serif; text-align: center; padding: 4rem 1rem; color: #333; }
    h1 { font-size: 2rem; margin-bottom: 1rem; }
    p { margin: 0.5rem 0; }
    a { color: #0066cc; }
    hr { margin: 2rem auto; max-width: 200px; border: none; border-top: 1px solid #ddd; }
  </style>
</head>
<body>
  <h1>404</h1>
  <p>Pagina non trovata.</p>
  <p><a href="/it/">Vai alla home</a></p>
  <hr>
  <p>Page not found.</p>
  <p><a href="/en/">Go to home</a></p>
</body>
</html>
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `peaceiris/actions-gh-pages` (branch deploy) | `actions/deploy-pages@v4` (artifact deploy) | 2023+ | No gh-pages branch needed; cleaner workflow |
| LibSass (embedded in Hugo) | Dart Sass (external) | Hugo v0.153.0 | Must install Dart Sass separately in CI |
| `config.toml` (old name) | `hugo.toml` (new name) | Hugo v0.110.0+ | New convention; old name still works |
| Single config file | Split `config/_default/` directory | Long-standing | Better organization for multilingual sites |
| No `disableDefaultLanguageRedirect` | Available since Hugo v0.140.0 | Dec 2024 | Can disable root redirect if desired (we don't want this) |

## DNS Configuration

### Required Records

| Type | Name | Value | Purpose |
|------|------|-------|---------|
| A | @ | 185.199.108.153 | Apex domain -> GitHub Pages |
| A | @ | 185.199.109.153 | Apex domain -> GitHub Pages |
| A | @ | 185.199.110.153 | Apex domain -> GitHub Pages |
| A | @ | 185.199.111.153 | Apex domain -> GitHub Pages |
| CNAME | www | toto-castaldi.github.io | www subdomain -> GitHub Pages |

### Optional IPv6 Records

| Type | Name | Value |
|------|------|-------|
| AAAA | @ | 2606:50c0:8000::153 |
| AAAA | @ | 2606:50c0:8001::153 |
| AAAA | @ | 2606:50c0:8002::153 |
| AAAA | @ | 2606:50c0:8003::153 |

### Records to NOT Touch

Existing DNS records for subdomains `docora`, `lumio`, and `helix` must be left untouched.

### Records to Remove

Any existing A records or CNAME records for the apex domain (`@`) and `www` that point to the current (non-GitHub) hosting provider must be removed before adding the new records.

## Open Questions

1. **Current DNS provider identity**
   - What we know: The domain is currently hosted elsewhere (not GitHub Pages)
   - What's unclear: Which DNS provider manages toto-castaldi.com (Cloudflare, Route53, registrar DNS, etc.)
   - Recommendation: User must provide DNS provider access; the planner should include a manual step for DNS changes

2. **Existing `toto-castaldi.github.io` site**
   - What we know: No `toto-castaldi.github.io` repo exists
   - What's unclear: Whether GitHub Pages has ever been enabled on the `toto-castaldi/toto-castaldi` repo before
   - Recommendation: Check repo Settings > Pages before first deploy; may need to configure source to "GitHub Actions"

3. **Subdomain DNS record types**
   - What we know: Subdomains docora/lumio/helix are working and must not be touched
   - What's unclear: Whether they use CNAME or A records; whether a wildcard record exists
   - Recommendation: Before changing DNS, audit all existing records; remove any wildcard (`*`) record to prevent domain takeover risk

## Sources

### Primary (HIGH confidence)
- `/gohugoio/hugodocs` (Context7) -- multilingual configuration, `defaultContentLanguageInSubdir`, `disableDefaultLanguageRedirect`, GitHub Actions workflow, alias redirect template
- [Hugo official docs: Configure languages](https://gohugo.io/configuration/languages/) -- `defaultContentLanguageInSubdir`, `disableDefaultLanguageRedirect` parameter definitions
- [Hugo official docs: Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/) -- Complete GitHub Actions workflow YAML
- [Hugo official docs: Custom 404 page](https://gohugo.io/templates/404/) -- 404 template naming and output location
- [Hugo official docs: URL management](https://gohugo.io/content-management/urls/) -- Alias redirect template (meta-refresh HTML)
- [GitHub docs: Managing custom domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) -- DNS records, order of operations, wildcard warnings, HTTPS enforcement
- [Hugo GitHub releases](https://github.com/gohugoio/hugo/releases) -- v0.155.3 confirmed as latest (Feb 8, 2026)
- [Dart Sass releases](https://github.com/sass/dart-sass/releases) -- v1.97.3 confirmed as latest (Jan 21, 2026)

### Secondary (MEDIUM confidence)
- [Hugo Discourse: baseURL with GitHub Actions custom domain](https://discourse.gohugo.io/t/problem-with-baseurl-with-github-action/56404) -- jmooring (Hugo maintainer) confirms removing `--baseURL` flag for custom domains
- [Hugo Discourse: defaultContentLanguageInSubdir gotchas](https://discourse.gohugo.io/t/defaultcontentlanguageinsubdir/4658) -- Community experience with URL structure decisions
- [Hugo Discourse: Root page for multilingual site](https://discourse.gohugo.io/t/root-page-for-a-multilingual-site/53059) -- Approaches to root URL handling
- [Cyrus Yip: Top-level 404 for multilingual Hugo](https://cyrusyip.org/en/posts/2023/11/06/hugo-top-level-404/) -- Three workarounds for missing root 404.html

### Tertiary (LOW confidence -- for awareness only)
- [Hugo issue #5161](https://github.com/gohugoio/hugo/issues/5161) -- 404.html not generated at root with `defaultContentLanguageInSubdir = true` (open issue, still unresolved)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- Hugo v0.155.3 and Dart Sass 1.97.3 versions confirmed via official releases; workflow verified via Context7
- Architecture: HIGH -- URL structure patterns verified via Context7 and official docs; split config pattern standard
- Pitfalls: HIGH -- all pitfalls verified against official docs or Hugo maintainer guidance; the `--baseURL` pitfall is particularly well-documented
- DNS setup: HIGH -- exact IPs and records from GitHub official docs; order of operations verified
- 404 workaround: HIGH -- Hugo issue is open and confirmed; `static/404.html` workaround verified by multiple sources

**Research date:** 2026-02-18
**Valid until:** 2026-03-18 (30 days -- Hugo and GitHub Pages are stable)
