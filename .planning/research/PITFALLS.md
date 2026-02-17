# Pitfalls Research

**Domain:** Hugo multilingual landing page on GitHub Pages (IT/EN, custom domain)
**Researched:** 2026-02-17
**Confidence:** HIGH (verified against official Hugo docs + GitHub Pages docs + community reports)

## Critical Pitfalls

### Pitfall 1: CNAME File Wiped on Every Deployment

**What goes wrong:**
Each GitHub Actions deployment rebuilds the `public/` directory from scratch. If the CNAME file only exists in the gh-pages branch (or was added via GitHub's Settings UI), it gets overwritten on every push, causing the custom domain (`toto-castaldi.com`) to detach. The site reverts to `username.github.io` and HTTPS breaks.

**Why it happens:**
GitHub Pages adds a CNAME file to the deployment branch when you set the custom domain through the UI. Developers assume this persists across deployments. It does not -- the next Hugo build replaces the entire deployment artifact.

**How to avoid:**
Place a `static/CNAME` file in the Hugo source repository containing `toto-castaldi.com` (just the domain, no protocol, no trailing newline). Hugo copies everything from `static/` to the root of the build output, so the CNAME survives every deployment.

```
# static/CNAME (entire file contents)
toto-castaldi.com
```

**Warning signs:**
- Custom domain disappears from GitHub Pages settings after a push
- Site loads at `username.github.io` instead of the custom domain
- HTTPS enforcement checkbox becomes unchecked

**Phase to address:** Phase 1 (project scaffolding) -- must be present from the very first deployment.

---

### Pitfall 2: baseURL Mismatch Breaks All Asset Paths

**What goes wrong:**
CSS, JavaScript, and images all 404 on the deployed site while working perfectly on `localhost:1313`. The page loads as unstyled raw HTML. This is the single most reported Hugo + GitHub Pages problem in community forums.

**Why it happens:**
Hugo defaults `baseURL` to `http://localhost:1313/` during development. If the production `baseURL` is missing, wrong, or lacks a trailing slash, all absolute paths in templates resolve incorrectly. Common mistakes: setting `baseURL = "https://toto-castaldi.com"` (missing trailing slash), or leaving it as the default.

**How to avoid:**
Set `baseURL` in `hugo.toml` to the exact production URL with trailing slash:

```toml
baseURL = "https://toto-castaldi.com/"
```

Use `relURL` and `absURL` template functions for all asset references. Never hardcode paths. The trailing slash is explicitly recommended by Hugo's official documentation.

**Warning signs:**
- Site works locally but is unstyled on GitHub Pages
- Browser dev console shows 404 for CSS/JS files
- Asset URLs contain `localhost` or have doubled path segments

**Phase to address:** Phase 1 (project scaffolding) -- must be correct before first deployment.

---

### Pitfall 3: defaultContentLanguageInSubdir Misconfiguration Creates Broken URL Structure

**What goes wrong:**
With `defaultContentLanguageInSubdir = false` (the default), Italian content lives at `/` and English at `/en/`. This seems intuitive but creates problems: the root URL has no language prefix, making the language switcher asymmetric, and Hugo does not generate redirect stubs for the default language's sections. Visitors hitting `/it/` get a 404 because that path does not exist.

With `defaultContentLanguageInSubdir = true`, both languages get prefixes (`/it/` and `/en/`), and Hugo generates a redirect at `/` pointing to `/it/`. This is cleaner but the redirect is an HTML meta-refresh (not a server 301), which adds latency and is imperfect for SEO.

**Why it happens:**
Developers don't think through the URL structure implications before building content and templates. Changing this setting later moves every URL, breaking bookmarks, search indexes, and any external links.

**How to avoid:**
For a two-language site like IT/EN, use `defaultContentLanguageInSubdir = true` so both languages have symmetric URL structures (`/it/` and `/en/`). This is slightly more work upfront but prevents URL asymmetry problems. Decide this before creating any content -- changing it later is a rewrite of all internal links.

```toml
defaultContentLanguage = "it"
defaultContentLanguageInSubdir = true
```

**Warning signs:**
- Language switcher links are asymmetric (one has a prefix, the other doesn't)
- 404s on `/it/` paths when Italian is the default language
- Confusion about whether `/` and `/it/` should show the same content

**Phase to address:** Phase 1 (project scaffolding) -- this is a Day 0 architecture decision.

---

### Pitfall 4: DNS Order-of-Operations Causes Domain Takeover Risk or Certificate Failure

**What goes wrong:**
Configuring DNS records before adding the custom domain in GitHub Pages settings opens a window for domain takeover -- someone else could claim your domain on their GitHub Pages site. Conversely, adding the domain in GitHub but misconfiguring DNS causes HTTPS certificate provisioning to fail silently, and the "Enforce HTTPS" checkbox never becomes available.

**Why it happens:**
The GitHub Pages documentation explicitly warns about configuration order but developers often set up DNS first (it feels like the natural first step). The HTTPS certificate is only provisioned after both DNS and the GitHub Pages custom domain setting are correctly configured, and it can take up to 24 hours.

**How to avoid:**
Follow this exact order:
1. Add `toto-castaldi.com` as custom domain in GitHub repo Settings > Pages
2. Create DNS A records for apex: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
3. Create DNS CNAME record: `www.toto-castaldi.com` -> `toto-castaldi.github.io`
4. Wait for DNS propagation (up to 24h)
5. Enable "Enforce HTTPS" once the checkbox appears
6. Never use wildcard DNS records (`*.toto-castaldi.com`) -- GitHub docs explicitly warn these create "immediate risk of domain takeovers"

**Warning signs:**
- "Enforce HTTPS" checkbox is grayed out or missing
- Browser shows certificate warning on the custom domain
- `dig toto-castaldi.com` returns incorrect IP addresses
- GitHub Pages settings shows a DNS check warning

**Phase to address:** Phase 1 (infrastructure/deployment setup) -- must be done correctly the first time.

---

### Pitfall 5: Missing or Incorrect hreflang Tags Tank Multilingual SEO

**What goes wrong:**
Search engines cannot determine the relationship between the Italian and English versions of each page. Google may index only one language, show the wrong language in search results for a given locale, or flag content as duplicate rather than as translations.

**Why it happens:**
Hugo does not automatically inject hreflang tags. Developers must add them to the `<head>` partial. Common mistakes (per research, ~75% of hreflang implementations have errors): using relative URLs instead of absolute, forgetting the `x-default` tag, making non-reciprocal references (page A links to B's translation but B doesn't link back to A), or using wrong language codes (`it-IT` vs `it`).

**How to avoid:**
Create a partial template `layouts/partials/hreflang.html`:

```html
{{ if .IsTranslated }}
{{ range .AllTranslations }}
<link rel="alternate" hreflang="{{ .Lang }}" href="{{ .Permalink }}">
{{ end }}
<link rel="alternate" hreflang="x-default" href="{{ .Permalink }}">
{{ end }}
```

Include it in the `<head>` of `baseof.html`. Key rules:
- Use `.Permalink` (absolute URL), never `.RelPermalink`
- Include `x-default` pointing to the default language version
- Ensure every page includes hreflang for ALL its translations (reciprocal)
- Use simple language codes (`it`, `en`) matching Hugo's language keys

**Warning signs:**
- Google Search Console shows "hreflang" warnings
- Same page indexed in wrong language for a locale
- Missing "alternate" link tags when viewing page source

**Phase to address:** Phase 2 (multilingual content/templates) -- must be in the base template.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoding text in templates instead of using i18n strings | Faster initial development | Adding a third language requires editing every template | Never for a multilingual site, even with only 2 languages |
| Using `relativeURLs = true` | Avoids baseURL issues locally | Breaks multilingual URL resolution, produces broken relative URLs with language prefixes | Never with multilingual Hugo |
| Skipping `static/CNAME` and setting domain via GitHub UI | One less file to manage | Domain detaches on every deployment | Never |
| Inlining all CSS rather than using Hugo Pipes | Simpler build | No minification, no cache-busting fingerprinting | Acceptable only if CSS is under ~2KB total |
| Putting translations in filename suffixes (`.it.md`, `.en.md`) instead of content directories | Fewer directories | Harder to manage as content grows; file list becomes cluttered | Acceptable for a very small site (3-5 pages) like this project |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| GitHub Pages deployment | Using `peaceiris/actions-gh-pages` (deploys to gh-pages branch) instead of the official `actions/deploy-pages` workflow | Use the official Hugo starter workflow from `actions/starter-workflows/pages/hugo.yml` which deploys via GitHub's artifact-based Pages pipeline |
| GitHub Pages + Custom Domain | Setting custom domain in GitHub UI only (no `static/CNAME`) | Always place CNAME in `static/` directory of Hugo source |
| DNS (apex + www) | Creating only A records for apex, forgetting CNAME for www | Configure both: A records for apex domain AND CNAME record for www subdomain pointing to `username.github.io` |
| Hugo theme as Git submodule | Running `git clone` for the theme instead of `git submodule add` | Use `git submodule add` so the GitHub Actions checkout step with `submodules: true` fetches it correctly |
| Hugo theme as module | Not including Go installation in GitHub Actions workflow | Add Go setup step before Hugo build if using Hugo Modules |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Large unoptimized images in `static/` | Slow page load, poor Lighthouse score | Use Hugo's image processing (`resources.Get` + `.Resize`) or pre-optimize images before committing | Immediately visible with images > 200KB |
| Loading full Google Fonts stylesheet | Render-blocking CSS, FOUT/FOIT | Self-host only the font weights/subsets needed, use `font-display: swap` | Noticeable on slow connections or mobile |
| No HTML/CSS minification | Larger payload than necessary | Use `hugo --minify` in the build command | Marginal for a small landing page, but free to enable |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Wildcard DNS record (`*.toto-castaldi.com`) | Domain takeover: anyone can claim subdomains via GitHub Pages | Only create specific DNS records (A for apex, CNAME for www) |
| Not enforcing HTTPS | Mixed content, credential interception, SEO penalty | Enable "Enforce HTTPS" in GitHub Pages settings after certificate provisioning |
| Exposing draft/future content | Publishing content marked `draft: true` if build flags are wrong | Never use `hugo -D` in production workflow; review build command in GitHub Actions |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Language switcher links to homepage instead of translated page | User loses context, must re-navigate | Use `.Translations` to link to the equivalent translated page: `{{ range .Translations }}<a href="{{ .Permalink }}">{{ .Language.LanguageName }}</a>{{ end }}` |
| No visual indicator of current language | User unsure which language they're viewing | Highlight the active language in the switcher; set `lang` attribute on `<html>` tag |
| Root URL (`/`) shows a meta-refresh redirect page | Flash of redirect page content, poor first impression | When using `defaultContentLanguageInSubdir = true`, create a custom `layouts/index.redir` or use JavaScript-based instant redirect on the root index |
| 404 page always in default language | Italian user hitting a broken English link sees Italian 404 | GitHub Pages only serves root `/404.html`. Use a single 404 template with JS to detect URL path (`/en/` vs `/it/`) and display appropriate language content |
| No `lang` attribute on `<html>` tag | Screen readers announce content in wrong language; SEO impact | Use `<html lang="{{ .Site.Language.Lang }}">` in `baseof.html` |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Custom domain:** CNAME file exists in `static/CNAME`, not just in GitHub Settings UI -- verify it survives a fresh deployment
- [ ] **HTTPS:** "Enforce HTTPS" is checked AND the certificate covers both `toto-castaldi.com` and `www.toto-castaldi.com` -- visit both URLs in browser
- [ ] **hreflang tags:** Every page includes reciprocal hreflang tags for ALL languages plus `x-default` -- view page source of every template type
- [ ] **Canonical URLs:** Each page has a `<link rel="canonical">` pointing to itself (not cross-language) -- check both IT and EN pages
- [ ] **Language switcher:** Links to the translated equivalent of the current page, not just the other language's homepage -- test on every page type
- [ ] **baseURL trailing slash:** `baseURL` ends with `/` in `hugo.toml` -- verify all asset paths work on deployed site, not just localhost
- [ ] **i18n strings:** All UI text (button labels, navigation, footer) uses `{{ i18n "key" }}`, not hardcoded strings -- check by switching languages
- [ ] **404 page:** Custom 404.html exists at root level and handles both languages -- visit a non-existent URL under both `/it/` and `/en/`
- [ ] **Open Graph / meta tags:** `og:locale` and `og:locale:alternate` are set correctly per language -- use a social media debugger tool
- [ ] **Sitemap:** Multilingual sitemap includes all language versions with correct URLs -- fetch `/sitemap.xml` on deployed site

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| CNAME lost on deployment | LOW | Add `static/CNAME`, push, re-enable HTTPS in GitHub settings. Wait up to 24h for cert. |
| baseURL wrong (broken assets) | LOW | Fix `baseURL` in `hugo.toml`, push. Instant fix after deployment completes. |
| defaultContentLanguageInSubdir changed mid-project | MEDIUM | Update config, rebuild. All URLs change -- update any external links, resubmit sitemap to search engines, add redirects via aliases if old URLs were indexed. |
| hreflang tags missing/wrong | LOW | Add/fix the partial template, push. Google re-crawls within days, but full re-indexing takes weeks. |
| DNS misconfigured | MEDIUM | Fix DNS records. Propagation takes up to 24h. HTTPS cert may need to be re-provisioned (disable then re-enable custom domain in GitHub settings). |
| Domain takeover via wildcard DNS | HIGH | Remove wildcard record immediately. Contact GitHub support if another site claimed your subdomain. Audit all DNS records. |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| CNAME file wiped on deployment | Phase 1: Project scaffolding | After first GitHub Pages deployment, verify custom domain persists in Settings |
| baseURL mismatch | Phase 1: Project scaffolding | Visit deployed site, confirm CSS/JS loads, check browser console for 404s |
| defaultContentLanguageInSubdir decision | Phase 1: Project scaffolding | Verify `/it/` and `/en/` both resolve; verify language switcher URL symmetry |
| DNS order-of-operations | Phase 1: Infrastructure setup | `dig` the domain, visit both apex and www, verify HTTPS padlock |
| hreflang tags | Phase 2: Multilingual templates | View source of deployed pages, check for reciprocal hreflang + x-default |
| Language switcher links wrong page | Phase 2: Multilingual templates | Click language switcher on every page type, verify URL changes but content context preserved |
| Hardcoded strings instead of i18n | Phase 2: Multilingual content | Switch language, verify all UI text changes (not just content) |
| Missing canonical URLs | Phase 2: SEO setup | Check page source for `<link rel="canonical">` on every page |
| 404 page single-language | Phase 2: Multilingual templates | Visit non-existent URLs under both language prefixes |
| Asset optimization | Phase 3: Polish/optimization | Run Lighthouse, verify image sizes and font loading strategy |

## Sources

- [Hugo: Host on GitHub Pages (official docs)](https://gohugo.io/host-and-deploy/host-on-github-pages/) -- HIGH confidence
- [Hugo: Configure Languages (official docs)](https://gohugo.io/configuration/languages/) -- HIGH confidence
- [Hugo: Multilingual Content Management (official docs)](https://gohugo.io/content-management/multilingual/) -- HIGH confidence
- [Hugo: URL Management (official docs)](https://gohugo.io/content-management/urls/) -- HIGH confidence
- [GitHub Pages: Managing Custom Domains (official docs)](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) -- HIGH confidence
- [GitHub Pages: Securing with HTTPS (official docs)](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https) -- HIGH confidence
- [GitHub Pages: Custom 404 pages (official docs)](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site) -- HIGH confidence
- [GitHub Actions starter workflow for Hugo](https://github.com/actions/starter-workflows/blob/main/pages/hugo.yml) -- HIGH confidence
- [Hugo Discourse: CNAME reset on deployment](https://github.com/orgs/community/discussions/22366) -- MEDIUM confidence
- [Hugo Discourse: defaultContentLanguageInSubdir gotchas](https://discourse.gohugo.io/t/defaultcontentlanguageinsubdir/4658) -- MEDIUM confidence
- [Hugo Discourse: Custom 404 per language](https://discourse.gohugo.io/t/custom-404-per-language/20239) -- MEDIUM confidence
- [Hugo Discourse: relativeURLs breaks multilingual](https://discourse.gohugo.io/t/relativeurls-true-with-multilingual-websites-produces-broken-relative-urls/28732) -- MEDIUM confidence
- [Hugo Issue #5898: Redirects for defaultContentLanguage sections](https://github.com/gohugoio/hugo/issues/5898) -- HIGH confidence
- [Multilingual SEO with Hugo (Glukhov, 2025)](https://www.glukhov.org/post/2025/10/multi-language-website-seo-with-hugo/) -- MEDIUM confidence
- [hreflang implementation on Hugo sites (Wieckiewicz)](https://dariusz.wieckiewicz.org/en/setting-hreflang-and-x-default-on-multilingual-site-part-2/) -- MEDIUM confidence

---
*Pitfalls research for: Hugo multilingual landing page on GitHub Pages*
*Researched: 2026-02-17*
