# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-17)

**Core value:** Visitors can immediately see what projects exist and navigate to each one -- in their preferred language.
**Current focus:** Phase 2 in progress. Plan 01 complete, Plan 02 next.

## Current Position

Phase: 2 of 3 (Bilingual Site with Design)
Plan: 1 of 2 in current phase
Status: In Progress
Last activity: 2026-02-18 -- Completed 02-01-PLAN.md

Progress: [######....] 50%

## Performance Metrics

**Velocity:**
- Total plans completed: 3
- Average duration: 7min
- Total execution time: 0.35 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1 | 2 | 19min | 10min |

**Recent Trend:**
- Last 5 plans: -
- Trend: -

*Updated after each plan completion*
| Phase 01 P01 | 2min | 2 tasks | 13 files |
| Phase 01 P02 | 17min | 3 tasks | 1 files |
| Phase 02 P01 | 2min | 2 tasks | 6 files |

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

### Pending Todos

None yet.

### Blockers/Concerns

- Phase 1 Day 0 decisions (baseURL, defaultContentLanguageInSubdir, CNAME, DNS order) cannot be changed cheaply later -- see research/PITFALLS.md
- Exact project descriptions for Docora, Lumio, Helix -- placeholder descriptions used, may need refinement
- Typography decision resolved: system font stack chosen

## Session Continuity

Last session: 2026-02-18
Stopped at: Completed 02-01-PLAN.md
Resume file: .planning/phases/02-bilingual-site-with-design/02-01-SUMMARY.md
