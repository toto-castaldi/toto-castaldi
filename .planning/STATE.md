# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-17)

**Core value:** Visitors can immediately see what projects exist and navigate to each one -- in their preferred language.
**Current focus:** All phases complete. Site is production-ready with SEO polish.

## Current Position

Phase: 3 of 3 (SEO and Production Polish)
Plan: 1 of 1 in current phase
Status: Complete
Last activity: 2026-02-18 -- Completed 03-01-PLAN.md (hreflang, canonical, Open Graph)

Progress: [##########] 100%

## Performance Metrics

**Velocity:**
- Total plans completed: 6
- Average duration: 4min
- Total execution time: 0.43 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1 | 2 | 19min | 10min |
| 2 | 3 | 5min | 2min |
| 3 | 1 | 2min | 2min |

**Recent Trend:**
- Last 5 plans: -
- Trend: -

*Updated after each plan completion*
| Phase 01 P01 | 2min | 2 tasks | 13 files |
| Phase 01 P02 | 17min | 3 tasks | 1 files |
| Phase 02 P01 | 2min | 2 tasks | 6 files |
| Phase 02 P02 | 1min | 2 tasks | 5 files |
| Phase 02 P03 | 2min | 2 tasks | 4 files |
| Phase 03 P01 | 2min | 1 tasks | 3 files |

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- [Roadmap]: 3-phase structure -- infrastructure first, then full bilingual site, then SEO polish
- [Roadmap]: defaultContentLanguageInSubdir must be decided in Phase 1 before any content exists
- [Phase 01]: Italian as defaultContentLanguage with defaultContentLanguageInSubdir=true for symmetric /it/ and /en/ paths
- [Phase 01]: Hardcoded baseURL in hugo.toml, no --baseURL flag in CI workflow
- [Phase 01]: Static bilingual 404.html workaround for Hugo issue #5161
- [Phase 01]: HTTPS enforcement deferred to async Let's Encrypt provisioning -- site functional over HTTP immediately
- [Phase 01]: DNS A records (4x GitHub Pages IPs) + CNAME for www -> toto-castaldi.github.io
- [Phase 02]: System font stack for zero network requests and sub-1s load
- [Phase 02]: Single-column centered layout (680px max-width) with vertically stacked cards
- [Phase 02]: Bordered cards with rounded corners, whole-card clickable via CSS ::after overlay
- [Phase 02]: WCAG AA verified color pairs for both light and dark modes
- [Phase 02]: Language switcher shows only alternate language (not current) since with two languages showing both is redundant
- [Phase 02]: TC monogram SVG favicon with embedded prefers-color-scheme for zero-JS dark mode
- [Phase 02]: Decorative logo images use alt='' since project name appears in adjacent h2 heading
- [Phase 02]: CSS-only checkbox :checked + sibling selector for dark mode toggle, no :has() for broader browser support
- [Phase 02]: page-wrapper with min-height: 100vh covers full viewport so toggle background change is seamless
- [Phase 02]: Layout centering moved from body to .page-wrapper to allow toggle-driven background on wrapper
- [Phase 03]: Used .AllTranslations (not .Translations) for hreflang to include self-reference as required by Google
- [Phase 03]: Territory-qualified language codes (it-IT, en-US) in languages.toml for OG locale compliance
- [Phase 03]: x-default hreflang points to site root, letting Hugo handle default language redirect

### Pending Todos

None yet.

### Blockers/Concerns

- Phase 1 Day 0 decisions (baseURL, defaultContentLanguageInSubdir, CNAME, DNS order) cannot be changed cheaply later -- see research/PITFALLS.md
- Exact project descriptions for Docora, Lumio, Helix -- placeholder descriptions used, may need refinement
- Typography decision resolved: system font stack chosen

## Session Continuity

Last session: 2026-02-18
Stopped at: Completed 03-01-PLAN.md -- All phases complete
Resume file: .planning/phases/03-seo-and-production-polish/03-01-SUMMARY.md
