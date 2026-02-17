# Phase 1: Foundation and Deploy Pipeline - Context

**Gathered:** 2026-02-17
**Status:** Ready for planning

<domain>
## Phase Boundary

Get a blank but deployable bilingual Hugo site live at toto-castaldi.com with HTTPS, correct `/it/` and `/en/` URL paths, GitHub Actions auto-deploy from main, and stable custom domain. Real content (project cards, design, language switcher) comes in Phase 2.

</domain>

<decisions>
## Implementation Decisions

### Default language & URL paths
- Default language: Claude's discretion (choose based on best practices for multilingual Hugo + GitHub Pages)
- URL structure (defaultContentLanguageInSubdir): Claude's discretion — pick the approach that works best for this stack and future phases
- This is a Day 0 decision — cannot be cheaply changed later

### Root URL behavior
- Root URL (`/`) behavior: Claude's discretion — pick the best approach for Hugo static sites on GitHub Pages
- 404 page: Claude's discretion — handle appropriately for Phase 1
- Brief downtime during DNS cutover from current host to GitHub Pages is acceptable
- Subdomains (docora/lumio/helix.toto-castaldi.com) are already handled separately — do not touch them

### Placeholder page content
- Placeholder must be in both Italian and English — validates bilingual routing works (Phase 1 success criterion)
- Content and styling: Claude's discretion — whatever minimal content makes sense
- Display name: "Antonio Toto Castaldi"

### Claude's Discretion
- Default language choice (it vs en)
- URL structure (both in subdirectories vs default at root)
- Root URL redirect behavior
- 404 page design and behavior
- Placeholder page content and styling beyond the name
- All technical implementation (Hugo config, GitHub Actions, DNS setup)

</decisions>

<specifics>
## Specific Ideas

- The current site is hosted elsewhere (not GitHub Pages) — DNS will need to be switched over
- Subdomains for the three projects are already configured and working — leave them alone
- The display name is "Antonio Toto Castaldi"

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-foundation-and-deploy-pipeline*
*Context gathered: 2026-02-17*
