# Phase 3: SEO and Production Polish - Research

**Researched:** 2026-02-18
**Domain:** Hugo multilingual SEO (hreflang, Open Graph, canonical URLs)
**Confidence:** HIGH

## Summary

Phase 3 adds three SEO capabilities to the existing Hugo multilingual site: hreflang tags for search engine language targeting, Open Graph meta tags for social sharing previews, and self-referencing canonical URLs to prevent duplicate content signals. All three are well-supported by Hugo's built-in features and embedded templates.

The current site (`head.html` partial) has no hreflang `<link>` tags in `<head>`, no Open Graph meta tags, and no canonical URL tags. The language switcher nav already uses `hreflang` attributes on its `<a>` tags, but those are navigation elements and not the `<link rel="alternate">` tags that search engines require.

**Primary recommendation:** Use Hugo's embedded `opengraph.html` partial for Open Graph tags, write a small custom partial for hreflang + canonical (10-15 lines), and update `languageCode` values in `languages.toml` to include territory codes (`it-IT`, `en-US`) for valid `og:locale` output.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SEO-01 | hreflang tags with reciprocal linking and x-default | Custom partial using `.AllTranslations` (includes self-reference, satisfying reciprocal requirement). x-default points to root URL which redirects to Italian. Uses `.Permalink` for absolute URLs. |
| SEO-02 | Open Graph meta tags (og:title, og:description, og:url, og:locale) | Hugo's embedded `{{ partial "opengraph.html" . }}` generates all required tags. Requires `languageCode` update to `it-IT`/`en-US` for valid og:locale format. |
| SEO-03 | Canonical URLs per language version | Self-referencing `<link rel="canonical" href="{{ .Permalink }}">` in head partial. `.Permalink` produces absolute URLs with baseURL. |
</phase_requirements>

## Standard Stack

### Core

| Component | Version | Purpose | Why Standard |
|-----------|---------|---------|--------------|
| Hugo embedded `opengraph.html` | 0.155.3 (bundled) | Generates og:title, og:description, og:url, og:locale, og:type and more | Official Hugo partial, maintained by core team, handles all edge cases |
| Hugo `.AllTranslations` | 0.155.3 (bundled) | Iterates all language versions of a page including self | Returns self + translations, exactly what hreflang reciprocal linking needs |
| Hugo `.Permalink` | 0.155.3 (bundled) | Produces absolute URLs for canonical and hreflang tags | Always produces full `https://` URLs based on `baseURL` config |

### Supporting

No additional libraries or dependencies needed. Everything is built into Hugo.

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Hugo embedded `opengraph.html` | Custom OG partial | More control but must maintain manually; embedded template handles image fallbacks, article metadata, series, audio/video |
| `.AllTranslations` for hreflang | `.Translations` + self-link | `.AllTranslations` already includes self-reference; using `.Translations` requires adding the current page separately |
| Self-referencing canonical per language | Single canonical pointing to default language | Self-referencing is correct for manually translated content; single-language canonical is only for machine translations |

**Installation:** None required. All features are built into Hugo 0.155.3.

## Architecture Patterns

### Recommended File Changes

```
layouts/partials/head.html       # Add canonical, hreflang, and opengraph calls
layouts/partials/seo.html        # NEW: hreflang + canonical partial (clean separation)
config/_default/languages.toml   # Update languageCode to include territory
```

### Pattern 1: SEO Partial Called from head.html

**What:** A dedicated `seo.html` partial containing hreflang links, canonical URL, and Open Graph include, called from the existing `head.html` partial.
**When to use:** When adding multiple `<head>` SEO elements to keep `head.html` clean.
**Example:**

```go-html-template
{{/* layouts/partials/seo.html */}}

{{/* Canonical URL - self-referencing per language version */}}
<link rel="canonical" href="{{ .Permalink }}">

{{/* hreflang tags - reciprocal linking for all language versions */}}
{{ range .AllTranslations }}
<link rel="alternate" hreflang="{{ .Language.LanguageCode }}" href="{{ .Permalink }}">
{{ end }}

{{/* x-default for unmatched language preferences */}}
<link rel="alternate" hreflang="x-default" href="{{ site.BaseURL }}">

{{/* Open Graph meta tags */}}
{{ partial "opengraph.html" . }}
```

