# Architecture Research

**Domain:** Hugo multilingual landing page (IT/EN) for software project showcase
**Researched:** 2026-02-17
**Confidence:** HIGH

## Standard Architecture

### System Overview

```
Build Time (Hugo)                              Runtime (GitHub Pages)
=================                              =====================

 config/                                        toto-castaldi.com/
 +-- _default/                                  +-- index.html        (EN home)
 |   +-- hugo.toml        ---+                  +-- it/
 |   +-- languages.toml       |                 |   +-- index.html    (IT home)
 |   +-- menus.en.toml        |                 +-- css/
 |   +-- menus.it.toml        |                 |   +-- style.min.*.css
 content/                      |  hugo --minify  +-- js/ (if any)
 +-- _index.en.md         ----+---------------> +-- images/
 +-- _index.it.md              |                 +-- favicon.ico
 i18n/                         |                 +-- robots.txt
 +-- en.toml               ---+
 +-- it.toml                   |
 layouts/                      |
 +-- _default/                 |
 |   +-- baseof.html       ---+
 +-- index.html                |
 +-- partials/                 |
 |   +-- head.html             |
 |   +-- header.html           |
 |   +-- project-card.html     |
 |   +-- language-switcher.html|
 |   +-- footer.html       ---+
 assets/                       |
 +-- css/                      |
 |   +-- main.css          ---+
 static/
 +-- images/
 +-- favicon.ico
```

