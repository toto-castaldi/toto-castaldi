---
phase: quick-260719-flq
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - static/images/lumio-logo.svg
  - data/projects.toml
autonomous: true
requirements: [QUICK-FIX-LUMIO-LOGO]
must_haves:
  truths:
    - "Lumio card on the landing page shows the Lumio logo (no broken image)"
    - "Logo is served from this site, not from the 404-ing lumio.toto-castaldi.com URL"
  artifacts:
    - path: "static/images/lumio-logo.svg"
      provides: "Self-hosted Lumio logo (tri-color pie + 3 rays)"
      contains: "viewBox=\"0 0 400 300\""
    - path: "data/projects.toml"
      provides: "Lumio logo reference pointing to local asset"
      contains: "/images/lumio-logo.svg"
  key_links:
    - from: "data/projects.toml"
      to: "static/images/lumio-logo.svg"
      via: "logo field consumed by layouts/index.html img src"
      pattern: "logo = \"/images/lumio-logo.svg\""
---

<objective>
Fix the broken Lumio logo on the landing page. `data/projects.toml` currently points to `https://lumio.toto-castaldi.com/logo.svg`, which returns 404 (the Lumio site has no hosted logo file — its logo is inline SVG). Self-host the logo in this Hugo repo and reference it locally.

Purpose: Lumio project card renders its logo instead of a broken image.
Output: `static/images/lumio-logo.svg` + updated `data/projects.toml`.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@data/projects.toml
@layouts/index.html

Diagnosis (already verified by orchestrator — do NOT re-check remote URLs):
- `https://lumio.toto-castaldi.com/logo.svg` and all likely variants return 404.
- `layouts/index.html` renders `<img src="{{ .logo }}" ... width="48" height="48">` — a root-relative path like `/images/lumio-logo.svg` flows through unchanged.
- `static/images/` does not exist yet; Hugo copies `static/` contents to site root, so `static/images/lumio-logo.svg` will be served at `/images/lumio-logo.svg`.

Canonical Lumio logo markup (extracted from the Lumio homepage inline SVG). Task 1 must write this file content VERBATIM:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 300">
  <!-- Lumio logo: tri-color pie + 3 rays -->
  <path d="M 200 200 L 200 100 A 100 100 0 0 1 286.6 250 Z" fill="#FFA726"/>
  <path d="M 200 200 L 286.6 250 A 100 100 0 0 1 113.4 250 Z" fill="#FF7061"/>
  <path d="M 200 200 L 113.4 250 A 100 100 0 0 1 200 100 Z" fill="#9C68D4"/>
  <rect x="115" y="-12" width="85" height="24" rx="6" fill="#FFA726" transform="translate(200, 200) rotate(-90)"/>
  <rect x="115" y="-12" width="85" height="24" rx="6" fill="#FF7061" transform="translate(200, 200) rotate(30)"/>
  <rect x="115" y="-12" width="85" height="24" rx="6" fill="#9C68D4" transform="translate(200, 200) rotate(150)"/>
</svg>
```
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create self-hosted Lumio logo SVG</name>
  <files>static/images/lumio-logo.svg</files>
  <action>
    Create `static/images/lumio-logo.svg` (creating the `static/images/` directory) with exactly the SVG markup given verbatim in the context section above ("Canonical Lumio logo markup"). Use the Write tool (no heredocs). Do not add width/height attributes to the SVG root — the img tag in layouts/index.html already sets 48x48, and the viewBox handles scaling. Do not reformat, minify, or alter colors/coordinates.
  </action>
  <verify>
    <automated>test -f static/images/lumio-logo.svg &amp;&amp; grep -c 'viewBox="0 0 400 300"' static/images/lumio-logo.svg</automated>
  </verify>
  <done>File exists at static/images/lumio-logo.svg with the tri-color pie + rays markup (3 path elements + 3 rect elements, fills #FFA726 / #FF7061 / #9C68D4).</done>
</task>

<task type="auto">
  <name>Task 2: Point projects.toml at local logo and verify build</name>
  <files>data/projects.toml</files>
  <action>
    In `data/projects.toml`, in the Lumio `[[project]]` block, change the logo value from `https://lumio.toto-castaldi.com/logo.svg` to `/images/lumio-logo.svg`. Leave the Docora entry untouched (its remote logo works).

    Then run `hugo --minify` and confirm the build succeeds and the logo file is copied to the output.
  </action>
  <verify>
    <automated>grep -c 'logo = "/images/lumio-logo.svg"' data/projects.toml &amp;&amp; hugo --minify &amp;&amp; test -f public/images/lumio-logo.svg &amp;&amp; grep -c '/images/lumio-logo.svg' public/index.html</automated>
  </verify>
  <done>projects.toml references /images/lumio-logo.svg; `hugo --minify` succeeds; public/images/lumio-logo.svg exists; generated public/index.html contains an img src of /images/lumio-logo.svg; no remaining reference to lumio.toto-castaldi.com/logo.svg in data/.</done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| None new | Static SVG asset served from own origin; removes a cross-origin image dependency |

## STRIDE Threat Register

| Threat ID | Category | Component | Disposition | Mitigation Plan |
|-----------|----------|-----------|-------------|-----------------|
| T-flq-01 | Tampering | static/images/lumio-logo.svg | accept | Static asset in own repo; contains no scripts/foreignObject and is rendered via img tag (no script execution context) |
</threat_model>

<verification>
- `hugo --minify` exits 0
- `public/images/lumio-logo.svg` exists in build output
- `public/index.html` and `public/en/index.html` reference `/images/lumio-logo.svg`
- No occurrence of `lumio.toto-castaldi.com/logo.svg` remains in `data/projects.toml`
</verification>

<success_criteria>
- Lumio card renders a valid logo from a self-hosted asset in both IT and EN pages
- Build passes with no warnings related to the change
</success_criteria>

<output>
Create `.planning/quick/260719-flq-fix-logo-lumio-mancante-in-landing-page-/260719-flq-SUMMARY.md` when done
</output>
