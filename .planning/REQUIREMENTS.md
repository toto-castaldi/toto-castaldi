# Requirements: toto-castaldi.com

**Defined:** 2026-02-17
**Core Value:** Visitors can immediately see what projects exist and navigate to each one — in their preferred language.

## v1 Requirements

### Infrastructure

- [ ] **INFR-01**: Site generated with Hugo and deployed to GitHub Pages via GitHub Actions
- [ ] **INFR-02**: Custom domain toto-castaldi.com and www.toto-castaldi.com with HTTPS
- [ ] **INFR-03**: CNAME file in `static/` for custom domain persistence across deploys
- [ ] **INFR-04**: Hugo multilingual configuration with Italian and English (filename-based i18n)
- [ ] **INFR-05**: `defaultContentLanguageInSubdir` set correctly as Day 0 decision

### Content

- [ ] **CONT-01**: Home page with project cards showing name, short description, and link for each project
- [ ] **CONT-02**: Three projects displayed: Docora (docora.toto-castaldi.com), Lumio (lumio.toto-castaldi.com), Helix (helix.toto-castaldi.com)
- [ ] **CONT-03**: Project data stored in `data/projects.toml` with per-language fields
- [ ] **CONT-04**: Proper `<title>` and meta description per language
- [ ] **CONT-05**: i18n string files for UI labels in Italian and English
- [ ] **CONT-06**: Favicon (SVG with dark mode support)

### UI

- [ ] **UI-01**: Text-based language switcher (IT/EN) in header
- [ ] **UI-02**: Responsive layout that stacks project cards vertically on mobile
- [ ] **UI-03**: Semantic HTML with proper heading hierarchy and `lang` attribute
- [ ] **UI-04**: Zero JavaScript — pure HTML + CSS
- [ ] **UI-05**: Page load under 1 second

### Design

- [ ] **DSGN-01**: Minimal clean design with whitespace-heavy layout and strong typography
- [ ] **DSGN-02**: Dark mode via CSS `prefers-color-scheme` (no JS toggle)
- [ ] **DSGN-03**: Modular typography scale with CSS custom properties
- [ ] **DSGN-04**: WCAG AA contrast ratios (4.5:1 body text, 3:1 large text)

### SEO

- [ ] **SEO-01**: hreflang tags with reciprocal linking and x-default
- [ ] **SEO-02**: Open Graph meta tags (og:title, og:description, og:url, og:locale)
- [ ] **SEO-03**: Canonical URLs per language version

## v2 Requirements

### Polish

- **PLSH-01**: Print-friendly stylesheet
- **PLSH-02**: Open Graph image for social sharing previews
- **PLSH-03**: Structured data / JSON-LD markup

### Growth

- **GRWT-01**: Additional projects beyond the initial three
- **GRWT-02**: Privacy-respecting analytics (e.g., GoatCounter)

## Out of Scope

| Feature | Reason |
|---------|--------|
| About/bio section | Focus on projects only — projects communicate capability |
| Contact form | Requires form backend, JS, spam management — overkill for project directory |
| Blog / dynamic content | Changes site from landing page to content site, requires ongoing maintenance |
| JS animations / scroll effects | Contradicts minimal design, adds bundle size and accessibility issues |
| Dark mode toggle button | Requires JS — OS preference via CSS is sufficient |
| Social media links | Adds visual clutter to minimal design |
| Analytics / tracking | Adds privacy concerns and cookie banner requirements (GDPR) |
| Cookie consent banner | No cookies = no banner needed |
| Auto-detect language from browser | Creates confusion, unpredictable bookmarking, caching issues |
| Project detail pages | Each project has its own subdomain — no need to duplicate |
| OAuth / accounts | Static site, no user accounts |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| INFR-01 | Phase 1 | Pending |
| INFR-02 | Phase 1 | Pending |
| INFR-03 | Phase 1 | Pending |
| INFR-04 | Phase 1 | Pending |
| INFR-05 | Phase 1 | Pending |
| CONT-01 | Phase 2 | Pending |
| CONT-02 | Phase 2 | Pending |
| CONT-03 | Phase 2 | Pending |
| CONT-04 | Phase 2 | Pending |
| CONT-05 | Phase 2 | Pending |
| CONT-06 | Phase 2 | Pending |
| UI-01 | Phase 2 | Pending |
| UI-02 | Phase 2 | Pending |
| UI-03 | Phase 2 | Pending |
| UI-04 | Phase 2 | Pending |
| UI-05 | Phase 2 | Pending |
| DSGN-01 | Phase 2 | Pending |
| DSGN-02 | Phase 2 | Pending |
| DSGN-03 | Phase 2 | Pending |
| DSGN-04 | Phase 2 | Pending |
| SEO-01 | Phase 3 | Pending |
| SEO-02 | Phase 3 | Pending |
| SEO-03 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 23 total
- Mapped to phases: 23
- Unmapped: 0

---
*Requirements defined: 2026-02-17*
*Last updated: 2026-02-17 after roadmap creation*
