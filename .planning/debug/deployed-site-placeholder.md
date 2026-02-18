---
status: diagnosed
trigger: "Deployed site shows placeholder instead of built features"
created: 2026-02-18T00:00:00Z
updated: 2026-02-18T15:20:00Z
---

## Current Focus

hypothesis: CONFIRMED - 24 commits never pushed to origin/main
test: git branch -vv shows main ahead 24 of origin/main
expecting: n/a - root cause confirmed
next_action: report findings

## Symptoms

expected: Full site with project cards, language switcher, dark mode, hreflang/OG tags
actual: Placeholder page showing "Sito in costruzione. I progetti saranno disponibili a breve."
errors: HTTPS cert mismatch (*.github.io instead of custom domain cert)
reproduction: Visit https://toto-castaldi.com
started: All Phase 2 and 3 features were never deployed - only Phase 1 scaffold exists on remote

## Eliminated

- hypothesis: GitHub Actions workflow is broken or misconfigured
  evidence: Workflow file (.github/workflows/hugo.yaml) is correct; it builds with hugo --minify and deploys to Pages. The workflow simply hasn't run with Phase 2/3 code because it was never pushed.
  timestamp: 2026-02-18T15:19:00Z

- hypothesis: Templates or data files are broken locally
  evidence: layouts/index.html has project cards, data/projects.toml has 3 projects, all templates exist locally. Local build works fine.
  timestamp: 2026-02-18T15:19:00Z

## Evidence

- timestamp: 2026-02-18T15:18:00Z
  checked: git branch -vv
  found: "* main f855254 [origin/main: ahead 24]" - local main is 24 commits ahead of origin/main
  implication: PRIMARY ROOT CAUSE - all Phase 2 and 3 work was committed locally but never pushed

- timestamp: 2026-02-18T15:18:30Z
  checked: git log origin/main --oneline -5
  found: Last commit on remote is "055744a fix(01-02): add mkdir -p for Dart Sass install directory" - only Phase 1 scaffold
  implication: Remote branch has only the initial scaffold with placeholder content

- timestamp: 2026-02-18T15:18:45Z
  checked: git show origin/main:content/_index.it.md
  found: Contains "Sito in costruzione. I progetti saranno disponibili a breve." - the exact placeholder text seen on deployed site
  implication: Confirms deployed content matches origin/main, not local main

- timestamp: 2026-02-18T15:19:00Z
  checked: git show origin/main:data/projects.toml
  found: FILE NOT FOUND - data directory does not exist on remote
  implication: Project data was never pushed

- timestamp: 2026-02-18T15:19:15Z
  checked: git diff --stat origin/main..HEAD
  found: 34 files changed, 3438 insertions - massive delta between local and remote
  implication: All of Phase 2 (data, CSS, templates, i18n, dark mode) and Phase 3 (SEO, hreflang, OG tags) exist only locally

- timestamp: 2026-02-18T15:19:30Z
  checked: curl -sk https://toto-castaldi.com/it/
  found: Served HTML shows Phase 1 placeholder with no cards, no switcher, no dark mode
  implication: Deployed site matches origin/main content exactly

- timestamp: 2026-02-18T15:19:45Z
  checked: curl -svI https://toto-castaldi.com (verbose SSL)
  found: SSL cert subject is "CN=*.github.io" not custom domain; curl exits with code 60 (cert validation failure)
  implication: SECONDARY ISSUE - Let's Encrypt cert for custom domain not provisioned. May need "Enforce HTTPS" enabled in GitHub Pages settings after push

- timestamp: 2026-02-18T15:20:00Z
  checked: .gitignore existence
  found: No .gitignore file exists in repo
  implication: SECONDARY ISSUE - public/ directory (Hugo build output) is untracked but not ignored; risk of accidentally committing build artifacts

- timestamp: 2026-02-18T15:20:15Z
  checked: data/projects.toml local content
  found: Line 13 has typo "SStudia" (double S) in Lumio Italian description
  implication: MINOR ISSUE - typo will be visible once site is deployed

## Resolution

root_cause: 24 commits (all of Phase 2 and Phase 3 work) exist only on local main branch and were never pushed to origin/main. The GitHub Actions workflow has only ever built the Phase 1 scaffold code, which contains placeholder text.
fix:
verification:
files_changed: []
