# Phase 6: Visual Polish and Extras - Research

**Researched:** 2026-02-19
**Domain:** CSS shadows/depth, hover transitions, prefers-reduced-motion, bento grid extra sections
**Confidence:** HIGH

## Summary

Phase 6 adds three capabilities to the existing bento grid: (1) card shadow/depth in light mode, (2) hover effects with accessibility-safe reduced-motion handling, and (3) at least one additional bento section beyond the three project cards (e.g., a GitHub link cell or a tech stack cell).

The current CSS is 320 lines (well under the 500-line plain-CSS ceiling). Cards already have `transition: border-color 0.15s ease` and a hover rule that changes `border-color` to `--color-link`. The implementation extends this with `box-shadow` for depth and `transform: translateY()` for hover elevation, wrapped in a `@media (prefers-reduced-motion: no-preference)` guard for the motion parts. In dark mode, `box-shadow` is ineffective on dark backgrounds, so depth is communicated through the existing lighter card background (`#1e293b` vs `#0f172a`) and border contrast -- no shadow needed in dark mode.

For the additional bento section, the simplest option is a single "GitHub profile" link cell added below the project cards grid, pointing to `https://github.com/toto-castaldi`. This requires a Hugo template change in `layouts/index.html`, new i18n strings in `it.toml`/`en.toml`, and minimal CSS for the new section. The zero-JS constraint is preserved -- all effects are pure CSS.

**Primary recommendation:** Use layered `box-shadow` on `.project-card` in light mode only (via the existing token system), add `transform: translateY(-2px)` on hover inside a `prefers-reduced-motion: no-preference` media query, and add a single GitHub profile bento cell below the project cards.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| LAYOUT-05 | Additional bento sections beyond project cards (tech stack, GitHub link, or similar) | Add a GitHub profile link section below the project cards grid. Uses Hugo template change + i18n strings + CSS styling. See "Additional Bento Section" architecture pattern. |
| STYLE-02 | Cards with shadows/depth in light mode | Layered `box-shadow` on `.project-card` in light mode. Token `--shadow-card` for base, `--shadow-card-hover` for elevated state. Dark mode: no shadow (depth via background contrast). See "Card Shadow/Depth" pattern. |
| STYLE-03 | Hover effects on project cards | `transform: translateY(-2px)` + elevated shadow on `:hover`, guarded by `@media (prefers-reduced-motion: no-preference)`. Border-color change already exists. See "Hover Effects" pattern. |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| CSS `box-shadow` | CSS3 (Baseline: July 2015) | Card depth/elevation | Native, 99%+ support, GPU-accelerated, respects `border-radius` |
| CSS `transform` | CSS3 (Baseline: widely available) | Hover elevation (`translateY`) | Composited property -- no repaints, smooth 60fps animation |
| CSS `prefers-reduced-motion` | Media Queries L5 (Baseline: Jan 2020) | Accessibility guard for motion | 97%+ support, WCAG 2.3.3 compliance |
| CSS `transition` | CSS3 (Baseline: widely available) | Smooth state changes | Already in use on `.project-card` for `border-color` |
| Hugo `T` function | Hugo i18n | Bilingual strings for new section | Already in use throughout the site |

### Supporting
No additional libraries needed. Plain CSS + Hugo templates handle everything.

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `box-shadow` for depth | `filter: drop-shadow()` | `drop-shadow` follows alpha channel shape but cannot be layered or spread; `box-shadow` is standard for card depth |
| Direct `box-shadow` animation | Pseudo-element opacity trick | Better performance for continuous animations, but overkill for a simple hover transition that happens once per user action |
| `transform: translateY` for hover | `margin-top` change | `margin-top` triggers layout reflow; `transform` is composited and does not trigger reflow |
| Light-mode-only shadow | Shadow in both themes | Shadows on dark backgrounds look unnatural or invisible. Better to skip shadow in dark mode. |
| GitHub link section | Tech stack section | Tech stack requires more content/design work; GitHub link is minimal and aligns with "keep it minimal" project philosophy |

**Installation:** None needed -- plain CSS + Hugo templates.

## Architecture Patterns

### Current Card State (Phase 5 Output)
```css
/* assets/css/main.css -- current */
.project-card {
  position: relative;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-card);
  padding: var(--space-lg);
  background: var(--color-card-bg);
  transition: border-color 0.15s ease;
}

.project-card:hover {
  border-color: var(--color-link);
}
```