Then in `head.html`, add:
```go-html-template
{{ partial "seo.html" . }}
```

Source: Google's hreflang documentation (https://developers.google.com/search/docs/specialty/international/localized-versions), Hugo docs (https://gohugo.io/templates/embedded/)

### Pattern 2: languageCode with Territory Codes

**What:** Update `languageCode` in `languages.toml` from bare language codes to language-territory format.
**When to use:** Required for valid og:locale output.
**Example:**

```toml
[it]
  languageCode = 'it-IT'
  languageName = 'Italiano'
  title = 'Antonio Toto Castaldi'
  weight = 1

[en]
  languageCode = 'en-US'
  languageName = 'English'
  title = 'Antonio Toto Castaldi'
  weight = 2
```

Hugo's embedded `opengraph.html` takes `site.Language.LanguageCode`, replaces hyphens with underscores, producing `it_IT` and `en_US` -- the correct og:locale format.

Source: Hugo embedded opengraph.html source (https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_partials/opengraph.html)

### Anti-Patterns to Avoid

- **Using `.RelPermalink` for hreflang or canonical:** Google requires absolute URLs (including protocol and domain). Always use `.Permalink` which includes the full `https://toto-castaldi.com/...` path.
- **Using `.Translations` instead of `.AllTranslations` for hreflang:** `.Translations` excludes the current page. Google requires each page to list itself in its own hreflang set (reciprocal + self-referencing).
- **Pointing x-default to a specific language version:** For a two-language site, x-default should point to a language-neutral page. The root URL (`https://toto-castaldi.com/`) already redirects to `/it/` (via Hugo's `defaultContentLanguageInSubdir`), which serves as a reasonable x-default entry point.
- **Setting og:locale without territory:** The Open Graph protocol specification requires `language_TERRITORY` format. `it` alone is not valid; must be `it_IT`.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Open Graph meta tags | Custom og:title/og:description/og:url template | `{{ partial "opengraph.html" . }}` | Embedded template handles 15+ OG properties, image fallbacks (feature/cover/thumbnail), article metadata, series, audio/video |
| Absolute URL generation | String concatenation with baseURL | `.Permalink` | Hugo handles trailing slashes, language prefixes, baseURL correctly |
| Locale format for og:locale | Manual `it_IT` / `en_US` strings | `languageCode` config with territory | Hugo's embedded template auto-converts hyphens to underscores |

**Key insight:** Hugo's embedded Open Graph template is comprehensive (handles og:url, og:locale, og:title, og:description, og:type, og:site_name, article metadata, images, audio, video, Facebook integration). Writing a custom version risks missing properties that social platforms check.

## Common Pitfalls

### Pitfall 1: Missing Reciprocal hreflang Links

**What goes wrong:** The Italian page links to English via hreflang, but the English page does not link back to Italian (or vice versa). Search engines ignore incomplete hreflang annotations.
**Why it happens:** Using `.Translations` (which excludes self) and forgetting to add a self-referencing link, or only adding hreflang on one language version.
**How to avoid:** Use `.AllTranslations` which includes the current page. Both pages will get identical hreflang sets because each page iterates all translations including itself.
**Warning signs:** Google Search Console reports "no return tag" hreflang errors.

### Pitfall 2: og:locale Without Territory Code

**What goes wrong:** Hugo's embedded OG template outputs `og:locale` content="it" instead of `it_IT` because `languageCode` is set to `it` without territory.
**Why it happens:** The initial `languages.toml` used bare language codes (`it`, `en`) which are valid for HTML `lang` attributes but not for the Open Graph `og:locale` property.
**How to avoid:** Set `languageCode = 'it-IT'` and `languageCode = 'en-US'` in `languages.toml`. Hugo's OG template replaces hyphens with underscores automatically.
**Warning signs:** Social sharing debuggers (LinkedIn Post Inspector, Facebook Sharing Debugger) show missing or default locale.

### Pitfall 3: Relative URLs in hreflang or Canonical Tags

**What goes wrong:** Using `.RelPermalink` produces `/it/` instead of `https://toto-castaldi.com/it/`. Search engines require absolute URLs for hreflang and canonical.
**Why it happens:** Hugo provides both `.RelPermalink` and `.Permalink`; easy to use the wrong one.
**How to avoid:** Always use `.Permalink` for any SEO `<link>` tag in `<head>`.
**Warning signs:** Validation tools flag relative URLs as errors.

### Pitfall 4: x-default Pointing to a Language-Specific Page

**What goes wrong:** Setting x-default to `/it/` or `/en/` means users whose language is not covered get sent to a specific language version rather than a neutral entry point.
**Why it happens:** Confusion about what x-default means -- it is for users who do NOT match any hreflang language.
**How to avoid:** Point x-default to `site.BaseURL` (the root `https://toto-castaldi.com/`) which currently redirects to `/it/` via Hugo's default language redirect. This is an acceptable pattern.
**Warning signs:** None obvious, but suboptimal for SEO purists who prefer a language-selector page.

### Pitfall 5: languageCode Change Affecting HTML lang Attribute

**What goes wrong:** Changing `languageCode` from `it` to `it-IT` might change the HTML `lang` attribute from `lang="it"` to `lang="it-IT"`.
**Why it happens:** Hugo uses `site.Language.Lang` (the config key, e.g., `it`) for some contexts and `site.Language.LanguageCode` for others.
**How to avoid:** The current `baseof.html` uses `.Site.Language.Lang` which returns the config key (`it`/`en`), NOT the `languageCode`. So the HTML `lang` attribute will remain `it` and `en`. Verify after the change.
**Warning signs:** Inspect HTML output after building -- `<html lang=it>` should remain, not become `<html lang=it-IT>`.

## Code Examples

Verified patterns from official sources:

### Complete SEO Partial (hreflang + canonical + OG)

```go-html-template
{{/* layouts/partials/seo.html */}}

{{/* SEO-03: Canonical URL - self-referencing, absolute */}}
<link rel="canonical" href="{{ .Permalink }}">

{{/* SEO-01: hreflang reciprocal links - .AllTranslations includes self */}}
{{ range .AllTranslations }}
<link rel="alternate" hreflang="{{ .Language.LanguageCode }}" href="{{ .Permalink }}">
{{ end }}
<link rel="alternate" hreflang="x-default" href="{{ site.BaseURL }}">

{{/* SEO-02: Open Graph meta tags via Hugo's embedded template */}}
{{ partial "opengraph.html" . }}
```

Source: Hugo `.AllTranslations` docs (https://gohugo.io/methods/page/alltranslations/), Hugo embedded templates docs (https://gohugo.io/templates/embedded/), Google hreflang guide (https://developers.google.com/search/docs/specialty/international/localized-versions)

### Updated head.html

```go-html-template
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{ .Title }}</title>
{{ with .Description }}
<meta name="description" content="{{ . }}">
{{ end }}
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
{{ partial "seo.html" . }}
```

### Updated languages.toml

```toml
[it]
  languageCode = 'it-IT'
  languageName = 'Italiano'
  title = 'Antonio Toto Castaldi'
  weight = 1

[en]
  languageCode = 'en-US'
  languageName = 'English'
  title = 'Antonio Toto Castaldi'
  weight = 2
```

### Expected Output: Italian Page `<head>`

After implementation, the Italian page (`/it/`) should contain:

```html
<link rel="canonical" href="https://toto-castaldi.com/it/">
<link rel="alternate" hreflang="it-IT" href="https://toto-castaldi.com/it/">
<link rel="alternate" hreflang="en-US" href="https://toto-castaldi.com/en/">
<link rel="alternate" hreflang="x-default" href="https://toto-castaldi.com/">
<meta property="og:url" content="https://toto-castaldi.com/it/">
<meta property="og:site_name" content="Antonio Toto Castaldi">
<meta property="og:title" content="Antonio Toto Castaldi">
<meta property="og:description" content="Progetti software di Toto Castaldi: Docora, Lumio e Helix">
<meta property="og:locale" content="it_IT">
<meta property="og:type" content="website">
```

### Expected Output: English Page `<head>`

```html
<link rel="canonical" href="https://toto-castaldi.com/en/">
<link rel="alternate" hreflang="it-IT" href="https://toto-castaldi.com/it/">
<link rel="alternate" hreflang="en-US" href="https://toto-castaldi.com/en/">
<link rel="alternate" hreflang="x-default" href="https://toto-castaldi.com/">
<meta property="og:url" content="https://toto-castaldi.com/en/">
<meta property="og:site_name" content="Antonio Toto Castaldi">
<meta property="og:title" content="Antonio Toto Castaldi">
<meta property="og:description" content="Software projects by Toto Castaldi: Docora, Lumio, and Helix">
<meta property="og:locale" content="en_US">
<meta property="og:type" content="website">
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `{{ template "_internal/opengraph.html" . }}` | `{{ partial "opengraph.html" . }}` | Hugo recent versions | Old `_internal` syntax removed; use `partial` now |
| Manual og:locale strings | `languageCode` config with auto-conversion | Hugo embedded template | Hugo replaces hyphens with underscores in LanguageCode |
| Schema.html microdata | JSON-LD structured data | Ongoing (out of scope for v1) | Hugo's embedded `schema.html` has known issues with missing itemscope/itemtype; JSON-LD is preferred but is a v2 requirement (PLSH-03) |

**Deprecated/outdated:**
- `{{ template "_internal/opengraph.html" . }}`: Replaced by `{{ partial "opengraph.html" . }}`
- `canonifyURLs = true` in Hugo config: This is a legacy post-render search-and-replace feature. Not needed; `.Permalink` already produces absolute URLs.

## Open Questions

1. **x-default target for a two-language site without a language-selector page**
   - What we know: Google recommends x-default point to a language-neutral page. The root URL (`https://toto-castaldi.com/`) redirects to `/it/` (the default language).
   - What's unclear: Whether pointing x-default to a page that redirects to a specific language is considered best practice by Google.
   - Recommendation: Use `site.BaseURL` as x-default. It is the closest to a neutral entry point in this architecture. Creating a dedicated language-selector page would be overkill for a two-language site and add unnecessary complexity. This is an acceptable and common pattern.

2. **og:locale territory code choice**
   - What we know: og:locale requires `language_TERRITORY` format. Italian maps naturally to `it_IT`. English has multiple options (`en_US`, `en_GB`, `en_AU`).
   - What's unclear: Whether `en_US` is the right choice given the site owner is Italian.
   - Recommendation: Use `en_US` as it is the Open Graph default and the most broadly recognized English locale. The site content is not UK/AU-specific.

## Sources

### Primary (HIGH confidence)
- Hugo `.AllTranslations` docs: https://gohugo.io/methods/page/alltranslations/ - confirms includes self-reference
- Hugo embedded templates docs: https://gohugo.io/templates/embedded/ - opengraph.html usage and configuration
- Hugo embedded `opengraph.html` source: https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_partials/opengraph.html - og:locale uses LanguageCode with hyphen-to-underscore replacement
- Google hreflang documentation: https://developers.google.com/search/docs/specialty/international/localized-versions - reciprocal linking, absolute URLs, x-default
- Open Graph Protocol specification: https://ogp.me/ - og:locale must be `language_TERRITORY` format
- Google x-default blog post: https://developers.google.com/search/blog/2013/04/x-default-hreflang-for-international-pages - x-default usage

### Secondary (MEDIUM confidence)
- Hugo multilingual SEO blog (Glukhov, 2025): https://www.glukhov.org/post/2025/10/multi-language-website-seo-with-hugo/ - canonical + hreflang patterns
- Hugo hreflang blog (Wieckiewicz): https://dariusz.wieckiewicz.org/en/setting-hreflang-and-x-default-on-multilingual-site/ - x-default strategy
- Hugo canonical URL blog: https://techtitbits.com/posts/hugo-canonical-url/ - `.Permalink` for canonical

### Tertiary (LOW confidence)
- None. All findings verified with primary or secondary sources.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Hugo's built-in features cover all requirements; verified via Context7 + official docs + source code
- Architecture: HIGH - Pattern is well-documented across official Hugo docs, Google guidelines, and community best practices
- Pitfalls: HIGH - All pitfalls verified via official specifications (Google hreflang docs, OGP spec) and confirmed against current codebase state

**Research date:** 2026-02-18
**Valid until:** 2026-03-18 (stable domain, Hugo 0.155.3 is current)
