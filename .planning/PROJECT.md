# toto-castaldi.com

## What This Is

A minimal, multilingual (Italian/English) landing page that showcases Toto Castaldi's public software projects. Built with Hugo and hosted on GitHub Pages at toto-castaldi.com and www.toto-castaldi.com. Zero JavaScript, CSS-only dark mode, responsive design with WCAG AA accessibility.

## Core Value

Visitors can immediately see what projects exist and navigate to each one — in their preferred language.

## Requirements

### Validated

- ✓ Multilingual landing page (IT/EN) generated with Hugo — v1.0
- ✓ Each project displayed with name, short description, and link — v1.0
- ✓ Three projects: Docora, Lumio, Helix with their subdomain URLs — v1.0
- ✓ Minimal clean design with strong typography — v1.0
- ✓ Hosted on GitHub Pages with custom domain (toto-castaldi.com + www) — v1.0
- ✓ Language switcher between Italian and English — v1.0

### Active

- [ ] Complete visual redesign with bento grid layout
- [ ] 3-column responsive project card grid (desktop → tablet → mobile)
- [ ] Dark + light mode both legible, with working toggle
- [ ] Fix light mode contrast (WCAG AA compliance)
- [ ] Additional bento sections where design warrants (tech stack, GitHub, etc.)

## Current Milestone: v1.1 Restyling Bento Grid

**Goal:** Redesign the landing page with a modern bento grid layout, proper responsive behavior, and legible dark/light themes.

**Target features:**
- Bento grid layout with 3-column project cards on desktop
- Responsive design that adapts from mobile to desktop
- Dark + light mode both meeting WCAG AA contrast
- Optional extra bento sections (tech stack, links, etc.)

### Out of Scope

- About/bio section — not needed, focus on projects only
- Contact form or social links — keep it minimal
- Blog or dynamic content — static landing page only
- Mobile app — web only

## Context

Shipped v1.0 with 501 LOC (Hugo templates, CSS, TOML config).
Tech stack: Hugo Extended 0.155.3, Dart Sass, GitHub Actions, GitHub Pages.
HTTPS live with Let's Encrypt. System font stack for zero network requests.

## Constraints

- **Stack**: Hugo static site generator — user preference
- **Hosting**: GitHub Pages — free, integrated with repo
- **Languages**: Italian and English — both required from day one
- **Design**: Bento grid, modern, responsive — dark + light mode both legible
- **Performance**: Zero JavaScript, sub-1s load

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Hugo over alternatives | User preference, excellent i18n support, fast builds | ✓ Good |
| GitHub Pages hosting | Free, simple deploy from repo, custom domain support | ✓ Good |
| No about/contact sections | User wants focus on projects only | ✓ Good |
| Italian as defaultContentLanguage with subdir | Symmetric /it/ and /en/ paths | ✓ Good |
| System font stack | Zero network requests, sub-1s load | ✓ Good |
| CSS-only dark mode toggle (checkbox hack) | No JS, broader browser support than :has() | ✓ Good |
| Static bilingual 404.html | Workaround for Hugo issue #5161 | ✓ Good |
| .AllTranslations for hreflang | Includes self-reference as required by Google | ✓ Good |

---
*Last updated: 2026-02-19 after v1.1 milestone start*
