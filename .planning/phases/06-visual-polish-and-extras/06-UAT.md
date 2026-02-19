---
status: complete
phase: 06-visual-polish-and-extras
source: [06-01-SUMMARY.md]
started: 2026-02-19T14:00:00Z
updated: 2026-02-19T14:10:00Z
---

## Current Test

[testing complete]

## Tests

### 1. Card shadows in light mode
expected: In light mode, project cards show a subtle layered shadow giving them visual depth against the page background.
result: pass

### 2. No shadows in dark mode
expected: In dark mode, project cards show NO shadow — depth comes from background contrast and border only.
result: pass

### 3. Toggle states preserve shadow behavior
expected: Toggle dark mode on (from light OS): shadows disappear. Toggle back off: shadows return. If OS is dark: cards have no shadow; toggle on (to light): shadows appear.
result: pass

### 4. Hover lift and shadow escalation
expected: Hovering a project card lifts it up ~2px and the shadow becomes more pronounced (smooth animation). This is the behavior for users without reduced-motion preference.
result: pass

### 5. Border-color hover for all users
expected: Hovering a project card changes its border color (to the link/accent color) regardless of motion preferences. This is the universal hover feedback.
result: pass

### 6. GitHub bento section visible
expected: Below the project cards, a GitHub section appears with the GitHub Octicon icon and the text "GitHub". Clicking it navigates to https://github.com/toto-castaldi.
result: pass

### 7. GitHub section bilingual
expected: On the Italian page (/it/), the GitHub section text shows "GitHub" and the section label is "Link". On the English page (/en/), text shows "GitHub" and section label is "Links".
result: pass

## Summary

total: 7
passed: 7
issues: 0
pending: 0
skipped: 0

## Gaps

[none]
