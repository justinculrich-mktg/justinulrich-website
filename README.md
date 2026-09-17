# justinulrich.com — Cloudflare Pages deploy

The contents of this folder are the site root. Copy everything here into the repo root
(replacing the old `index.html`, `styles.css`, `images/`) and Cloudflare Pages will serve it
with no build step.

## Cloudflare Pages settings
- Framework preset: **None**
- Build command: *(leave empty)*
- Build output directory: `/` (or `deploy` if you keep this folder as-is)
- Node version: not needed

Pages serves `/work` for `work.html` automatically, so URLs are clean.

## What's here
- `index.html`, `about.html`, `work.html`, `volunteer.html`, `references.html`, `contact.html`
- `Nav.dc.html`, `Footer.dc.html` — shared header/footer, loaded by every page. **Keep these filenames.**
- `support.js` — rendering runtime. Must stay at the site root.
- `assets/` — all images, the resume PDF, and video. Self-hosted; nothing points at Wix.
- `_headers` — long cache on `/assets/*`, basic security headers.
- `robots.txt`, `sitemap.xml`

## Editing
The editable source is the project-root `*.dc.html` files. This `deploy/` folder is generated
output — edit the source, then regenerate. Nav links here are absolute clean URLs (`/about`), so
these copies only browse correctly when served at a site root, not from a subfolder.

## Contact form
Submits through a `mailto:` link to JustinC.Ulrich@gmail.com. No server or form service required.

## Known follow-ups
1. **Volunteer page** still needs a fidelity pass against the Wix original.
2. **React from CDN** — `support.js` loads React from unpkg.com at runtime. It works, but the site
   depends on that CDN staying up. If you want zero external dependencies, the fix is to vendor
   `react.production.min.js` and `react-dom.production.min.js` into `assets/vendor/` and point
   `support.js` at them.
3. **Client-side rendering** — page HTML is assembled in the browser, so search crawlers and
   LinkedIn/Twitter link previews see only the `<head>`. Per-page `<title>`, description and
   Open Graph tags are static in the `<head>` so previews and search snippets work, but the body
   copy isn't in the initial HTML.

## Domain / DNS
See the notes in chat. Short version: move `justinulrich.com` nameservers to Cloudflare, then
attach the domain to the Pages project as a custom domain.
