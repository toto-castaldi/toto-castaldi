---
phase: 02-bilingual-site-with-design
plan: 01
subsystem: ui
tags: [hugo-data, i18n, css-custom-properties, dark-mode, fluid-typography, toml, wcag-aa]

# Dependency graph
requires:
  - phase: 01-foundation-and-deploy-pipeline
    provides: "Hugo project scaffold with bilingual config, CSS pipeline, and template structure"
provides:
  - "Project data for Docora, Lumio, Helix in data/projects.toml with bilingual descriptions"
  - "i18n strings (tagline, languageSwitch, viewProject) in both Italian and English"
  - "Content front matter with meta descriptions per language"
  - "Complete CSS design system with light/dark mode tokens, fluid typography, and layout components"
affects: [02-02-templates-and-components]

# Tech tracking
tech-stack:
  added: []
  patterns: [css-custom-properties-design-tokens, prefers-color-scheme-dark-mode, clamp-fluid-typography, toml-array-of-tables-data, pseudo-element-card-overlay]

key-files:
  created:
    - data/projects.toml
  modified:
    - i18n/en.toml
    - i18n/it.toml
    - content/_index.en.md
    - content/_index.it.md
    - assets/css/main.css

key-decisions:
  - "System font stack (system-ui, -apple-system, sans-serif) for zero network requests and sub-1s load"
  - "Single-column centered layout (680px max-width) with vertically stacked cards at all viewports"
  - "Bordered cards with rounded corners and hover border-color transition for visual containment"
  - "Whole-card clickable via ::after pseudo-element overlay pattern (zero JS)"
  - "WCAG AA verified color pairs for both light and dark modes"

patterns-established:
  - "CSS design tokens: all colors, type sizes, spacing defined as custom properties on :root"
  - "Dark mode: @media (prefers-color-scheme: dark) overrides color tokens only"
  - "Fluid typography: clamp() with viewport-relative middle value for smooth scaling"
  - "TOML data: [[project]] array-of-tables with nested [project.description] map keyed by language code"
  - "i18n format: [key] other = 'value' pattern for Hugo T function"

requirements-completed: [CONT-02, CONT-03, CONT-04, CONT-05, DSGN-01, DSGN-02, DSGN-03, DSGN-04, UI-04, UI-05]

# Metrics
duration: 2min
completed: 2026-02-18
---

# Phase 2 Plan 1: Data Layer and CSS Design System Summary

**Bilingual project data in TOML with three projects, expanded i18n strings, and a complete CSS design system with dark mode tokens, fluid clamp() typography, and card layout primitives**

## Performance

- **Duration:** 2 min
- **Started:** 2026-02-18T14:01:20Z
- **Completed:** 2026-02-18T14:02:50Z
- **Tasks:** 2
- **Files modified:** 6

## Accomplishments
- Created `data/projects.toml` with Docora, Lumio, and Helix project entries including bilingual descriptions, URLs, and logo paths
- Expanded i18n string files in both Italian and English with tagline, languageSwitch, and viewProject keys
- Added meta descriptions to content front matter for both languages and removed placeholder body text
- Built a complete CSS design system: reset, color tokens (light + dark), modular type scale with clamp(), spacing tokens, base element styles, and all component classes for Plan 02 templates

## Task Commits

Each task was committed atomically:

1. **Task 1: Create project data, i18n strings, and content descriptions** - `f333368` (feat)
2. **Task 2: Build CSS design system with dark mode, typography scale, and layout primitives** - `f5cf8c1` (feat)

## Files Created/Modified
- `data/projects.toml` - Three project entries with name, url, logo, and bilingual description map
- `i18n/en.toml` - English UI translation strings (siteTitle, tagline, languageSwitch, viewProject)
- `i18n/it.toml` - Italian UI translation strings (siteTitle, tagline, languageSwitch, viewProject)
- `content/_index.en.md` - English homepage with description meta, no placeholder body
- `content/_index.it.md` - Italian homepage with description meta, no placeholder body
- `assets/css/main.css` - Complete design system: CSS reset, design tokens, dark mode, typography scale, base styles, layout components

## Decisions Made
- System font stack chosen over web fonts for zero network requests and sub-1s page load
- Centered single-column layout (680px max-width) chosen for 3 cards -- avoids orphan card in grid, matches airy aesthetic
- Bordered cards with subtle rounded corners -- clean visual separation without heavy shadows
- Whole-card clickable via CSS ::after pseudo-element overlay -- no JavaScript required
- Color pairs verified for WCAG AA compliance in both light and dark modes

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- All data and i18n strings ready for template rendering in Plan 02
- CSS design system defines all tokens and component classes needed by Plan 02 templates
- Content front matter has descriptions ready for meta tag rendering
- Hugo builds cleanly with all new files

## Self-Check: PASSED

All 6 files verified present (1 created, 5 modified). Both task commits (f333368, f5cf8c1) verified in git log.

---
*Phase: 02-bilingual-site-with-design*
*Completed: 2026-02-18*