### Pattern 1: Card Shadow/Depth (STYLE-02)
**What:** Add layered `box-shadow` to project cards in light mode for visual depth. Use CSS custom property tokens for consistency and theme-awareness.
**When to use:** Cards on light backgrounds where depth helps distinguish content areas.

```css
/* Source: Josh Comeau's shadow design principles + MDN box-shadow */
:root {
  /* Layered shadow: subtle ambient + directional */
  --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.08),
                 0 4px 12px rgba(0, 0, 0, 0.04);
  --shadow-card-hover: 0 2px 6px rgba(0, 0, 0, 0.1),
                       0 8px 24px rgba(0, 0, 0, 0.08);
}

/* Dark mode: no shadow (depth from background contrast) */
@media (prefers-color-scheme: dark) {
  :root {
    --shadow-card: none;
    --shadow-card-hover: none;
  }
}

/* Checkbox toggle: dark override -- no shadow */
#dark-toggle:checked + .page-wrapper {
  --shadow-card: none;
  --shadow-card-hover: none;
}

/* Checkbox toggle: dark OS light override -- restore shadow */
@media (prefers-color-scheme: dark) {
  #dark-toggle:checked + .page-wrapper {
    --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.08),
                   0 4px 12px rgba(0, 0, 0, 0.04);
    --shadow-card-hover: 0 2px 6px rgba(0, 0, 0, 0.1),
                         0 8px 24px rgba(0, 0, 0, 0.08);
  }
}

.project-card {
  box-shadow: var(--shadow-card);
}
```

**Why layered shadows:** Single-shadow approaches look flat. Two layers (tight ambient + diffuse directional) create realistic depth that looks modern without being heavy.

**Why no shadow in dark mode:** Shadows on dark backgrounds (`#0f172a`) are invisible or create an unnatural "glow" effect. The existing contrast between card bg (`#1e293b`) and page bg (`#0f172a`) plus the border already provides visual separation in dark mode.

**Critical: 4-state toggle compatibility.** Shadow tokens must be set in all 4 theme blocks (`:root` default, `prefers-color-scheme: dark`, `#dark-toggle:checked +`, and dark OS + checked). This mirrors the existing pattern for `--color-*` tokens.

### Pattern 2: Hover Effects (STYLE-03)
**What:** Smooth elevation change on card hover using `transform: translateY()` and shadow escalation, guarded by `prefers-reduced-motion`.
**When to use:** Interactive cards that are clickable (the project cards are full-card links via `::after` overlay).

```css
/* Source: MDN prefers-reduced-motion + Tatiana Mac no-motion-first approach */

/* Base: no motion (safe default for all users) */
.project-card {
  transition: border-color 0.15s ease;
  box-shadow: var(--shadow-card);
}

/* Motion-safe users get smooth transitions */
@media (prefers-reduced-motion: no-preference) {
  .project-card {
    transition: border-color 0.2s ease,
                box-shadow 0.2s ease,
                transform 0.2s ease;
  }

  .project-card:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-card-hover);
  }
}

/* Border-color hover works for ALL users (not motion) */
.project-card:hover {
  border-color: var(--color-link);
}
```

**Why no-motion-first:** The "no-motion-first" approach (Tatiana Mac) starts without animation and adds it only for users who have `prefers-reduced-motion: no-preference`. This is safer than the reverse: older browsers that don't support the media query get no animation rather than unwanted motion.

**Why `translateY(-2px)` not `-4px` or `-10px`:** The cards are small (roughly 300px wide on desktop). A 2px lift is perceptible without being dramatic. Larger values feel bouncy and distract from the minimal design.

**Why 0.2s:** 200ms is the sweet spot for hover feedback -- fast enough to feel responsive, slow enough to not feel jarring. The existing `border-color` transition is 0.15s; 0.2s groups all transitions at a comfortable speed.

### Pattern 3: Additional Bento Section (LAYOUT-05)
**What:** A GitHub profile link section below the project cards, styled as a bento cell.
**When to use:** When the page needs content beyond project cards to fill the bento grid concept.

```html
<!-- layouts/index.html addition -->
<section class="bento-extras" aria-label="{{ T "extras" }}">
  <a href="https://github.com/toto-castaldi" class="bento-cell bento-github" rel="noopener">
    <span class="bento-github-icon" aria-hidden="true">
      <!-- inline SVG GitHub mark or Unicode symbol -->
    </span>
    <span>{{ T "githubProfile" }}</span>
  </a>
</section>
```

