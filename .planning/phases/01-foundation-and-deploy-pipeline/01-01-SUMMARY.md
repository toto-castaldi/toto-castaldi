---
phase: 01-foundation-and-deploy-pipeline
plan: 01
subsystem: infra
tags: [hugo, github-actions, github-pages, multilingual, i18n, hugo-pipes]

# Dependency graph
requires: []
provides:
  - "Complete Hugo project scaffold with bilingual /it/ and /en/ URL structure"
  - "GitHub Actions CI/CD pipeline for GitHub Pages deployment"
  - "CNAME file for custom domain persistence"
  - "Bilingual 404.html workaround for Hugo issue #5161"
  - "Hugo Pipes CSS pipeline with fingerprinting"
affects: [01-02, 02-content-and-design]

# Tech tracking
tech-stack:
  added: [hugo-extended-0.155.3, dart-sass-1.97.3, github-actions, github-pages]
  patterns: [split-config, filename-based-i18n, hugo-pipes-fingerprint, artifact-deploy]

key-files:
  created:
    - config/_default/hugo.toml
    - config/_default/languages.toml
    - content/_index.it.md
    - content/_index.en.md
    - layouts/_default/baseof.html
    - layouts/index.html
    - layouts/partials/head.html
    - assets/css/main.css
    - i18n/it.toml
    - i18n/en.toml
    - .github/workflows/hugo.yaml
    - static/CNAME
    - static/404.html
  modified: []

key-decisions:
  - "Italian (it) as defaultContentLanguage with defaultContentLanguageInSubdir=true for symmetric /it/ and /en/ paths"
  - "Hardcoded baseURL in hugo.toml, no --baseURL flag in CI (prevents project site URL injection)"
  - "Static bilingual 404.html workaround for Hugo issue #5161"
  - "Hugo Pipes with Minify + Fingerprint for CSS asset pipeline"

patterns-established:
  - "Split config: config/_default/hugo.toml for site settings, languages.toml for language definitions"
  - "Filename-based i18n: _index.it.md and _index.en.md for content translations"
  - "Hugo Pipes pipeline: resources.Get | resources.Minify | resources.Fingerprint for CSS"
  - "Artifact-based deploy: actions/upload-pages-artifact + actions/deploy-pages (no gh-pages branch)"

requirements-completed: [INFR-01, INFR-03, INFR-04, INFR-05]

# Metrics
duration: 2min
completed: 2026-02-18
---

# Phase 1 Plan 1: Hugo Project Scaffold Summary

**Bilingual Hugo site with /it/ and /en/ paths, Hugo Pipes CSS fingerprinting, and GitHub Actions artifact-based deploy to GitHub Pages**

## Performance

- **Duration:** 2 min
- **Started:** 2026-02-18T11:21:01Z
- **Completed:** 2026-02-18T11:23:08Z
- **Tasks:** 2
- **Files modified:** 13

## Accomplishments
- Complete Hugo project scaffold with split config, bilingual content, and template hierarchy
- Symmetric /it/ and /en/ URL structure with root meta-refresh redirect to /it/
- GitHub Actions workflow with pinned Hugo v0.155.3 and Dart Sass v1.97.3, no --baseURL pitfall
- Hugo Pipes CSS pipeline validated with fingerprinted output
- CNAME and bilingual 404.html for custom domain and error page support

## Task Commits

Each task was committed atomically:

1. **Task 1: Create Hugo project scaffold with multilingual configuration and templates** - `359f210` (feat)
2. **Task 2: Create GitHub Actions workflow, CNAME, and bilingual 404 page** - `65880cf` (feat)

## Files Created/Modified
- `config/_default/hugo.toml` - Site-wide Hugo config: baseURL, defaultContentLanguage=it, defaultContentLanguageInSubdir=true
- `config/_default/languages.toml` - Italian and English language definitions with weights
- `content/_index.it.md` - Italian placeholder homepage
- `content/_index.en.md` - English placeholder homepage
- `layouts/_default/baseof.html` - Base HTML template with lang attribute and head partial
- `layouts/index.html` - Home page template rendering title and content
- `layouts/partials/head.html` - Head partial with meta tags and Hugo Pipes CSS
- `assets/css/main.css` - Minimal stylesheet with system font stack, centered layout
- `i18n/it.toml` - Italian i18n strings (siteTitle)
- `i18n/en.toml` - English i18n strings (siteTitle)
- `.github/workflows/hugo.yaml` - CI/CD pipeline: build with Hugo, deploy to GitHub Pages
- `static/CNAME` - Custom domain persistence (toto-castaldi.com)
- `static/404.html` - Bilingual 404 page (Hugo issue #5161 workaround)

## Decisions Made
- Italian as default language: primary audience is Italian, domain owner is Italian
- defaultContentLanguageInSubdir=true: symmetric /it/ and /en/ paths, no ghost 404 on /it/
- Hardcoded baseURL: avoids configure-pages injecting repo name for project sites with custom domain
- Static 404.html: workaround for Hugo not generating root 404.html when both languages use subdirectories
- System font stack: validates Hugo Pipes pipeline without external font dependencies

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. DNS configuration (A records, CNAME for www) and GitHub Pages source setting are covered in plan 01-02.

## Next Phase Readiness
- Hugo project is fully functional and builds successfully
- GitHub Actions workflow is ready for deployment on push to main
- Bilingual content structure validated with placeholder pages
- Phase 2 can add real content, styling, and project cards without structural changes

## Self-Check: PASSED

All 13 created files verified present. Both task commits (359f210, 65880cf) verified in git log.

---
*Phase: 01-foundation-and-deploy-pipeline*
*Completed: 2026-02-18*
