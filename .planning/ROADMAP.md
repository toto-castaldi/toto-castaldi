# Roadmap: toto-castaldi.com

## Overview

A minimal multilingual landing page showcasing three software projects (Docora, Lumio, Helix) in Italian and English, built with Hugo and hosted on GitHub Pages. The roadmap moves from infrastructure foundation (Day 0 decisions that cannot change later) through the complete bilingual site (content, design, interactivity) to SEO verification and production polish.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation and Deploy Pipeline** - Hugo project, multilingual config, GitHub Actions, custom domain with HTTPS
- [ ] **Phase 2: Bilingual Site with Design** - Project cards, language switcher, responsive layout, typography, dark mode, favicon
- [ ] **Phase 3: SEO and Production Polish** - hreflang tags, Open Graph, canonical URLs, final verification

## Phase Details

### Phase 1: Foundation and Deploy Pipeline
**Goal**: A blank but deployable bilingual site is live at toto-castaldi.com with correct URL structure, HTTPS, and stable custom domain
**Depends on**: Nothing (first phase)
**Requirements**: INFR-01, INFR-02, INFR-03, INFR-04, INFR-05
**Success Criteria** (what must be TRUE):
  1. Visiting https://toto-castaldi.com loads a page served by GitHub Pages with a valid HTTPS certificate
  2. Visiting https://www.toto-castaldi.com redirects to or serves the same site
  3. The site has distinct `/it/` and `/en/` URL paths that each render a page in the correct language
  4. Pushing to main branch triggers GitHub Actions and deploys automatically without losing the custom domain
**Plans**: 2 plans

Plans:
- [ ] 01-01-PLAN.md -- Hugo project scaffold, bilingual config, templates, GitHub Actions workflow, CNAME, 404 page
- [ ] 01-02-PLAN.md -- GitHub Pages configuration, custom domain DNS, HTTPS verification

### Phase 2: Bilingual Site with Design
**Goal**: Visitors see three project cards (Docora, Lumio, Helix) with names, descriptions, and links in their chosen language, on a clean responsive page with language switching
**Depends on**: Phase 1
**Requirements**: CONT-01, CONT-02, CONT-03, CONT-04, CONT-05, CONT-06, UI-01, UI-02, UI-03, UI-04, UI-05, DSGN-01, DSGN-02, DSGN-03, DSGN-04
**Success Criteria** (what must be TRUE):
  1. Each project card shows the project name, a short description in the current language, and a clickable link to its subdomain
  2. Clicking the language switcher navigates between Italian and English versions of the same page (not just the homepage)
  3. The page renders correctly on mobile (cards stack vertically) and desktop with no horizontal scrolling
  4. The site respects OS dark mode preference, switching color scheme automatically with no JavaScript
  5. Page loads in under 1 second with zero JavaScript in the output
**Plans**: TBD

Plans:
- [ ] 02-01: TBD

### Phase 3: SEO and Production Polish
**Goal**: The site is discoverable by search engines with correct multilingual signals and renders rich previews when shared on social platforms
**Depends on**: Phase 2
**Requirements**: SEO-01, SEO-02, SEO-03
**Success Criteria** (what must be TRUE):
  1. Each language version includes hreflang tags pointing to its counterpart and an x-default, with all references using absolute URLs
  2. Sharing a page URL on a platform like LinkedIn or Slack shows a rich preview with correct title, description, and locale
  3. Each language version has a canonical URL pointing to itself, preventing duplicate content signals
**Plans**: TBD

Plans:
- [ ] 03-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation and Deploy Pipeline | 0/0 | Not started | - |
| 2. Bilingual Site with Design | 0/0 | Not started | - |
| 3. SEO and Production Polish | 0/0 | Not started | - |
