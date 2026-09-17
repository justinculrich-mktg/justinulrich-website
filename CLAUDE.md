justinulrich.com
Personal portfolio site. Static, no framework, no build step. Deployed on Cloudflare Pages
from `main` — push and it goes live. Repo root is the site root.
Structure
Six pages (`index`, `about`, `work`, `volunteer`, `references`, `contact` as `.html`), plus
`Nav.dc.html` and `Footer.dc.html` imported by each page, `support.js` (the rendering runtime),
and `assets/` (images, `assets/pdfs/`, favicon, one video).
Each page holds an `<x-dc>` template and a `<script type="text/x-dc" data-dc-script>` with a
`class Component` supplying logic. `{{ name }}` holes resolve from `renderVals()`.
Rules
Inline `style=""` only. No stylesheets, no CSS classes. Pseudo-states via
`style-hover` / `style-active` / `style-focus`.
Never edit `support.js` — it's generated.
No expressions in `{{ }}` — dotted lookups only. Compute in `renderVals()`.
No fixed pixel widths/heights on containers. Use `max-width` so they collapse at narrow
viewports. Don't combine a fixed `height` with `aspect-ratio` on video.
Don't introduce a build step, framework, or npm dependency.
Fonts: Oswald 500 headings, Figtree 400 body. Nav 23px. Buttons fully rounded.
Palette: navy `#1B2A41`, slate `#3F5B7A`, orange `#CE7213` (hover `#E08A2E`), body text
`#4A5561`, headings `#2E3A47`.
Media
Videos live on Cloudflare R2, not in the repo (Pages caps files at 25 MB):
```
https://pub-bc8f6a2cf6154e4d90616851a25b359f.r2.dev/
```
Filenames contain spaces and `®` — keep URLs percent-encoded exactly as written.
Known quirks
`work.html`: the monitor carousel layers `assets/monitor-frame.png` (screen cut to
transparency) above the slide. Do not add padding to the screen box — the PNG is cropped
at the glass edge, so padding reads as a panel on top of the monitor.
`work.html`: `componentDidMount` forces `muted = true` and calls `play()` on
`video[autoplay]`, retrying at 600ms/2s and on `visibilitychange`. React doesn't reliably
set `muted`, and browsers block unmuted autoplay. Autoplay bugs start here.
`work.html`: Pro XP videos are `aspect-ratio: 2.39/1` + `object-fit: cover` to crop
letterbox bars baked into the source files. Their `src` ends in `#t=3` to show a real frame
instead of a poster image.
`volunteer.html`: the SungateKids logo's charcoal ground is baked into the PNG. Don't swap
in the original white-on-transparent file.
`contact.html`: the form is a `mailto:` link to JustinC.Ulrich@gmail.com. No backend.
`support.js` loads React from unpkg.com at runtime — the one external dependency. Vendor it
into `assets/vendor/` if full independence is wanted.
Rendering is client-side; crawlers see only the `<head>`. Per-page title/description/OG tags
are static there so search snippets and link previews work.
