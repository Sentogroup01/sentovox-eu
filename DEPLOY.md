# Deploy guide — SentoVox

Step-by-step from local repo to live `https://sentovox.eu`. ~20 minutes once you have a GitHub + Cloudflare account.

## 0. Prerequisites

- Local repo at `C:\Sentovox hugo\sentovox-hugo` with an initial commit (already done — `git log` should show "Initial commit: SentoVox Hugo site").
- GitHub account with permission to create new repositories.
- Cloudflare account (free tier is fine) with `sentovox.eu` already added as a zone, OR access to the DNS for that domain.
- Hugo extended installed locally (for testing): `winget install Hugo.Hugo.Extended`.

## 1. Create a new GitHub repository

1. Open <https://github.com/new>.
2. Fill in:
   - **Repository name:** `sentovox-eu` (or whatever you prefer — must be a fresh repo, not connected to `sentobib-hugo`).
   - **Description:** "SentoVox Hugo site — visitor research for performing arts venues."
   - **Visibility:** Private (recommended) or Public.
   - **DO NOT** tick "Add a README", "Add .gitignore", or "Choose a license". The local repo already has `.gitignore` and `README.md` — adding files on GitHub creates merge conflicts on first push.
3. Click **Create repository**.

## 2. Push your local repo

GitHub will show a panel titled "...or push an existing repository from the command line" with three lines. They look like:

```powershell
git remote add origin https://github.com/<your-username>/sentovox-eu.git
git branch -M main
git push -u origin main
```

Open PowerShell, navigate to the repo, and paste:

```powershell
cd "C:\Sentovox hugo\sentovox-hugo"
git remote add origin https://github.com/<your-username>/sentovox-eu.git
git branch -M main
git push -u origin main
```

If you don't have a credential helper set up, GitHub will ask for a Personal Access Token (PAT) instead of a password. Create one at <https://github.com/settings/tokens> with the `repo` scope.

After the push, refresh GitHub — your code should be visible there.

## 3. Connect Cloudflare Pages

This is a NEW Cloudflare Pages project, separate from any existing `sentobib-hugo` project.

1. Open <https://dash.cloudflare.com> → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**.
2. Authorise Cloudflare to access GitHub if prompted, then select the `sentovox-eu` repo.
3. Configure the build:
   - **Project name:** `sentovox-eu` (this becomes a `*.pages.dev` subdomain by default).
   - **Production branch:** `main`.
   - **Framework preset:** None (or "Hugo" if it appears — same effect).
   - **Build command:** `hugo --gc --minify`
   - **Build output directory:** `public`
   - **Root directory (advanced):** leave empty.
   - **Environment variables (Production):** add one:
     - Name: `HUGO_VERSION`
     - Value: `0.140.2` (or whatever is in `.hugo_version`)
4. Click **Save and Deploy**.

The first build takes 1-3 minutes. When it finishes, you'll get a preview URL like `https://sentovox-eu.pages.dev`. Open it — the site should load with red branding and 6 languages.

## 4. Custom domain (sentovox.eu)

1. In the Cloudflare Pages project: **Custom domains** tab → **Set up a custom domain** → enter `sentovox.eu`.
2. Cloudflare auto-creates the necessary `CNAME` record in your DNS (since the zone is on Cloudflare). If your DNS is elsewhere, you'll get the target hostname to add as a CNAME yourself.
3. Repeat with `www.sentovox.eu` if you want the www variant (recommend redirecting www -> apex via a Cloudflare Page Rule or Bulk Redirect).
4. Wait 1-2 minutes for SSL provisioning. Once green, `https://sentovox.eu` serves your site.

> Note: if `sentovox.eu` already points to the current Wix site, Cloudflare will replace the existing DNS record. The Wix site goes offline immediately. Only do this when you're ready to switch over.

## 5. Switch GA4 from placeholder to real

Before going live publicly, edit `layouts/partials/head/analytics.html` and replace `G-XXXXXXXXXX` with your real GA4 measurement ID (e.g. `G-ABCD123456`). Commit + push:

```powershell
git add layouts/partials/head/analytics.html
git commit -m "Set GA4 measurement ID"
git push
```

Cloudflare auto-rebuilds on push.

## 6. Iterate

Every push to `main` triggers a rebuild. To test locally before pushing:

```powershell
hugo server -D
```

For draft preview deploys (Cloudflare gives you a preview URL per branch), push to a feature branch and open a Pull Request — Cloudflare will comment with the preview link.

## Troubleshooting

- **"hugo: command not found" on Cloudflare** — `HUGO_VERSION` env var missing or misspelled. Set it to a valid Hugo extended version.
- **"failed to load translations: yaml: control characters"** — null bytes / non-printable chars in an `i18n/*.yaml` or `content/**/*.md`. Run a clean re-save in your editor (UTF-8, no BOM).
- **Site loads but pages 404** — `defaultContentLanguageInSubdir = true` means there is no `/`, only `/en/`, `/nl/`, etc. The home `/` redirects to the visitor's preferred language via JavaScript (defined in `layouts/partials/head/meta.html`).
- **Logo / favicon doesn't update** — clear browser cache or open in private window. Cloudflare cache TTL is short for HTML but longer for static assets; you can purge in the Cloudflare dashboard if needed.
- **Build fails with "undefined variable"** — most likely a Hugo template references a front-matter key that no longer exists. Check the exact template + line number in the error and either add the key to the relevant `_index.md` or remove the template line.

## Rollback

Cloudflare Pages keeps every previous deployment. From the project dashboard, click an older deployment → **Rollback** to revert. Alternatively, `git revert <commit>` and push.
