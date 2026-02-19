# toto-castaldi.com

## What This Is

A minimal, multilingual (Italian/English) landing page that showcases Toto Castaldi's public software projects. Built with Hugo and hosted on GitHub Pages at toto-castaldi.com and www.toto-castaldi.com. Zero JavaScript, CSS-only dark mode, responsive bento grid design with WCAG AA accessibility.

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
- ✓ Complete visual redesign with bento grid layout — v1.1
- ✓ 3-column responsive project card grid (desktop → tablet → mobile) — v1.1
- ✓ Dark + light mode both legible with WCAG AA contrast — v1.1
- ✓ Light mode border/text contrast fixed to WCAG AA — v1.1
- ✓ Additional bento sections: GitHub profile link — v1.1

### Active

(None — planning next milestone)

### Out of Scope

- About/bio section — not needed, focus on projects only
- Contact form or social links — keep it minimal
- Blog or dynamic content — static landing page only
- Mobile app — web only
- SCSS migration — under 500 lines CSS, custom properties sufficient
- Container queries — single-page, media queries suffice
- JavaScript — zero-JS constraint maintained

## Context

Shipped v1.1 with 489 LOC (394 CSS, 55 templates, 40 i18n TOML).
Tech stack: Hugo Extended 0.155.3, Dart Sass, GitHub Actions, GitHub Pages.
HTTPS live with Let's Encrypt. System font stack for zero network requests.
CSS custom properties for design tokens (colors, spacing, typography, shadows, radius).
Responsive bento grid: 3 columns desktop, 2 tablet, 1 mobile.
WCAG AA contrast verified in both light and dark themes.

## Constraints

- **Stack**: Hugo static site generator — user preference
- **Hosting**: GitHub Pages — free, integrated with repo
- **Languages**: Italian and English — both required from day one
- **Design**: Bento grid, modern, responsive — dark + light mode both legible
- **Performance**: Zero JavaScript, sub-1s load
- **CSS**: Plain CSS with custom properties, no preprocessor needed under 500 LOC

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
| Tailwind Slate palette for dark tokens | Better contrast ratios than gray palette | ✓ Good |
| page-wrapper / container separation | Full-width bg transitions + content constraint | ✓ Good |
| Explicit repeat(N,1fr) over auto-fit | Deterministic 3-item layout, no surprises | ✓ Good |
| 0.75rem card radius via --radius-card token | Bento aesthetic, design-token driven | ✓ Good |
| Inline SVG for GitHub icon | fill=currentColor for theme, zero dependencies | ✓ Good |
| Reusable .bento-cell class | Separate from .project-card, extensible for future sections | ✓ Good |
| prefers-reduced-motion guard for hover | Accessible animations, CSS-only approach | ✓ Good |

---
*Last updated: 2026-02-19 after v1.1 milestone completion*