This is a static site generator architecture. There is no server, no database, no API. Hugo compiles templates + content + data into a flat directory of HTML/CSS/JS files. GitHub Pages serves those files directly. The entire "backend" is the build step.

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| **config/** | Site metadata, language definitions, menu structures | TOML files split by concern (hugo.toml, languages.toml, menus.*.toml) |
| **content/** | Page content in Markdown with front matter | `_index.en.md` and `_index.it.md` for the home page (translation by filename) |
| **i18n/** | UI string translations (buttons, labels, headings) | `en.toml` and `it.toml` with key-value pairs |
| **layouts/** | HTML templates that define page structure | baseof.html (shell), index.html (home), partials (reusable components) |
| **assets/** | CSS/JS processed through Hugo Pipes | `main.css` processed with minification and fingerprinting |
| **static/** | Files copied verbatim to output (images, favicon) | Binary assets, robots.txt |
| **data/** | Structured data accessible in templates via `.Site.Data` | Project definitions (Docora, Lumio, Helix) as TOML/YAML |
| **.github/workflows/** | CI/CD pipeline for building and deploying | GitHub Actions workflow using Hugo extended edition |

## Recommended Project Structure

```
toto-castaldi/
+-- .github/
|   +-- workflows/
|       +-- hugo.yaml              # Build and deploy to GitHub Pages
+-- config/
|   +-- _default/
|       +-- hugo.toml              # Site-wide settings (baseURL, title)
|       +-- languages.toml         # Language definitions (en, it)
|       +-- menus.en.toml          # English navigation
|       +-- menus.it.toml          # Italian navigation
+-- content/
|   +-- _index.en.md              # English home page content
|   +-- _index.it.md              # Italian home page content
+-- data/
|   +-- projects.toml             # Project data (Docora, Lumio, Helix)
+-- i18n/
|   +-- en.toml                   # English UI strings
|   +-- it.toml                   # Italian UI strings
+-- layouts/
|   +-- _default/
|   |   +-- baseof.html           # Base HTML shell (head, body wrapper)
|   +-- index.html                # Home page template
|   +-- partials/
|       +-- head.html             # <head> tag (meta, CSS, fonts)
|       +-- header.html           # Site header with nav + language switcher
|       +-- project-card.html     # Single project display component
|       +-- language-switcher.html # EN/IT toggle
|       +-- footer.html           # Site footer
+-- assets/
|   +-- css/
|       +-- main.css              # Stylesheet (processed by Hugo Pipes)
+-- static/
|   +-- images/                   # Project logos, hero images
|   +-- favicon.ico
|   +-- CNAME                     # Custom domain for GitHub Pages
+-- .planning/                    # Project planning (not published)
```

### Structure Rationale

- **config/_default/:** Hugo supports split configuration. Separating `hugo.toml`, `languages.toml`, and menu files keeps each concern isolated. The `_default` subdirectory is Hugo's convention for the default configuration environment.

- **content/ with filename-based translation:** For a simple 2-language landing page, using `_index.en.md` and `_index.it.md` in the same directory is simpler than separate content directories. Files sharing the same basename are automatically linked as translations. This approach keeps related translations adjacent and visible.

- **data/projects.toml:** Project information (Docora, Lumio, Helix) is structured data, not prose content. Storing it in `data/` makes it accessible via `.Site.Data.projects` in templates, enabling a single template to loop over all projects. This separates data from presentation cleanly.

- **i18n/ for UI strings:** All user-facing text that is not page content (section headings, button labels, "View Project", "About") goes in i18n files. The `{{ T "key" }}` function retrieves the correct translation automatically based on the current language context.

- **layouts/partials/:** Each visual component gets its own partial template. This enables reuse and keeps the main template readable. Hugo partials are called with `{{ partial "name.html" . }}`.

- **assets/ vs static/:** CSS goes in `assets/` to leverage Hugo Pipes (minification, fingerprinting for cache busting). Images and other binary files go in `static/` since they do not need processing.

## Architectural Patterns

### Pattern 1: Translation by Filename (Recommended for This Project)

**What:** Content files use language suffixes: `_index.en.md`, `_index.it.md`. Hugo automatically links them as translations of the same page.

**When to use:** Small sites with few pages where you want translations side by side. Perfect for a 2-language landing page.

**Trade-offs:** Easy to spot missing translations (files are adjacent). Does not scale well to 10+ languages or hundreds of pages, but that is irrelevant for this project.

**Example:**
```markdown
<!-- content/_index.en.md -->
---
title: "Toto Castaldi"
description: "Software projects showcase"
---

Welcome to my projects.
```

```markdown
<!-- content/_index.it.md -->
---
title: "Toto Castaldi"
description: "Vetrina progetti software"
---

Benvenuto tra i miei progetti.
```

### Pattern 2: Structured Data for Project Listings

**What:** Define project metadata in `data/projects.toml` as an array. Templates iterate over `.Site.Data.projects` to render project cards. Translation of project descriptions uses i18n keys or per-language fields in the data file.

**When to use:** When multiple items share the same display structure but vary in content. Avoids duplicating template logic.

**Trade-offs:** Couples data structure to template expectations. For 3 projects this is the right level of abstraction; a CMS-like approach (each project as a content page) would be overengineering.

**Example:**
```toml
# data/projects.toml
[[projects]]
  name = "Docora"
  url = "https://github.com/toto-castaldi/docora"
  description_en = "Short description of Docora"
  description_it = "Breve descrizione di Docora"

[[projects]]
  name = "Lumio"
  url = "https://github.com/toto-castaldi/lumio"
  description_en = "Short description of Lumio"
  description_it = "Breve descrizione di Lumio"

[[projects]]
  name = "Helix"
  url = "https://github.com/toto-castaldi/helix"
  description_en = "Short description of Helix"
  description_it = "Breve descrizione di Helix"
```

```html
<!-- layouts/partials/project-card.html -->
{{ $lang := .Site.Language.Lang }}
{{ range .Site.Data.projects.projects }}
<article class="project-card">
  <h3>{{ .name }}</h3>
  <p>{{ index . (printf "description_%s" $lang) }}</p>
  <a href="{{ .url }}">{{ T "view_project" }}</a>
</article>
{{ end }}
```

### Pattern 3: Base Template with Blocks

**What:** `baseof.html` defines the HTML shell with named `{{ block }}` placeholders. Page templates fill those blocks with `{{ define }}`. This separates structural HTML from page-specific content.

**When to use:** Always. This is Hugo's standard template inheritance model.

**Trade-offs:** None meaningful. It is the canonical way to build Hugo templates.

**Example:**
```html
<!-- layouts/_default/baseof.html -->
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}">
<head>
  {{ partial "head.html" . }}
</head>
<body>
  {{ partial "header.html" . }}
  <main>
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
</body>
</html>
```

```html
<!-- layouts/index.html -->
{{ define "main" }}
<section class="hero">
  <h1>{{ .Title }}</h1>
  <p>{{ .Description }}</p>
</section>

<section class="projects">
  <h2>{{ T "projects_heading" }}</h2>
  {{ partial "project-card.html" . }}
</section>
{{ end }}
```

### Pattern 4: Hugo Pipes for Asset Processing

**What:** CSS files in `assets/` are processed through Hugo Pipes for minification and fingerprinting (cache busting), then referenced in templates.

**When to use:** Always for CSS and JS. Produces optimized output with unique filenames that work correctly with CDN caching.

**Trade-offs:** Slightly more complex template syntax than a plain `<link>` tag, but worth it for production builds.

**Example:**
```html
<!-- layouts/partials/head.html -->
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}" integrity="{{ $css.Data.Integrity }}">
```

## Data Flow

### Build Flow

```
[Content .md files] + [i18n .toml files] + [data/ .toml files]
         |                    |                      |
         v                    v                      v
    .Page object         T() function          .Site.Data
         |                    |                      |
         +--------------------+----------------------+
                              |
                              v
                    [layouts/ templates]
                              |
                              v
                 [Hugo Pipes: CSS/JS processing]
                              |
                              v
                    [public/ output directory]
                              |
                              v
              [GitHub Actions: upload artifact]
                              |
                              v
                [GitHub Pages: serve static files]
```

### Language Resolution Flow

```
[User visits toto-castaldi.com/]
    |
    +--> GitHub Pages serves /index.html (English, default language)
    |
[User visits toto-castaldi.com/it/]
    |
    +--> GitHub Pages serves /it/index.html (Italian)

At build time:
[Hugo processes _index.en.md]
    |
    +--> Renders with .Site.Language.Lang = "en"
    |    T() reads from i18n/en.toml
    |    Data descriptions use description_en field
    |    Output: public/index.html
    |
[Hugo processes _index.it.md]
    |
    +--> Renders with .Site.Language.Lang = "it"
         T() reads from i18n/it.toml
         Data descriptions use description_it field
         Output: public/it/index.html
```

### Key Data Flows

1. **Content to page:** Markdown front matter becomes `.Title`, `.Description`, `.Params.*`. Markdown body becomes `.Content`. Hugo renders content through the matching template based on lookup order.

2. **i18n to UI strings:** `{{ T "key" }}` looks up the key in the current language's i18n file. Falls back to `defaultContentLanguage` if missing. Returns empty string as last resort.

3. **Data to template:** `{{ .Site.Data.projects.projects }}` returns the array from `data/projects.toml`. Templates range over it and extract per-language fields.

4. **Assets to output:** `resources.Get` loads from `assets/`, piped through `Minify` and `Fingerprint`, published to `public/` with a content-hash filename.

5. **Translation linking:** Hugo automatically links `_index.en.md` and `_index.it.md` as translations. Templates access translations via `.Translations` for building the language switcher.

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 2 languages, 1 page (current) | Filename-based translation, single data file, no theme needed |
| 5 languages, 5 pages | Consider switching to content-directory translation (`contentDir` per language). Still no theme needed. |
| 10+ languages, 20+ pages | Use content-directory approach. Consider a Hugo theme for maintainability. Add `translationKey` front matter for pages that do not follow naming conventions. |

### Scaling Priorities

1. **First to outgrow:** The data file approach for projects. If projects grow beyond 10 with rich descriptions, switch to content pages (`content/projects/docora/index.en.md`). For 3 projects, the data file is perfect.

2. **Second to outgrow:** Filename-based translation. If adding more than 4-5 languages, the directory approach (`content/en/`, `content/it/`) becomes cleaner. For 2 languages, filename is ideal.

## Anti-Patterns

### Anti-Pattern 1: Using a Full Theme for a Minimal Site

**What people do:** Install a pre-built Hugo theme (e.g., PaperMod, Ananke) and try to customize it heavily for a landing page.

**Why it's wrong:** Themes carry hundreds of files, complex configuration, and opinions about structure. For a single landing page, the overhead of understanding and overriding theme templates exceeds the cost of writing 5-6 simple templates from scratch.

**Do this instead:** Write templates directly in `layouts/`. A landing page needs `baseof.html`, `index.html`, and 4-5 partials. Total: ~150 lines of HTML. Full control, no theme dependency.

### Anti-Pattern 2: Duplicating HTML Across Languages

**What people do:** Create separate `index-en.html` and `index-it.html` layout files with hardcoded text in each.

**Why it's wrong:** Any design change must be applied twice. Translations drift. Hugo's entire multilingual system exists to prevent this.

**Do this instead:** One set of templates. Use `{{ T "key" }}` for UI strings, `.Content` for page prose, and language-aware data access for project descriptions. The template is language-agnostic; Hugo handles language switching.

### Anti-Pattern 3: Putting CSS in static/ Instead of assets/

**What people do:** Place CSS in `static/css/style.css` and link it with a plain `<link>` tag.

**Why it's wrong:** No minification, no fingerprinting (cache busting), no Sass/PostCSS processing. Users may see stale CSS after updates due to browser caching.

**Do this instead:** Place CSS in `assets/css/main.css`. Use Hugo Pipes: `resources.Get | resources.Minify | resources.Fingerprint`. The output gets a content-hash filename that changes on every edit, preventing cache staleness.

### Anti-Pattern 4: Content-Directory Translation for a 2-Language Single-Page Site

**What people do:** Set up `content/en/_index.md` and `content/it/_index.md` with `contentDir` configuration for each language, even when the site has just one page.

**Why it's wrong:** Adds unnecessary configuration complexity (`contentDir` per language in `languages.toml`). For a single page, the filename approach (`_index.en.md`, `_index.it.md`) is simpler and achieves the same result with zero extra configuration.

**Do this instead:** Use filename-based translation. Add `.en.md` / `.it.md` suffixes. Hugo links them automatically.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| **GitHub Pages** | GitHub Actions workflow builds Hugo site and deploys `public/` as artifact | Set source to "GitHub Actions" in repo Settings > Pages |
| **Custom Domain (toto-castaldi.com)** | `CNAME` file in `static/`, DNS A/CNAME records pointing to GitHub | Place `CNAME` in `static/` so it copies to `public/` root on build |
| **Google Fonts (optional)** | `<link>` tag in `head.html` partial, or self-host in `static/fonts/` | Self-hosting avoids GDPR cookie consent issues in EU (Italy) |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| **config -> templates** | `.Site.Title`, `.Site.Params.*`, `.Site.Menus.*` | Config values available globally in all templates |
| **content -> templates** | `.Title`, `.Content`, `.Params.*`, `.Description` | Each content file maps to a Page object |
| **i18n -> templates** | `{{ T "key" }}` function | Automatic language context from current page |
| **data -> templates** | `.Site.Data.projects` | Merged data structure, available globally |
| **assets -> templates** | `resources.Get "path"` | Returns a Resource for pipeline processing |
| **templates -> output** | Hugo renderer | Templates produce HTML; Hugo writes to `public/` |

## Build Order (Dependencies Between Components)

The following order reflects true dependencies -- each step requires the previous ones to exist.

```
Phase 1: Foundation
  hugo.toml + languages.toml     (site exists, languages defined)
  baseof.html                     (HTML shell exists)
  head.html partial               (CSS link, meta tags)

Phase 2: Content Structure
  _index.en.md + _index.it.md    (pages exist)
  index.html template             (home page renders)
  i18n/en.toml + i18n/it.toml   (UI strings translate)

Phase 3: Visual Components
  assets/css/main.css             (styling)
  header.html partial             (navigation)
  footer.html partial             (footer)
  language-switcher.html partial  (EN/IT toggle)

Phase 4: Project Showcase
  data/projects.toml              (project data)
  project-card.html partial       (project display)

Phase 5: Deployment
  .github/workflows/hugo.yaml    (CI/CD pipeline)
  static/CNAME                   (custom domain)
```

This order works because each phase produces a functional (if incomplete) site. Phase 1 gives you a blank page that builds. Phase 2 gives you content. Phase 3 gives you design. Phase 4 gives you the actual project showcase. Phase 5 puts it on the internet.

## Sources

- [Hugo Multilingual Mode](https://gohugo.io/content-management/multilingual/) -- Official docs on translation by filename/directory, language configuration, translation linking (HIGH confidence)
- [Hugo Directory Structure](https://gohugo.io/getting-started/directory-structure/) -- Official docs on standard directories (HIGH confidence)
- [Hugo Template Types](https://gohugo.io/templates/types/) -- Official docs on base, home, single, list, partial templates (HIGH confidence)
- [Hugo Template Lookup Order](https://gohugo.io/templates/lookup-order/) -- How Hugo resolves which template to use (HIGH confidence)
- [Hugo lang.Translate](https://gohugo.io/functions/lang/translate/) -- i18n function, file format, pluralization (HIGH confidence)
- [Hugo Pipes Introduction](https://gohugo.io/hugo-pipes/introduction/) -- Asset processing pipeline (HIGH confidence)
- [Hugo Data Templates](https://gohugo.io/templates/data-templates/) -- data/ directory and .Site.Data access (HIGH confidence)
- [Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/) -- GitHub Actions workflow for Hugo deployment (HIGH confidence)
- [Hugo Multilingual Part 1 (Regis Philibert)](https://www.regisphilibert.com/blog/2018/08/hugo-multilingual-part-1-managing-content-translation/) -- Community comparison of translation approaches (MEDIUM confidence)
- [CloudCannon Translation Guides](https://cloudcannon.com/documentation/developer-guides/hugo-multilingual/translation-by-filename/) -- Practical examples of both approaches (MEDIUM confidence)

---
*Architecture research for: Hugo multilingual landing page (toto-castaldi.com)*
*Researched: 2026-02-17*
