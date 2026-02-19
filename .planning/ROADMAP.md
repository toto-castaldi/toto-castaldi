# Roadmap: toto-castaldi.com

## Milestones

- ✅ **v1.0 MVP** -- Phases 1-3 (shipped 2026-02-19)
- **v1.1 Restyling Bento Grid** -- Phases 4-6 (in progress)

## Phases

<details>
<summary>v1.0 MVP (Phases 1-3) -- SHIPPED 2026-02-19</summary>

- [x] Phase 1: Foundation and Deploy Pipeline (2/2 plans) -- completed 2026-02-18
- [x] Phase 2: Bilingual Site with Design (3/3 plans) -- completed 2026-02-18
- [x] Phase 3: SEO and Production Polish (1/1 plan) -- completed 2026-02-18

</details>

### v1.1 Restyling Bento Grid

- [ ] **Phase 4: Design Tokens and Container** - Fix WCAG AA contrast in both themes, widen container to support 3-column grid
- [ ] **Phase 5: Bento Grid Layout** - CSS Grid with responsive breakpoints, template restructure into bento cells
- [ ] **Phase 6: Visual Polish and Extras** - Card depth, hover effects, additional bento sections

## Phase Details

### Phase 4: Design Tokens and Container
**Goal**: Both themes pass WCAG AA contrast and the page container is wide enough for a 3-column grid
**Depends on**: Nothing (first phase of v1.1)
**Requirements**: LAYOUT-04, THEME-01, THEME-02, THEME-03, THEME-04, THEME-05, STYLE-05
**Success Criteria** (what must be TRUE):
  1. Light mode text and borders all pass WCAG AA contrast ratios (4.5:1 text, 3:1 UI components)
  2. Dark mode text and borders all pass WCAG AA contrast ratios (4.5:1 text, 3:1 UI components)
  3. Cards have a visually distinct background from the page in both light and dark mode
  4. Dark/light toggle still works in all 4 states (OS-light unchecked, OS-light checked, OS-dark unchecked, OS-dark checked)
  5. Page container is wide enough (~1080px) that 3 columns of content would fit comfortably
**Plans:** 1 plan

Plans:
- [ ] 04-01-PLAN.md -- WCAG AA contrast tokens + container restructure

### Phase 5: Bento Grid Layout
**Goal**: Landing page displays project cards in a responsive bento grid that adapts from 3 columns (desktop) to 1 column (mobile)
**Depends on**: Phase 4
**Requirements**: LAYOUT-01, LAYOUT-02, LAYOUT-03, STYLE-01, STYLE-04
**Success Criteria** (what must be TRUE):
  1. Desktop viewport (>1024px) shows 3 project cards side by side in a CSS Grid
  2. Tablet viewport (481-1024px) shows 2 columns with cards reflowing
  3. Mobile viewport (<=480px) shows 1 column with stacked cards
  4. Cards have rounded corners and consistent typography across all breakpoints
  5. Dark mode toggle still works after template restructure (no sibling selector breakage)
**Plans:** 1 plan

Plans:
- [ ] 05-01-PLAN.md -- Responsive grid columns + border-radius token + typography verification

### Phase 6: Visual Polish and Extras
**Goal**: Cards have depth and interactivity, and the page includes additional bento sections beyond project cards
**Depends on**: Phase 5
**Requirements**: LAYOUT-05, STYLE-02, STYLE-03
**Success Criteria** (what must be TRUE):
  1. Project cards show shadow/depth in light mode and elevation change on hover
  2. Hover effects are visible and smooth, disabled when user prefers reduced motion
  3. At least one additional bento section exists beyond project cards (tech stack, GitHub link, or similar)
**Plans**: TBD

## Progress

**Execution Order:** Phase 4 -> 5 -> 6

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Foundation and Deploy Pipeline | v1.0 | 2/2 | Complete | 2026-02-18 |
| 2. Bilingual Site with Design | v1.0 | 3/3 | Complete | 2026-02-18 |
| 3. SEO and Production Polish | v1.0 | 1/1 | Complete | 2026-02-18 |
| 4. Design Tokens and Container | v1.1 | 0/? | Not started | - |
| 5. Bento Grid Layout | v1.1 | 0/? | Not started | - |
| 6. Visual Polish and Extras | v1.1 | 0/? | Not started | - |