```css
/* Bento extras section */
.bento-extras {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-lg);
  padding: var(--space-md) 0;
  margin-top: var(--space-lg);
}

.bento-cell {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-card);
  padding: var(--space-md) var(--space-lg);
  background: var(--color-card-bg);
  box-shadow: var(--shadow-card);
  text-decoration: none;
  color: var(--color-text);
  display: flex;
  align-items: center;
  gap: var(--space-sm);
}

@media (prefers-reduced-motion: no-preference) {
  .bento-cell {
    transition: border-color 0.2s ease,
                box-shadow 0.2s ease,
                transform 0.2s ease;
  }

  .bento-cell:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-card-hover);
  }
}

.bento-cell:hover {
  border-color: var(--color-link);
}
```

**Why GitHub link over tech stack:** The project's philosophy is "minimal, focus on projects." A GitHub profile link is one cell, one link, bilingual via `T` function -- minimal effort and content. A tech stack section would require listing technologies, adding icons, and creating more substantial content that the user hasn't requested. The requirement says "tech stack, GitHub link, or similar" -- GitHub link is the simplest valid choice.

**Why below the project cards, not beside them:** The 3-column project card grid fills the row exactly. Adding a 4th cell would break the 3-column layout or require grid-spanning logic. Placing the extras section below keeps the project cards grid untouched and adds a visually distinct section.

**i18n strings needed:**
- `en.toml`: `githubProfile = "GitHub"`, `extras = "Links"`
- `it.toml`: `githubProfile = "GitHub"`, `extras = "Link"`

### Anti-Patterns to Avoid

