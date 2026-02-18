---
phase: 01-foundation-and-deploy-pipeline
plan: 02
subsystem: infra
tags: [github-pages, custom-domain, dns, https, deployment, github-actions]

# Dependency graph
requires:
  - phase: 01-foundation-and-deploy-pipeline
    provides: "Complete Hugo project scaffold with bilingual /it/ and /en/ URL structure"
provides:
  - "Live site at toto-castaldi.com served by GitHub Pages"
  - "Custom domain with DNS A records and www CNAME"
  - "GitHub Actions deployment pipeline verified end-to-end"
  - "Bilingual /it/ and /en/ paths serving correct language content"
  - "Bilingual 404 page working on custom domain"
affects: [02-bilingual-site-with-design]

# Tech tracking
tech-stack:
  added: [github-pages-custom-domain, lets-encrypt]
  patterns: [github-actions-deploy-pages, cname-persistence]

key-files:
  created: []
  modified:
    - .github/workflows/hugo.yaml

key-decisions:
  - "HTTPS enforcement deferred to async Let's Encrypt provisioning -- site functional over HTTP immediately, HTTPS cert auto-provisions within 1 hour"
  - "DNS A records for apex domain (4x GitHub Pages IPs) + CNAME for www -> toto-castaldi.github.io"

patterns-established:
  - "Custom domain set in GitHub API before DNS records to prevent domain takeover window"
  - "CNAME file in static/ persists custom domain across GitHub Actions deploys"

requirements-completed: [INFR-02]

# Metrics
duration: 17min
completed: 2026-02-18
---

# Phase 1 Plan 2: GitHub Pages Deployment and Custom Domain Summary

**Live deployment at toto-castaldi.com with DNS, GitHub Actions pipeline, bilingual content, and HTTPS certificate provisioning in progress**

## Performance

- **Duration:** 17 min (includes ~10 min waiting for Let's Encrypt certificate provisioning)
- **Started:** 2026-02-18T12:01:53Z
- **Completed:** 2026-02-18T12:19:13Z
- **Tasks:** 3 (2 from previous agent, 1 in this continuation)
- **Files modified:** 1

## Accomplishments
- GitHub Pages configured with "GitHub Actions" source and custom domain via `gh api`
- DNS records verified: apex A records (4x GitHub Pages IPs) and www CNAME to toto-castaldi.github.io
- Live site serving bilingual content: `/it/` (Italian), `/en/` (English) with correct `lang` attributes
- www.toto-castaldi.com 301 redirects to apex domain
- Bilingual 404 page working (Italian + English with links to respective home pages)
- Custom domain persists across deploys (CNAME file in `static/`)

## Task Commits

Each task was committed atomically:

1. **Task 1: Configure GitHub Pages source and trigger first deployment** - `055744a` (fix)
2. **Task 2: User configures DNS records for custom domain** - (human action, no commit)
3. **Task 3: Verify HTTPS enforcement and full deployment** - (verification only, no code changes)

## Files Created/Modified
- `.github/workflows/hugo.yaml` - Added `mkdir -p` for Dart Sass install directory (Task 1 fix)

## Decisions Made
- HTTPS enforcement via API deferred: Let's Encrypt certificate provisioning is async and can take up to 1 hour after DNS propagation. The site is fully functional over HTTP and HTTPS (via GitHub's wildcard cert). The custom domain cert will auto-provision and HTTPS can be enforced afterward via Settings or API.
- DNS A records chosen over ALIAS/ANAME: standard GitHub Pages recommended setup with 4 A records for apex + CNAME for www.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Added mkdir -p for Dart Sass install directory**
- **Found during:** Task 1 (GitHub Actions deployment)
- **Issue:** Dart Sass install step failed because the target directory did not exist
- **Fix:** Added `mkdir -p` before the Dart Sass extraction command in the workflow
- **Files modified:** `.github/workflows/hugo.yaml`
- **Verification:** Workflow run completed successfully after fix
- **Committed in:** `055744a`

---

**Total deviations:** 1 auto-fixed (1 blocking)
**Impact on plan:** Necessary fix for CI pipeline to function. No scope creep.

## Issues Encountered

- **Let's Encrypt certificate not yet provisioned:** After DNS propagation, the GitHub Pages Let's Encrypt certificate takes up to 1 hour to provision. The `gh api` call to enable `https_enforced=true` returns "The certificate does not exist yet" during this window. This is expected GitHub Pages behavior. The site is accessible over HTTPS via GitHub's wildcard `*.github.io` certificate in the interim. Once the custom domain cert is issued, HTTPS enforcement can be enabled via repo Settings > Pages > "Enforce HTTPS" checkbox or via API.

## User Setup Required

**HTTPS enforcement pending.** After the Let's Encrypt certificate is provisioned (check in ~30-60 minutes), enable HTTPS enforcement:
- Go to repo Settings > Pages > check "Enforce HTTPS"
- Or run: `gh api repos/toto-castaldi/toto-castaldi/pages --method PUT --field https_enforced=true`

## Next Phase Readiness
- Site is live and accessible at http://toto-castaldi.com with bilingual content
- GitHub Actions deployment pipeline verified end-to-end
- HTTPS will auto-enable once Let's Encrypt cert provisions (no code changes needed)
- Phase 2 can proceed with content and design work immediately

## Self-Check: PASSED

All files verified present:
- .github/workflows/hugo.yaml: FOUND
- static/CNAME: FOUND
- Task 1 commit 055744a: FOUND
- 01-02-SUMMARY.md: FOUND

---
*Phase: 01-foundation-and-deploy-pipeline*
*Completed: 2026-02-18*
