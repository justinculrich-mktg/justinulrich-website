# Cloudflare Pages setup — justinulrich.com

## Project
- Pages project: `justinulrich-website`
- Connected repo: `justinculrich-mktg/justinulrich-website`, branch `main`
- Framework preset: **None**
- Build command: *(empty)*
- Build output directory: `/`
- Preview URL: `https://justinulrich-website.pages.dev`

No build step. Push to `main` → Pages deploys the repo root as-is. Pages serves `/work` for
`work.html` automatically, which is why URLs are clean.

Note: the project was created via Git integration, so it cannot be switched to Direct Upload
later. That's fine for this setup.

## R2 bucket
- Bucket: `justinulrich-website-media`
- Public access: enabled via r2.dev subdomain
- Base URL: `https://pub-bc8f6a2cf6154e4d90616851a25b359f.r2.dev/`

Holds all video (12 files) plus the RZR Pro XP brochure PDF, which exceeds the 25 MB Pages
file cap. Free tier covers this usage with no egress charges.

## DNS migration (in progress)
1. Add `justinulrich.com` as a site in Cloudflare (Free plan). It scans existing records.
2. Cloudflare returns two nameservers. Set those at the registrar.
   - Check whether the domain is registered through Wix. If so, unlock it and get the auth
     code there first; Wix can be slow to release domains.
3. Wait for propagation (usually under an hour).
4. Pages project → **Custom domains** → add `justinulrich.com` and `www.justinulrich.com`.
   Cloudflare issues SSL automatically.
5. Confirm the live domain serves the new site, then cancel Wix.

Canonical tags in every page's `<head>` point at `https://www.justinulrich.com/...`, so `www`
is the intended canonical host. Redirect the bare domain to `www` (or change the canonicals if
flipping that decision).

## Headers
`_headers` at the repo root sets:
- `/assets/*` → `Cache-Control: public, max-age=31536000, immutable`
- `/*` → `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`

Because assets are cached for a year, **changing an image requires a new filename**, not an
overwrite — that's why several files carry `-v2`/`-v5` suffixes.
