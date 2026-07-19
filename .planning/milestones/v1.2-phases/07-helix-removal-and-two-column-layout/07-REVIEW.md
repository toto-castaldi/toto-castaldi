---
phase: 07-helix-removal-and-two-column-layout
reviewed: 2026-07-19T08:55:39Z
depth: standard
files_reviewed: 4
files_reviewed_list:
  - assets/css/main.css
  - content/_index.en.md
  - content/_index.it.md
  - data/projects.toml
findings:
  critical: 0
  warning: 0
  info: 4
  total: 4
status: issues_found
---

# Phase 07: Code Review Report

**Reviewed:** 2026-07-19T08:55:39Z
**Depth:** standard
**Files Reviewed:** 4
**Status:** issues_found

## Summary

Reviewed the Helix removal (data entry, IT/EN SEO descriptions) and the desktop grid reflow from 3 to 2 columns. The phase's core changes are correct:

- `data/projects.toml` is valid TOML with exactly two `[[project]]` entries (Docora, Lumio); the Helix table, URL, logo, and both descriptions are fully removed.
- SEO descriptions in `content/_index.it.md` and `content/_index.en.md` are updated symmetrically ("Docora e Lumio" / "Docora and Lumio") and match the two remaining projects.
- Verified via repo-wide grep: no leftover "helix" references in any source file (CSS, templates, i18n, config, content, data, CLAUDE.md). Remaining mentions exist only in `.planning/` historical artifacts, which is expected. The generated `public/` directory is gitignored and not committed.
- Verified via a Hugo build (`hugo --minify` to scratchpad): build succeeds, both language pages render exactly 2 `<article class="project-card">` elements, meta descriptions match the updated front matter, and no "Helix" string appears anywhere in the rendered output.
- Breakpoint analysis: mobile (≤480px) 1 column, tablet (≥481px) 2 columns, desktop (≥1025px) 2 columns — no gap or conflict between breakpoints; later rules do not fight earlier ones.

No Critical or Warning issues found. Four Info-level quality observations below; only IN-01 was introduced by this phase's diff, the rest are pre-existing conditions in the reviewed files.

## Info

### IN-01: Redundant desktop media query after 3-to-2 column reflow

**File:** `assets/css/main.css:217-222`
**Issue:** The change replaced `repeat(3, 1fr)` with `repeat(2, 1fr)` inside `@media (min-width: 1025px)`, making the block byte-for-byte identical in effect to the tablet rule at `@media (min-width: 481px)` (lines 210-215). The desktop block is now dead code: it can never produce a different layout, and a future editor may change one block without realizing the other exists.
**Fix:** Delete the redundant block and update the tablet comment to document the full range:

```css
/* Tablet and desktop (>=481px): 2 columns */
@media (min-width: 481px) {
  .project-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

If a distinct desktop layout is anticipated soon, keep the block but add a comment explaining why it is intentionally identical.

### IN-02: IT/EN description mismatch for Docora entry

**File:** `data/projects.toml:6-7`
**Issue:** The two language descriptions for Docora are not equivalent translations. IT: "Controllo e notifica di cambiamenti su repository GitHub" (monitoring and notification of GitHub repository changes). EN: "Headless GitHub repository monitoring with push-based file notifications" — the EN adds "headless" and "push-based file" qualifiers absent from the IT. Pre-existing (unchanged by this phase), but it is an i18n content drift in a reviewed file. By contrast, the Lumio entry (lines 14-15) is faithfully translated.
**Fix:** Align the IT description, e.g. `it = "Monitoraggio headless di repository GitHub con notifiche push sui file"`, or simplify the EN to match the IT.

### IN-03: Dark-mode token set duplicated four times

**File:** `assets/css/main.css:58-116`
**Issue:** The full dark token set (7 colors + 2 shadows) is duplicated in four places: `:root` under `prefers-color-scheme: dark` (59-69), `.page-wrapper` in the same media query (71-81), and the checked-toggle override (87-97); the light set is likewise duplicated between `:root` (17-53) and the dark-OS checked override (102-116). Any future palette change must be applied in multiple locations, and a missed one silently causes theme drift. Pre-existing, not touched by this phase.
**Fix:** Consolidate each palette into a reusable declaration, e.g. define the token sets once on shared selector lists (`:root` dark media + `#dark-toggle:checked + .page-wrapper` share one block), or adopt a single `[data-theme]`/custom-property indirection so each palette exists exactly once.

### IN-04: `body` background not switched by the manual dark-mode toggle

**File:** `assets/css/main.css:121-134` (interaction with 87-97 and 102-116)
**Issue:** `body { background: var(--color-bg) }` resolves `--color-bg` from `:root`, but the checkbox-toggle overrides (lines 87-97, 103-115) only redefine tokens on `.page-wrapper`. When the user toggles the theme against the OS preference, `.page-wrapper` switches but `body` keeps the OS-preference background. Because `.page-wrapper` has `min-height: 100vh` this is normally hidden, but it becomes visible as a mismatched band during overscroll/rubber-banding (macOS/iOS) or if wrapper height ever falls short. Pre-existing, not touched by this phase.
**Fix:** Drop the `background` declaration from `body` (the wrapper already paints it), or move the page background entirely to `.page-wrapper` and let `html`/`body` stay transparent so the toggled color is the only one painted.

---

_Reviewed: 2026-07-19T08:55:39Z_
_Reviewer: Claude (gsd-code-reviewer)_
_Depth: standard_
