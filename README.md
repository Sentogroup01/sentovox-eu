# SentoVox — Hugo site

European visitor research site for performing arts venues. Built with [Hugo](https://gohugo.io) (extended), 6 languages.

## Quick start

```powershell
# 1. Install Hugo extended (one-time)
winget install Hugo.Hugo.Extended

# 2. Run dev server
cd C:\Sentovox hugo\sentovox-hugo
hugo server -D

# 3. Open http://localhost:1313
```

The required Hugo version is pinned in `.hugo_version` (currently `0.140.2`). Cloudflare Pages reads this file too.

## Project layout

```
archetypes/          Hugo content archetypes
config/              Per-environment overrides (production)
content/             6 languages x 5 pages (en, nl, fr, de, it, es)
i18n/                UI translation strings (yaml per language)
layouts/             HTML templates (rich home + simple article layout for the rest)
static/              Images, CSS, JS, robots.txt, llms.txt
hugo.toml            Main config (title, baseURL, languages, menus)
.hugo_version        Hugo version pin
```

## Pages per language

- `/` (home) — hero, trust bar, partners, intro, FAQ, World Land Trust
- `/details/` — methodology, modules, security, target venues
- `/participate/` — timing, prices, three-step process, helpdesk
- `/contact/` — contact form (Formspree)
- `/privacy/` and `/terms/` — legal

## Brand

- Primary red: `#ec7c25` — defined as CSS variable `--teal` (variable name kept from template; value swapped). Dark variant `#c46220`, light `#f5a566`.
- Fonts: Playfair Display (display) + DM Sans (body).
- Logo + favicon + 1200x630 og-image: `static/img/{logo,favicon,og-sentovox}.svg` (PNG fallbacks alongside).

## Editing content

Most page content lives in markdown under `content/{lang}/`. The home page (`_index.md`) uses front-matter heavily for structured sections (hero, trust items, why blocks, FAQ, World Land Trust) — edit YAML keys at the top of each language's `_index.md` and the layout renders them.

UI strings (nav labels, footer, CTA texts) are in `i18n/{lang}.yaml`.

## Analytics

GA4 placeholder is in `layouts/partials/head/analytics.html`. Replace `G-XXXXXXXXXX` with your real measurement ID before going live. Until then, no tracking script is emitted.

## Build for production

```powershell
hugo --gc --minify
# Output goes to public/
```

## Deploy

See `DEPLOY.md` for GitHub + Cloudflare Pages setup.