- **Animating `box-shadow` directly on low-end devices:** For simple hover transitions that fire once per interaction, direct `box-shadow` transition is acceptable. The pseudo-element opacity trick is overkill for hover effects (it's designed for continuous/looping animations).
- **Shadows in dark mode:** Shadows on dark backgrounds are invisible or create an ugly "glow." Use background-color elevation or border contrast instead.
- **Removing hover effects entirely for reduced-motion users:** `prefers-reduced-motion` means reduce MOTION, not remove all hover feedback. Border-color change (not motion) should still apply for all users.
- **Using `!important` to override dark mode shadows:** Follow the existing 4-state token pattern -- define `--shadow-card: none` in the correct toggle blocks.
- **Adding a heavy GitHub SVG icon:** Keep the icon minimal. An inline SVG of the GitHub mark (~400 bytes) or a simple Unicode symbol is fine. Do not add a font icon library.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Shadow tokens in dark mode | Manual `@media` checks per element | CSS custom property tokens (`--shadow-card`) set per theme block | One token, consumed everywhere; mirrors existing `--color-*` pattern |
| Motion accessibility | Custom JS motion detection | `@media (prefers-reduced-motion: no-preference)` | Native, zero-JS, 97%+ browser support |
| GitHub icon | Icon font library (FontAwesome, etc.) | Inline SVG (~400 bytes) | Zero network requests, matches the system font stack philosophy |
| Bento cell hover | Per-element hover styles | Shared `.bento-cell` class with hover rules | Reusable for future bento sections |

**Key insight:** The existing codebase already has the 4-state toggle token pattern (`:root`, dark media, checked override, dark+checked override). Shadow tokens simply follow the same pattern.

## Common Pitfalls

### Pitfall 1: Shadow Tokens Not Set in All 4 Toggle States
**What goes wrong:** Shadow appears in light mode but persists when user toggles to dark mode (or vice versa).
**Why it happens:** The checkbox-hack toggle uses 4 separate CSS blocks to set tokens. If `--shadow-card` is only defined in `:root` and not overridden to `none` in the dark-mode blocks, shadows leak into dark mode.
**How to avoid:** Define `--shadow-card` and `--shadow-card-hover` in ALL 4 existing theme blocks, mirroring the `--color-*` pattern exactly. Test all 4 states.
**Warning signs:** Cards with shadows visible on the dark slate background; shadows disappearing when toggling back to light mode.

### Pitfall 2: Transform Hover Shifts Adjacent Card Layout
**What goes wrong:** When one card elevates on hover, adjacent cards shift position, creating a "jumpy" layout.
**Why it happens:** If `margin-top` or `position: relative; top` is used instead of `transform`, it triggers layout reflow. Adjacent grid items reposition.
**How to avoid:** Use `transform: translateY(-2px)` exclusively. `transform` is composited -- it does NOT trigger layout reflow. The card visually lifts but its grid slot remains unchanged.
**Warning signs:** Other cards moving when hovering over one card.

### Pitfall 3: Card ::after Overlay Blocks Hover on Bento Cell
**What goes wrong:** The `.card-link::after { position: absolute; inset: 0 }` overlay captures pointer events, potentially interfering with bento cell hover.
**Why it happens:** The overlay makes the entire `.project-card` clickable. If a bento cell is positioned too close or overlaps, the overlay could intercept events.
**How to avoid:** The bento extras section is a separate `<section>` below the project cards. No overlap is possible. The `.card-link::after` overlay is scoped to `.project-card` via `position: relative` on the card. No issue expected.
**Warning signs:** GitHub link not clickable; hover effect on wrong element.

### Pitfall 4: Transition Shorthand Overriding Existing Border-Color Transition
**What goes wrong:** Adding `transition: transform 0.2s, box-shadow 0.2s` inside the motion media query overwrites the existing `transition: border-color 0.15s ease`.
**Why it happens:** CSS `transition` is a shorthand. Setting it again replaces the previous value entirely.
**How to avoid:** Include ALL transitioned properties in one declaration: `transition: border-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease`. The base rule (outside media query) keeps only `transition: border-color 0.15s ease` for reduced-motion users. The full declaration lives inside `@media (prefers-reduced-motion: no-preference)`.
**Warning signs:** Border-color no longer transitions smoothly on hover.

### Pitfall 5: Muted Text Contrast Worsened by Shadow
**What goes wrong:** The muted text (`#6b7280`) on light background (`#f9fafb`) currently passes WCAG AA with only 0.13 headroom (4.63:1). Adding shadow could visually darken the card background perception.
**Why it happens:** Shadows don't change actual contrast ratios, but they can alter perceived contrast. This is minor.
**How to avoid:** The shadow is OUTSIDE the card (not inset). The card background remains `#ffffff`. Muted text contrast on white is 4.83:1 -- not affected by external shadows. No change needed.
**Warning signs:** Muted text appearing less legible on cards with shadows.

## Code Examples

Verified patterns for the implementation:

### Complete Shadow Token System
```css
/* Source: Existing 4-state toggle pattern + Josh Comeau shadow design */

/* Light mode (default) */
:root {
  --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.08),
                 0 4px 12px rgba(0, 0, 0, 0.04);
  --shadow-card-hover: 0 2px 6px rgba(0, 0, 0, 0.1),
                       0 8px 24px rgba(0, 0, 0, 0.08);
}

/* Dark mode (OS preference) */
@media (prefers-color-scheme: dark) {
  :root {
    --shadow-card: none;
    --shadow-card-hover: none;
  }

  .page-wrapper {
    --shadow-card: none;
    --shadow-card-hover: none;
  }
}

/* Light OS -> Dark override (checkbox checked) */
#dark-toggle:checked + .page-wrapper {
  --shadow-card: none;
  --shadow-card-hover: none;
}

/* Dark OS -> Light override (checkbox checked) */
@media (prefers-color-scheme: dark) {
  #dark-toggle:checked + .page-wrapper {
    --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.08),
                   0 4px 12px rgba(0, 0, 0, 0.04);
    --shadow-card-hover: 0 2px 6px rgba(0, 0, 0, 0.1),
                         0 8px 24px rgba(0, 0, 0, 0.08);
  }
}
```

### Complete Hover Effect with Motion Guard
```css
/* Source: MDN prefers-reduced-motion + Tatiana Mac no-motion-first */

/* Base: border-color hover for ALL users (not motion) */
.project-card {
  box-shadow: var(--shadow-card);
  transition: border-color 0.15s ease;
}

.project-card:hover {
  border-color: var(--color-link);
}

/* Enhanced: motion effects only for users who accept motion */
@media (prefers-reduced-motion: no-preference) {
  .project-card {
    transition: border-color 0.2s ease,
                box-shadow 0.2s ease,
                transform 0.2s ease;
  }

  .project-card:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-card-hover);
  }
}
```

### GitHub Profile Bento Cell (Hugo Template)
```html
<!-- Source: Hugo T function pattern from existing codebase -->
<section class="bento-extras" aria-label="{{ T "extras" }}">
  <a href="https://github.com/toto-castaldi" class="bento-cell bento-github" rel="noopener">
    <svg class="bento-icon" width="24" height="24" viewBox="0 0 16 16" fill="currentColor" aria-hidden="true">
      <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27s1.36.09 2 .27c1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.01 8.01 0 0 0 16 8c0-4.42-3.58-8-8-8z"/>
    </svg>
    <span>{{ T "githubProfile" }}</span>
  </a>
</section>
```

### i18n Strings
```toml
# en.toml additions
[githubProfile]
other = "GitHub"

[extras]
other = "Links"
```

```toml
# it.toml additions
[githubProfile]
other = "GitHub"

[extras]
other = "Link"
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single heavy `box-shadow` | Layered subtle shadows (ambient + directional) | ~2022-2023 | More realistic depth, modern look |
| Global animations + `@media (reduce)` to disable | No-motion-first: no animation by default, add inside `@media (no-preference)` | ~2021 (Tatiana Mac) | Better accessibility default, safer for old browsers |
| Shadow in both light and dark mode | Light-mode-only shadow, dark uses bg-color elevation | ~2023-2024 | Avoids ugly glow on dark backgrounds |
| `box-shadow` animation via pseudo-element opacity trick | Direct `box-shadow` transition for hover (acceptable perf) | Ongoing | Pseudo-element trick still best for continuous animations; direct transition fine for hover |

**Deprecated/outdated:**
- Heavy single-layer shadows (e.g., `0 10px 30px rgba(0,0,0,0.3)`): replaced by layered subtle shadows
- Assuming all users want animation: replaced by `prefers-reduced-motion` requirement (WCAG 2.3.3)
- Identical shadows in light and dark mode: replaced by theme-aware shadow tokens

## Open Questions

1. **GitHub icon approach: inline SVG vs Unicode symbol**
   - What we know: The GitHub Octicon mark is a well-known SVG (~400 bytes inline). Unicode has no official GitHub symbol.
   - What's unclear: Whether the user prefers a minimal Unicode approach (e.g., just text "GitHub") or the recognizable mark icon.
   - Recommendation: Use inline SVG of the GitHub mark. It's standard, recognizable, tiny, and works in all browsers. The `fill="currentColor"` technique makes it theme-aware automatically.

2. **Whether to add more bento cells beyond GitHub**
   - What we know: The requirement says "at least one additional bento section." GitHub link satisfies this minimum.
   - What's unclear: Whether the user wants tech stack, contact, or other cells in this phase.
   - Recommendation: Start with one GitHub cell. Additional cells can be added later. The `.bento-extras` grid and `.bento-cell` class are reusable for future sections.

3. **Shadow intensity preference**
   - What we know: The recommended values (`0 1px 3px rgba(0,0,0,0.08), 0 4px 12px rgba(0,0,0,0.04)`) are on the subtle end. More dramatic shadows are possible.
   - What's unclear: The user's preference for shadow intensity.
   - Recommendation: Start subtle. The token system makes it trivial to adjust values later.

## Sources

### Primary (HIGH confidence)
- MDN: `box-shadow` property - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/box-shadow - Syntax, browser support (Baseline: July 2015), interpolation behavior
- MDN: `prefers-reduced-motion` - https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion - Values (`reduce`, `no-preference`), browser support (Baseline: Jan 2020)
- Current codebase: `assets/css/main.css` (320 lines) - existing 4-state toggle pattern, card styles, transition behavior

### Secondary (MEDIUM confidence)
- Josh Comeau: "Designing Beautiful Shadows in CSS" - https://www.joshwcomeau.com/css/designing-shadows/ - Layered shadow approach, hue-matched colors, elevation levels
- Tatiana Mac: "prefers-reduced-motion: no-motion-first approach" - https://www.tatianamac.com/posts/prefers-reduced-motion - No-motion-first pattern, progressive enhancement
- Tobias Ahlin: "How to animate box-shadow with silky smooth performance" - https://tobiasahlin.com/blog/how-to-animate-box-shadow/ - Pseudo-element opacity trick (documented but not recommended for this use case)
- W3C WCAG Technique C39 - https://www.w3.org/WAI/WCAG21/Techniques/css/C39 - prefers-reduced-motion for WCAG 2.3.3 compliance

### Tertiary (LOW confidence)
- None -- all findings verified with primary or secondary sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- `box-shadow`, `transform`, `transition`, `prefers-reduced-motion` are all mature CSS features with 97%+ browser support
- Architecture: HIGH -- follows the exact 4-state toggle token pattern already in the codebase; hover effect extends existing `transition` rule
- Pitfalls: HIGH -- the main risks (4-state toggle consistency, transition shorthand override, transform vs margin reflow) are well-documented CSS behaviors
- Bento section: MEDIUM -- the GitHub link cell is a design choice (vs tech stack, etc.); the template/CSS pattern is straightforward but content is a judgment call

**Research date:** 2026-02-19
**Valid until:** 2026-08-19 (stable domain -- CSS features are mature, shadow/motion patterns are established)
