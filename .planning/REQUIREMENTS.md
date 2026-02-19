# Requirements: toto-castaldi.com

**Defined:** 2026-02-19
**Core Value:** Visitors can immediately see what projects exist and navigate to each one -- in their preferred language.

## v1.1 Requirements

Requirements for the bento grid restyling milestone. Each maps to roadmap phases.

### Layout

- [x] **LAYOUT-01**: Landing page uses CSS Grid with 3 columns on desktop (>1024px)
- [x] **LAYOUT-02**: Grid adapts to 2 columns on tablet (481-1024px)
- [x] **LAYOUT-03**: Grid adapts to 1 column on mobile (<=480px)
- [x] **LAYOUT-04**: Container max-width increases from 680px to ~1080px
- [ ] **LAYOUT-05**: Additional bento sections beyond project cards (tech stack, GitHub link, or similar)

### Theme

- [x] **THEME-01**: Light mode legible with WCAG AA contrast on all elements
- [x] **THEME-02**: Dark mode legible with WCAG AA contrast on all elements
- [x] **THEME-03**: Dark/light toggle functional (checkbox hack preserved)
- [x] **THEME-04**: Cards visually distinct from background in both themes
- [x] **THEME-05**: Border contrast >= 3:1 for UI components (WCAG SC 1.4.11)

### Styling

- [x] **STYLE-01**: Cards with rounded corners (border-radius)
- [ ] **STYLE-02**: Cards with shadows/depth in light mode
- [ ] **STYLE-03**: Hover effects on project cards
- [x] **STYLE-04**: Consistent typography with modular scale
- [x] **STYLE-05**: Off-white page background in light mode to differentiate cards

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
| LAYOUT-01 | Phase 5 | Complete |
| LAYOUT-02 | Phase 5 | Complete |
| LAYOUT-03 | Phase 5 | Complete |
| LAYOUT-04 | Phase 4 | Complete |
| LAYOUT-05 | Phase 6 | Pending |
| THEME-01 | Phase 4 | Complete |
| THEME-02 | Phase 4 | Complete |
| THEME-03 | Phase 4 | Complete |
| THEME-04 | Phase 4 | Complete |
| THEME-05 | Phase 4 | Complete |
| STYLE-01 | Phase 5 | Complete |
| STYLE-02 | Phase 6 | Pending |
| STYLE-03 | Phase 6 | Pending |
| STYLE-04 | Phase 5 | Complete |
| STYLE-05 | Phase 4 | Complete |

**Coverage:**
- v1.1 requirements: 15 total
- Mapped to phases: 15
- Unmapped: 0

---
*Requirements defined: 2026-02-19*
*Last updated: 2026-02-19 after roadmap creation*
