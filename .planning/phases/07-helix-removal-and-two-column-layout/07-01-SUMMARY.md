---
phase: 07-helix-removal-and-two-column-layout
plan: 01
subsystem: ui
tags: [hugo, i18n, seo, content, toml]

requires:
  - phase: v1.1
    provides: bento grid layout rendering project cards from data/projects.toml
provides:
  - Helix-free project data (data/projects.toml with exactly two entries)
  - Helix-free SEO descriptions in IT and EN front matter
  - Repo docs (CLAUDE.md) naming only Docora and Lumio
affects: [07-02-two-column-layout]

tech-stack:
  added: []
  patterns: []

key-files:
  created: []
  modified:
    - data/projects.toml
    - content/_index.it.md
    - content/_index.en.md
    - CLAUDE.md

key-decisions:
  - "Removed Helix source references entirely rather than hiding via draft flag — project is decommissioned"

patterns-established: []

requirements-completed: [CONT-07, CONT-08]

duration: 4min
completed: 2026-07-19
---

# Phase 7 Plan 01: Helix Removal Summary

**Removed the Helix project from all shipped artifacts — project data, IT/EN SEO descriptions, and repo docs — leaving a clean two-project (Docora, Lumio) Hugo build with zero Helix matches.**

## Performance

- **Duration:** ~4 min
- **Tasks:** 2
- **Files modified:** 4

## Accomplishments
- Deleted the Helix `[[project]]` entry from `data/projects.toml`, leaving exactly two entries (Docora, Lumio)
- Updated IT and EN front-matter `description` meta to list only Docora and Lumio
- Updated `CLAUDE.md` to name two projects and drop the helix subdomain mention
- Verified a clean `hugo --minify` build renders exactly two project cards per language with zero Helix references in source or built output

## Task Commits

Each task was committed atomically:

1. **Task 1: Remove the Helix entry from project data** - `80a6d32` (feat)
2. **Task 2: Remove Helix from SEO descriptions and repo docs** - `6ab8a7e` (feat)

## Files Created/Modified
- `data/projects.toml` - Removed the third `[[project]]` block (Helix); now two entries
- `content/_index.it.md` - `description` now "Progetti software di Toto Castaldi: Docora e Lumio"
- `content/_index.en.md` - `description` now "Software projects by Toto Castaldi: Docora and Lumio"
- `CLAUDE.md` - Project section names two projects; Content section drops helix subdomain

## Decisions Made
None - followed plan as specified.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None. The `hugo --minify` build emitted a pre-existing, unrelated `taxonomy` layout warning (not introduced by this plan) and exited 0.

## Verification Results
- `hugo --minify` exits 0
- `grep -ri 'helix' data/ content/ layouts/ i18n/` → zero matches
- `grep -ril 'helix' public/` after fresh build → zero matches
- `public/it/index.html` and `public/en/index.html` each render exactly two `<article>` project cards (Docora, Lumio)

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Data and content are now two-project; plan 07-02 can reflow the bento grid to a two-column desktop layout with the GitHub profile card full-width below.
- Note (pre-existing blocker, out of scope): helix.toto-castaldi.com subdomain redirect/teardown is not handled in this repo.

## Self-Check: PASSED

All modified files present and all task/doc commits (`80a6d32`, `6ab8a7e`, `c8b2900`) verified in git history.

---
*Phase: 07-helix-removal-and-two-column-layout*
*Completed: 2026-07-19*
