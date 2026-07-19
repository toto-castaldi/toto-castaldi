---
phase: quick-260719-flq
plan: 01
subsystem: landing-page
tags: [hugo, static-assets, svg, bugfix]
requires: []
provides:
  - Self-hosted Lumio logo at /images/lumio-logo.svg
  - Lumio card renders logo on IT and EN pages
affects: []
tech-stack:
  added: []
  patterns:
    - Self-host cross-origin assets under static/ to avoid external 404s
key-files:
  created:
    - static/images/lumio-logo.svg
  modified:
    - data/projects.toml
decisions:
  - Self-host Lumio logo instead of depending on lumio.toto-castaldi.com (which serves its logo only as inline SVG, no hosted file)
metrics:
  duration: ~2 min
  completed: 2026-07-19
requirements-completed: [QUICK-FIX-LUMIO-LOGO]
---

# Quick Task 260719-flq: Fix Lumio Logo Missing on Landing Page — Summary

Self-hosted the Lumio tri-color logo as `static/images/lumio-logo.svg` and repointed `data/projects.toml` from the 404-ing `https://lumio.toto-castaldi.com/logo.svg` to the local `/images/lumio-logo.svg`.

## What Was Done

### Task 1: Create self-hosted Lumio logo SVG
- Created `static/images/lumio-logo.svg` with the canonical markup verbatim from the plan (tri-color pie: 3 `<path>` slices + 3 `<rect>` rays, fills `#FFA726` / `#FF7061` / `#9C68D4`, `viewBox="0 0 400 300"`, no width/height on root).
- Commit: `a11fc89`

### Task 2: Point projects.toml at local logo and verify build
- Changed the Lumio `logo` field in `data/projects.toml` to `/images/lumio-logo.svg`. Docora entry untouched.
- `hugo --minify` succeeds; `public/images/lumio-logo.svg` present in output.
- Commit: `3b75f39`

## Verification

- `hugo --minify` exits 0.
- `public/images/lumio-logo.svg` exists in build output.
- `public/it/index.html` and `public/en/index.html` both contain `src=/images/lumio-logo.svg` on the Lumio card.
- No remaining reference to `lumio.toto-castaldi.com/logo.svg` in `data/`.

Note: the plan's verify step grepped `public/index.html`, but the site root `index.html` is only a language-redirect alias to `/it/`. The real rendered pages are `public/it/index.html` and `public/en/index.html` — both verified. This matches the plan's own `<verification>` section, which correctly lists the localized pages.

## Deviations from Plan

None - plan executed exactly as written.

## Known Stubs

None.

## Threat Flags

None — change removes a cross-origin image dependency; SVG contains no scripts and is rendered via `<img>`.

## Pre-existing Issues (out of scope, not touched)

- Hugo build WARN: "found no layout file for html for kind taxonomy" — pre-existing, unrelated to this change.

## Self-Check: PASSED

- static/images/lumio-logo.svg: FOUND
- data/projects.toml updated: FOUND
- Commits a11fc89, 3b75f39: FOUND

