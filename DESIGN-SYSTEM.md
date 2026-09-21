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
- Ch. 4 — Pantry / Mobile → Pantry content is on `pantry.html`; Mobile
  content is on `mobile.html`; the cross-program "how it all connects"
  narrative is on `programs.html`

## Site structure

Single-page homepage (`index.html`) plus standalone pages: `about.html`,
`academy.html`, `pantry.html`, `mobile.html`, `programs.html`, `projects.html`,
`project-facility.html`, `project-innovation-lab.html`,
`project-food-access.html` (all built), `get-involved.html` (not yet built —
footer "Get Involved" sub-links and CTA-strip secondary buttons point at
`/#get-involved` instead until it exists). Every page shares the exact same
`<nav>` and `<footer>` markup, copied verbatim from the homepage (includes
the brand logo as an inline base64 `<img>`, so don't try to "clean up" or
re-encode it — copy it as-is between files).

**Link convention — no `.html` in hrefs.** GitHub Pages serves this repo's
`.html` files at their extensionless path too (confirmed: `/about` serves
`about.html`), so every internal `href` sitewide is written without the
`.html` suffix (`href="about"`, not `href="about.html"`) and the homepage is
always linked as root — `href="/"` or `href="/#impact"`, never `href="index"`
or `href="index.html"`. The physical files themselves keep their `.html`
names on disk (don't rename them) — only the *links pointing at them* drop
the extension. `<link rel="canonical">` and `og:url`/`twitter:url` follow the
same rule (`https://houseofmannamd.com/about`, not `/about.html`). When
adding a new page, write every internal link to it (nav, footer, cross-nav,
CTAs) without `.html` from the start.

**Primary nav (all pages, verbatim):** About → `about` · Programs →
`programs` · Projects → `projects` · Impact → `/#impact` · Get Involved →
`/#get-involved`. `programs.html` is the flagship overview/gateway into the
three ongoing programs; `projects.html` is the equivalent overview for the
funding/capital projects. Individual program pages (`academy.html`,
`pantry.html`, `mobile.html`) and individual project pages
(`project-facility.html`, `project-innovation-lab.html`,
`project-food-access.html`) are intentionally *not* in the primary nav —
they're reachable from their respective overview page, the footer, and
their own "Explore The Ecosystem" / "Explore The Other Projects" cross-nav
(see below).

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

**Explore The Ecosystem** — cross-nav component (`.ecosystem-links` /
`.ecosystem-link-card` / `.is-current`) reused verbatim on every individual
program page (`academy.html`, `pantry.html`, `mobile.html`), placed right
before the closing `.cta-strip`. Three cards link to the other two programs;
the current page's own card gets `.is-current` (a non-link `<div>`, sage
tint, "You Are Here" tag) instead of an `<a>`. Not used on `programs.html`
itself since that page already features all three programs directly.

**Program feature / flow / connect components** (`programs.html` only) —
`.program-feature` (`.on-light`/`.on-dark`, `.is-reverse`) is the editorial
media+copy layout used for the Pantry/Academy/Mobile features; `.flow-path`
/`.flow-stage` is the Today→Tomorrow→Beyond ecosystem diagram; `.connect-*`
classes build the "How It All Connects" diagram in semantic HTML/CSS (no
text-in-image); `.find-place-grid`/`.find-place-card` is the closing
5-path CTA grid. All page-specific to `programs.html`; not shared elsewhere
yet, but written generically enough to lift into another page if needed.

**Dark-break pull quote** — `.dark-break` / `.outline-heading` /
`.quote-attribution`. Centered, large outlined-text quote on a navy-2
background. Used once per page for a signature quote from the prospectus.

**Stat band, tag chips, icon grids, CTA strip** — see `.stat-band`,
`.career-tags` / `.beyond-tags` / `.criteria-strip`, `.ecosystem-grid` /
`.barriers-grid` / `.career-framework`, `.cta-strip` in the relevant page's
`<style>` block. All follow the same visual language (pill chips, simple
2px-stroke line icons drawn by hand in the site's style — not an icon
library).

## Donate CTA routing — not all "Donate Now" buttons go to the same place

`.donate-btn` is one visual style, but it points at different destinations
depending on *where* it sits, and this is intentional — don't "fix" it back
to one URL:

- **`.nav-donate`** (the nav's top-right "Donate Now", verbatim on every
  page) → `href="projects"`. It's a router: send people to the projects hub
  so they can pick what to fund, not straight to a single campaign.
- **Hero / CTA-strip / give-section `.donate-btn` on `project-facility.html`,
  `project-innovation-lab.html`, `project-food-access.html`** → that
  project's own Givebutter campaign, `target="_blank"`:
  - Facility: `https://givebutter.com/house-of-manna-pantry`
  - Innovation Lab: `https://givebutter.com/youth-business-ai-community-innovation-lab-omm61p`
  - Food Access: `https://givebutter.com/baltimore-neighborhood-food-access-family-wellness-4xbodr`
- **Every other `.donate-btn`** (about/academy/pantry/mobile/programs/index
  hero + cta-strip, and `projects.html`'s own general give-section) → stays
  on the general fund, `https://givebutter.com/house-of-manna-pantry`,
  `target="_blank"`.

When adding a 4th project page, give it its own Givebutter campaign link on
every `.donate-btn` except `.nav-donate` (leave that pointing at `projects`).

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

Built: homepage, `about.html`, `academy.html`, `mobile.html`, `pantry.html`,
`programs.html`, `projects.html`, `project-facility.html`,
`project-innovation-lab.html`, `project-food-access.html`.
Outstanding: `get-involved.html` — until it exists, every "Get Involved"
sub-link (Donate/Volunteer/Partner/Sponsorship in the footer, and the
`.cta-strip`'s secondary button) points at `/#get-involved` (the homepage
footer) rather than a dead link.
