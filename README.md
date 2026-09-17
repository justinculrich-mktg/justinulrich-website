# Handoff: justinulrich.com (Cloudflare Pages)

## Overview

This is a personal portfolio site for Justin Ulrich, a marketing leader in Denver. It was
rebuilt from a Wix site so it can be self-hosted and the Wix subscription canceled. It is
**already live** on Cloudflare Pages, deployed from a GitHub repo.

This package is **not** a design-to-code handoff. The HTML in this repo is the production
site. Your job is maintenance and changes, not reimplementation. Do not port it to a
framework, do not introduce a build step, do not "modernize" it unless asked.

## Where things live

| | |
| --- | --- |
| Repo | `justinculrich-mktg/justinulrich-website` (private), branch `main` |
| Host | Cloudflare Pages project `justinulrich-website` |
| Preview URL | `https://justinulrich-website.pages.dev` |
| Production domain | `justinulrich.com` (DNS migration in progress at time of writing) |
| Large media | Cloudflare R2, public bucket base `https://pub-bc8f6a2cf6154e4d90616851a25b359f.r2.dev/` |

Deploys are automatic: push to `main` and Pages rebuilds. There is no build command and no
output directory transform — the repo root **is** the site root.

## How the site is built

Six pages, no framework, no bundler, no npm.

```
index.html          Home
about.html          About Me
work.html           Work
volunteer.html      Volunteer
references.html     References
contact.html        Contact Me
Nav.dc.html         Shared header  — imported by every page
Footer.dc.html      Shared footer  — imported by every page
support.js          Rendering runtime (do not edit)
assets/             Images, PDFs, favicon, one local video
assets/pdfs/        30 self-hosted PDFs (collateral, case studies, resume)
_headers            Cache + security headers
robots.txt
sitemap.xml
```

Each page is a self-contained HTML document. Inside it:

- A `<head>` with static `<title>`, meta description, canonical, and Open Graph tags.
- A `<script src="./support.js">` that boots the renderer.
- An `<x-dc>` element containing the page template.
- A `<script type="text/x-dc" data-dc-script>` containing a `class Component` with the page's
  logic (carousel state, modal state, video controls).

**Templating.** `{{ name }}` holes in the template are resolved from the values returned by
the logic class's `renderVals()`. Holes are dotted lookups only — no expressions. Control
flow uses `<sc-if value="{{ flag }}">` and `<sc-for list="{{ items }}" as="item">`.
`<dc-import name="Nav">` mounts `Nav.dc.html`.

**Styling is 100% inline `style=""` attributes.** There is no stylesheet and no CSS classes.
This is deliberate. The only global CSS is a small `<helmet><style>` block per page holding
font `@font-face`/`<link>` tags, body resets, and `a` / `a:hover` colors. Keep it that way —
adding a stylesheet will fight the rest of the file.

Pseudo-states are expressed as sibling attributes: `style-hover`, `style-active`,
`style-focus`.

**Rendering is client-side.** The body HTML is assembled in the browser, so crawlers see only
the `<head>`. Per-page title/description/OG tags are static in the `<head>` so search snippets
and link previews work. If SEO on body copy ever matters, that's the thing to change.

## Typography and color

| Token | Value |
| --- | --- |
| Headings | Oswald, weight 500 (300 for large light headings) |
| Body | Figtree, weight 400 |
| Navy (nav, hero panels) | `#1B2A41` |
| Slate blue (contact panel) | `#3F5B7A` |
| Orange accent (links, progress) | `#CE7213`, hover `#E08A2E` |
| Body text | `#4A5561` |
| Heading text | `#2E3A47` |
| Muted / secondary | `#8FA1B5`, `#9FB0C4` |
| Charcoal (logo ground) | `#4A4A4A` |
| Video letterbox / player ground | `#000000`, screen layer `#0B0E12` |

Nav font size is 23px. Buttons are fully rounded. Page content is centered at
`max-width: 1000px` (some sections 1080px) with `24px` side padding.

## Page-specific behavior

### Home (`index.html`)
Hero animates in as a sequence. A phone mockup holds a local video
(`assets/homepage-phone.mp4`) that autoplays muted and loops, with a mute/unmute button in the
video's top-right corner. The mute toggle lives in the logic class's `renderVals()` as
`toggleMute` / `soundIcon`.

### About (`about.html`)
Hero lines animate in sequence, roughly 1.5s per line. Skill icons have hover explainers. The
family photo deliberately overhangs the gray section by 30px top and bottom.

### Work (`work.html`) — the most complex page
Four distinct pieces:

1. **Monitor carousel.** A slide list is returned by `slides()` in the logic class. Each entry
   is `{ src, title, body, fill?, video? }`. The monitor is `assets/monitor-frame.png` — the
   original photo with the screen area cut to transparency — layered *above* the slide via
   `z-index`, so the bezel overlaps the artwork and it reads as a real display. The slide sits
   in an absolutely positioned box at `left:8.6% top:4.5% width:82.9% height:63.8%`. **Do not
   add padding to that box** — it was a long-running visual bug; the frame PNG is cropped
   exactly at the glass edge, so any padding reads as a panel floating on top of the monitor.
   - `fill: true` → `object-fit: cover` (photos, which should bleed to the screen edges)
   - omitted → `object-fit: contain` (screenshots with white backgrounds, which must not crop)
   - `video: <url>` → the slide renders a video player instead of an image
   Slide 1 is the AI Marketing Director hype video with custom controls (see below).
   An orange band (`#C4700F`) is anchored to the monitor element, not the section, so it holds
   still as captions change length.

2. **Custom video player** (`videoSlide()` in the logic class). Mute toggle top-right; a
   control bar at the bottom with rewind 10s, play/pause, forward 10s, and a draggable
   scrubber with a white circle handle. The bar shows on hover, on drag, and while paused, and
   fades during playback. On end, playback stops and a circular replay button appears centered.
   Pointer capture is used for the drag, so `vScrubTo` seeks live as you move.

3. **Bloodlines and Pro XP videos.** Each has an expand icon in the top-right that opens a
   full-screen modal (`epOpen` / `epSrc` state, `epUrls()` returns the six URLs; indices 0-3
   are Bloodlines episodes, 4-5 are the Pro XP films). Bloodlines episodes use generated
   poster images `assets/bl-ep1-v5.jpg` … `bl-ep4-v5.jpg`. The Pro XP films have **no** poster
   file — instead their `src` ends in `#t=3`, a media fragment that makes the browser display
   the frame at 3 seconds. That's a placeholder; the user may supply real timestamps.
   The Pro XP videos are `aspect-ratio: 2.39/1` with `object-fit: cover` because the source
   files have letterbox bars baked into a 16:9 frame; the crop removes them.

4. **Autoplay hardening.** `componentDidMount` walks `video[autoplay]`, forces
   `v.muted = true`, and calls `play()`, retrying at 600ms and 2s and again on
   `visibilitychange`. This exists because React does not reliably set the `muted` attribute,
   and browsers block unmuted autoplay. If autoplay breaks, look here first.

### References (`references.html`)
A strip of eight headshots laid out with flex. On hover, the hovered image's `flex-grow` goes
to 3 while neighbors compress, revealing the full face. Click opens a quote modal.

### Volunteer (`volunteer.html`)
Org logos vertically centered against their text blocks, 26px headings, and a pull quote whose
two lines stagger left and right. The SungateKids logo is white-on-transparent, so its
charcoal ground (`#4A4A4A`, 15px padding on all sides) is **baked into the PNG** — don't
replace that file with the original.

### Contact (`contact.html`)
Full-bleed slate-blue panel, content centered at 1080px. The form submits via a `mailto:`
link to `JustinC.Ulrich@gmail.com`. There is no backend and none is needed.

## Media on R2

Videos live in the R2 bucket, not the repo, because Cloudflare Pages caps individual files at
25 MB. Base URL:

```
https://pub-bc8f6a2cf6154e4d90616851a25b359f.r2.dev/
```

Contents: four `RZR BLOODLINES Ep. N …` files, two `2020 RZR PRO XP …` films,
`Performance Sci 1.mp4` through `6.mp4`, `Camp RZR.mp4`,
`AI Marketing Director Hype Video.mp4`, `RZR ProXP Brochure.pdf`, and a copy of
`Homepage phone video.mp4` (the homepage currently uses the local copy in `assets/`).

Filenames contain spaces and a `®`, so URLs are percent-encoded in the HTML. Preserve the
encoding exactly when editing. To add media, upload to the same bucket and reference
`<base>/<percent-encoded-filename>`.

## Things that will bite you

- **Don't edit `support.js`.** It's the generated runtime.
- **Inline styles only.** No stylesheets, no classes.
- **No expressions in `{{ }}` holes.** Compute in `renderVals()` and expose by name.
- **Don't set fixed pixel `width`/`height` on containers.** Several were converted to
  `max-width` so they collapse instead of clipping at narrow viewports. Same for videos: a
  fixed `height` alongside `aspect-ratio` produces dead space.
- **`support.js` loads React from unpkg.com at runtime.** The site therefore has one external
  dependency. If the user wants full independence, vendor
  `react.production.min.js` and `react-dom.production.min.js` into `assets/vendor/` and point
  `support.js` at them.
- **`assets/` has ~190 files.** Don't bulk-rewrite it.

## Open items

1. Pro XP film preview frames are a `#t=3` placeholder — the user was asked for real
   timestamps and hasn't supplied them.
2. DNS migration to Cloudflare and attaching `justinulrich.com` + `www` as custom domains.
   Canonical tags currently say `www.justinulrich.com`.
3. Wix subscription cancellation, once the live domain serves this site.
4. 17 of the 30 PDFs in `assets/pdfs/` are not linked from any page (extra one-pagers and case
   studies). The user declined additional thumbnails; they're there if wanted later.

## Suggested first move

Copy `CLAUDE.md` from this package into the repo root and commit it. It's a condensed version
of the rules above, so future sessions pick up the constraints automatically.
