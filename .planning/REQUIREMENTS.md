# Requirements: toto-castaldi.com

**Defined:** 2026-02-19
**Core Value:** Visitors can immediately see what projects exist and navigate to each one -- in their preferred language.

## v1.1 Requirements

Requirements for the bento grid restyling milestone. Each maps to roadmap phases.

### Layout

- [ ] **LAYOUT-01**: Landing page uses CSS Grid with 3 columns on desktop (>1024px)
- [ ] **LAYOUT-02**: Grid adapts to 2 columns on tablet (481-1024px)
- [ ] **LAYOUT-03**: Grid adapts to 1 column on mobile (<=480px)
- [ ] **LAYOUT-04**: Container max-width increases from 680px to ~1080px
- [ ] **LAYOUT-05**: Additional bento sections beyond project cards (tech stack, GitHub link, or similar)

### Theme

- [ ] **THEME-01**: Light mode legible with WCAG AA contrast on all elements
- [ ] **THEME-02**: Dark mode legible with WCAG AA contrast on all elements
- [ ] **THEME-03**: Dark/light toggle functional (checkbox hack preserved)
- [ ] **THEME-04**: Cards visually distinct from background in both themes
- [ ] **THEME-05**: Border contrast >= 3:1 for UI components (WCAG SC 1.4.11)

### Styling

- [ ] **STYLE-01**: Cards with rounded corners (border-radius)
- [ ] **STYLE-02**: Cards with shadows/depth in light mode
- [ ] **STYLE-03**: Hover effects on project cards
- [ ] **STYLE-04**: Consistent typography with modular scale
- [ ] **STYLE-05**: Off-white page background in light mode to differentiate cards

## Future Requirements

### Theme Enhancements

- **THEME-06**: Per-project accent colors (card top borders)
- **THEME-07**: Migrate from checkbox hack to `:has()` selector
- **THEME-08**: localStorage theme persistence (requires JS)

### Polish

- **STYLE-06**: Print stylesheet
- **STYLE-07**: CSS transitions with `prefers-reduced-motion` respect

## Out of Scope

| Feature | Reason |
|---------|--------|
| SCSS migration | Under 500 lines CSS, custom properties sufficient |
| Container Queries | Single-page, media queries suffice |
| CSS Subgrid | Flat layout, not needed |
| JavaScript | Zero-JS constraint maintained |
| Tailwind CSS | Hugo + plain CSS handles everything natively |
| About/bio section | Focus on projects only (unchanged from v1.0) |
| Contact form | Minimal showcase, not a portfolio |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| LAYOUT-01 | Phase 5 | Pending |
| LAYOUT-02 | Phase 5 | Pending |
| LAYOUT-03 | Phase 5 | Pending |
| LAYOUT-04 | Phase 4 | Pending |
| LAYOUT-05 | Phase 6 | Pending |
| THEME-01 | Phase 4 | Pending |
| THEME-02 | Phase 4 | Pending |
| THEME-03 | Phase 4 | Pending |
| THEME-04 | Phase 4 | Pending |
| THEME-05 | Phase 4 | Pending |
| STYLE-01 | Phase 5 | Pending |
| STYLE-02 | Phase 6 | Pending |
| STYLE-03 | Phase 6 | Pending |
| STYLE-04 | Phase 5 | Pending |
| STYLE-05 | Phase 4 | Pending |

**Coverage:**
- v1.1 requirements: 15 total
- Mapped to phases: 15
- Unmapped: 0

---
*Requirements defined: 2026-02-19*
*Last updated: 2026-02-19 after roadmap creation*
