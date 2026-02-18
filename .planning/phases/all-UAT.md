---
status: diagnosed
phase: all (01 + 02 + 03)
source: 01-01-SUMMARY.md, 01-02-SUMMARY.md, 02-01-SUMMARY.md, 02-02-SUMMARY.md, 02-03-SUMMARY.md, 03-01-SUMMARY.md
started: 2026-02-18T16:00:00Z
updated: 2026-02-18T16:15:00Z
---

## Current Test

[testing complete]

## Tests

### 1. Site loads at toto-castaldi.com
expected: Visiting https://toto-castaldi.com loads a page served by GitHub Pages with visible content.
result: issue
reported: "c'è una pagina con 'Sito in costruzione. I progetti saranno disponibili a breve.' NON funziona HTTPS"
severity: major

### 2. www redirects to apex domain
expected: Visiting https://www.toto-castaldi.com redirects to or serves the same site as toto-castaldi.com.
result: pass

### 3. Bilingual URL paths
expected: /it/ path shows a page in Italian, /en/ path shows a page in English. Each has the correct lang attribute.
result: pass

### 4. Bilingual 404 page
expected: Visiting a non-existent URL (e.g., /xyz) shows a 404 page with text in both Italian and English, with links to each homepage.
result: pass

### 5. Three project cards visible
expected: The homepage shows three project cards: Docora, Lumio, and Helix, each with a name, description, and link.
result: issue
reported: "Solo titolo e placeholder 'Sito in costruzione'. Nessuna card progetto visibile."
severity: major

### 6. Card descriptions match language
expected: On /it/ descriptions are in Italian, on /en/ descriptions are in English. Switching language changes the descriptions.
result: issue
reported: "Nessuna card visibile, solo placeholder. Non è possibile verificare le descrizioni."
severity: major

### 7. Card links to subdomain
expected: Each card links to its subdomain (docora.toto-castaldi.com, lumio.toto-castaldi.com, helix.toto-castaldi.com). Clicking a card navigates to the link.
result: issue
reported: "Blocked by same root cause: no cards rendered on deployed site"
severity: major

### 8. Language switcher works
expected: A language switcher is visible (shows "EN" on Italian page, "IT" on English page). Clicking it navigates to the same page in the other language.
result: issue
reported: "No language switcher visible on deployed site - only placeholder content"
severity: major

### 9. Responsive layout
expected: On desktop, cards are centered in a single column. On mobile, cards stack vertically with no horizontal scrolling.
result: issue
reported: "No cards or layout components visible to test - placeholder only"
severity: major

### 10. Dark mode follows OS preference
expected: With OS set to dark mode, the site renders in a dark color scheme automatically (dark background, light text). With OS in light mode, light background and dark text.
result: issue
reported: "No dark mode theming visible despite OS dark mode being active in screenshot"
severity: major

### 11. Dark mode manual toggle
expected: A sun/moon toggle icon is visible in the header. Clicking it switches between light and dark themes regardless of OS preference.
result: issue
reported: "No toggle visible in header - only title and placeholder text"
severity: major

### 12. Favicon visible
expected: A TC monogram favicon appears in the browser tab. It adapts to dark/light mode.
result: issue
reported: "No TC monogram favicon visible in browser tab"
severity: minor

### 13. Hreflang tags in page source
expected: Viewing page source on /it/ shows hreflang link tags for it-IT, en-US, and x-default. Same on /en/ with reciprocal links. All URLs are absolute.
result: issue
reported: "Cannot verify - likely missing if templates not deployed"
severity: major

### 14. Canonical URL in page source
expected: Each language page has a canonical link tag pointing to itself (e.g., /it/ page has canonical to its own absolute URL).
result: issue
reported: "Cannot verify - likely missing if templates not deployed"
severity: major

### 15. Open Graph meta tags
expected: Page source includes og:title, og:description, og:url, og:locale, og:site_name meta tags with correct values for the current language.
result: issue
reported: "Cannot verify - likely missing if templates not deployed"
severity: major

## Summary

total: 15
passed: 3
issues: 12
pending: 0
skipped: 0

## Gaps

- truth: "Site loads at toto-castaldi.com with project cards and working HTTPS"
  status: failed
  reason: "User reported: page shows 'Sito in costruzione' placeholder instead of project cards. HTTPS does not work."
  severity: major
  test: 1
  root_cause: "24 commits never pushed to origin/main. Remote has Phase 1 only. HTTPS cert is *.github.io wildcard, not custom domain Let's Encrypt cert."
  artifacts:
    - path: "all 34 changed files"
      issue: "git push never executed after Phase 1 deployment"
  missing:
    - "Run git push origin main to deploy all Phase 2+3 work"
    - "Enable Enforce HTTPS in GitHub repo Settings > Pages"
  debug_session: ".planning/debug/deployed-site-placeholder.md"
- truth: "Homepage shows three project cards: Docora, Lumio, Helix with names, descriptions, and links"
  status: failed
  reason: "User reported: only title and 'Sito in construzione' placeholder visible. No project cards."
  severity: major
  test: 5
  root_cause: "Same as test 1: code never pushed. Also typo in data/projects.toml line 13: 'SStudia' (double S)"
  artifacts:
    - path: "data/projects.toml"
      issue: "Typo 'SStudia' on line 13, plus uncommitted local changes"
  missing:
    - "Fix typo in data/projects.toml"
    - "Commit uncommitted changes"
    - "Push to remote"
  debug_session: ".planning/debug/deployed-site-placeholder.md"
- truth: "All Phase 2+3 features visible on deployed site (cards, switcher, toggle, SEO tags)"
  status: failed
  reason: "Deployed site shows Phase 1 placeholder content only. Phase 2 and 3 templates/content not rendering."
  severity: major
  test: 6-15
  root_cause: "Same as test 1: 24 commits never pushed"
  artifacts: []
  missing:
    - "Push to remote triggers full rebuild with all features"
    - "Add .gitignore with public/ and .hugo_build.lock"
  debug_session: ".planning/debug/deployed-site-placeholder.md"
