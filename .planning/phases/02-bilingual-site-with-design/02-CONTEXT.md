# Phase 2: Bilingual Site with Design - Context

**Gathered:** 2026-02-18
**Status:** Ready for planning

<domain>
## Phase Boundary

Visitors see three project cards (Docora, Lumio, Helix) with names, descriptions, and links in their chosen language, on a clean responsive page with language switching, dark mode, and a favicon. No new capabilities beyond what ROADMAP.md defines.

</domain>

<decisions>
## Implementation Decisions

### Project card design
- Each card shows: official project logo (from each product's site), project name, bilingual description, link to subdomain
- Card visual containment style: Claude's discretion (bordered, elevated, or borderless)
- Link interaction (whole card clickable vs explicit button): Claude's discretion

### Page layout & color palette
- Short tagline above the project cards (e.g., "Software projects by Toto Castaldi" — bilingual)
- Tagline feels like a quiet subtitle — smaller, lighter weight, understated; cards are the focus
- No footer — page ends after the project cards
- Overall page structure (centered column vs grid): Claude's discretion
- Color palette: Claude's discretion

### Typography
- Font choice (system vs web font): Claude's discretion
- Airy and spacious density — generous line-height, large margins, content breathes
- Project names on cards use the same font as body text, just bold and larger — hierarchy through size and weight, not a different typeface
- Modular typography scale with CSS custom properties (per DSGN-03 requirement)

### Language switcher
- Positioned in top-right corner of the page
- Visual style (text links, flags, full names): Claude's discretion
- Whether to show both languages or only the alternate: Claude's discretion

### Favicon
- Design and style: Claude's discretion (SVG with dark mode support per CONT-06)

### Claude's Discretion
- Card visual containment style (bordered, elevated, borderless)
- Card link interaction pattern (whole card clickable vs explicit link)
- Page structure (centered column vs card grid)
- Color palette for light and dark modes
- Font choice (system stack vs specific web font)
- Language switcher appearance and behavior details
- Favicon design
- Dark mode color strategy (CSS prefers-color-scheme, no JS)

</decisions>

<specifics>
## Specific Ideas

- Official logos from each product's own site (docora.toto-castaldi.com, lumio.toto-castaldi.com, helix.toto-castaldi.com) to be used as project icons on cards
- Tagline should be quiet and understated — the project cards are the main content
- No footer at all — ultra minimal page

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-bilingual-site-with-design*
*Context gathered: 2026-02-18*
