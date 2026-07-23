# House of Manna — Website Design System & Conventions

This doc exists so Claude Code can make consistent edits to this site without
re-deriving the design system from scratch every session. Read this file
first, then look at `about.html` as the reference implementation before
editing or creating any page.

## Source of truth for content

`Master_Prospectus.pdf` (Founder Chad Parker) is the authoritative source for
all organizational content — mission, programs, numbers, quotes. When adding
or editing copy, pull from the prospectus rather than inventing new claims.
Chapter map:
- Ch. 1 — Executive Vision → used on `about.html`
- Ch. 2 — Ecosystem Overview → used on `about.html`
- Ch. 3 — Academy → used on `academy.html`
- Ch. 4 — Pantry / Mobile → Pantry content still needs its own page
  (`pantry.html`); Mobile content is on `mobile.html`
- Letter From the Founder → used on `about.html`
- Five Organizational Pillars → used on `about.html`

## Site structure

Single-page homepage (`House-of-Manna-brevo-no-freeze` file) plus standalone
pages: `about.html`, `academy.html`, `mobile.html` (built),
`pantry.html`, `get-involved.html` (not yet built). Every page shares the
exact same `<nav>` and `<footer>` markup, copied verbatim from the homepage
(includes the brand logo as an inline base64 `<img>`, so don't try to
"clean up" or re-encode it — copy it as-is between files).

## Brand tokens (CSS custom properties, defined once in `:root`)

```css
--hom-navy:   #2D3B4F   /* primary navy, mid-tone */
--hom-navy-2: #172334   /* darker navy, used for most dark sections + footer */
--hom-gold:   #C8941F
--hom-sage:   #8E9C85   /* secondary accent, used for "dark-on-color" sections */
--hom-cream:  #F7F4EC   /* primary light background */
--hom-tan:    #D9C8A0   /* small accents, eyebrow text on dark backgrounds */
--font-display: 'Anton', ...   /* headlines — ALWAYS uppercase, never body copy */
--font-body:    'Inter', ...   /* everything else */
```

Rules that keep breaking if not followed explicitly:
- Anything using `--font-display` (Anton) needs an explicit
  `text-transform:uppercase` if it isn't a real `<h1>/<h2>/<h3>` tag (those
  get it for free from a global rule). Anton is a condensed caps-only face —
  mixed case looks broken.
- Every content section needs a `data-theme="dark"` or `data-theme="light"`
  attribute on the `<section>` itself. A scroll listener reads these to flip
  the nav text color as sections pass under it. Navy and sage backgrounds =
  `dark`; cream and white = `light`.

## Reusable components (copy the exact class names/markup, don't reinvent)

**Hero (video montage)** — every interior page hero is 5 short muted/looping
video clips stacked and cross-faded (not looped individually; each plays to
its natural end via the `ended` event, then the next fades in). Overlay is a
5-stop `linear-gradient` from ~50% navy at the top to **`rgba(23,35,52,1)`**
(fully opaque, exactly `--hom-navy-2`) at the very bottom — this must match
the page-break SVG fill below it exactly or you'll get a visible seam.
Classes: `.{page}-hero-video-wrap`, `.{page}-hero-overlay`. JS:
`initMontage(containerId)`, called once per hero container id.

**Page-break dividers** — torn-paper SVG transition between sections,
`<div class="page-break-bottom">...</div>` sitting between two `<section>`s
with negative margins pulling it over the seam. Fill color set via
`.pagebreak-navy` / `.pagebreak-sage` / `.pagebreak-tan` / `.pagebreak-white`
class on the polygon. **Every instance needs a unique `id` suffix** on the
`<svg>` and `<clipPath>` (e.g. `Layer_2_bottom_academyhero`,
`clippath-bottom-academyhero`) — duplicate IDs on one page will break the
clip-path silently.

**Accordion** — class `.phase-item` / `.phase-toggle` / `.phase-panel`
(originally the homepage's "Six Phases of Growth"). Reused for: Five
Pillars (about), Scholarship Requirements + Career Pathways (academy),
Distribution Model (mobile). JS: `initAccordion(listId)`, one call per
`<div class="phases-list" id="...">` on the page. Color variant is done by
scoping overrides under the list's own `#id` (see `#pillarsList`,
`#reqsList`, `#modelList`, `#pathwaysList` in the CSS) — don't edit the base
`.phase-item` rules, add a new scoped block instead.

**Flip cards** (about.html Executive Vision) — `.vision-flip-card` /
`.vision-flip-card-inner` (3D rotateY). One card variant has class
`.vision-flip-static` and starts with `.is-flipped` already in the markup
and is excluded from the JS click-toggle (`:not(.vision-flip-static)`) —
that's intentional per a past client request, not a bug.

**Interactive tab-switcher** (mobile.html Target Neighborhoods) — a newer
pattern, not on other pages yet. `.neighborhood-tabs` / `.neighborhood-tab`
/ `.neighborhood-panel`, plain click-to-swap JS, no libraries.

**Green swirl decoration** — a PNG cutout reused from the homepage's Pillars
section, base64-inlined. Currently on: about.html (Pillars, right;
Letter section, left/flipped/rotated/bleeding off the edge). Reuse by
copying the exact `<img class="{section}-swirl" src="data:image/png;base64,...">`
tag between files — the data URL is identical everywhere, just the wrapping
class/position changes.

**Dark-break pull quote** — `.dark-break` / `.outline-heading` /
`.quote-attribution`. Centered, large outlined-text quote on a navy-2
background. Used once per page for a signature quote from the prospectus.

**Stat band, tag chips, icon grids, CTA strip** — see `.stat-band`,
`.career-tags` / `.beyond-tags` / `.criteria-strip`, `.ecosystem-grid` /
`.barriers-grid` / `.career-framework`, `.cta-strip` in the relevant page's
`<style>` block. All follow the same visual language (pill chips, simple
2px-stroke line icons drawn by hand in the site's style — not an icon
library).

## Video sourcing convention

All hero videos are hotlinked directly from Pexels (`videos.pexels.com`,
`images.pexels.com` for posters) — same approach the homepage already uses
for its Unsplash hero photo. No video files are hosted locally. When sourcing
new clips:
- Confirm subject matter via the Pexels page's `og:image` filename and the
  `meta-article:tag` list before using a clip — this is the only way to
  verify content without being able to preview frames directly.
- Prefer clips from the same photographer/shoot when you need several similar
  clips (consistent look, easier to confirm they show the same
  people/setting).
- Landscape-oriented source files (`.../uhd_2560_1440` or `hd_1920_1080`)
  work much better than portrait ones for a full-bleed hero — check the
  filename before committing to a clip.

## Local assets

Only one local image exists: `images/cp-headshot.jpg` (founder photo, used
on about.html). Any new local images/video should go in `images/` next to
each HTML file, referenced with a relative path — don't inline new large
assets as base64 unless matching an existing reused asset (like the swirl).

## What's built vs. outstanding

Built: homepage, `about.html`, `academy.html`, `mobile.html`.
Outstanding: `pantry.html` (Ch. 4 Pantry content is written in the
prospectus but not yet turned into a page), `get-involved.html`.
