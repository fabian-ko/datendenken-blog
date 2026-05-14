# datendenken-blog

Source for [blog.datendenken.de](https://blog.datendenken.de) — a Hugo static
site deployed for free via GitHub Pages.

## Stack at a glance

```
Author (you) ──► Markdown in content/ ──► Hugo (static site generator)
                                            │
                                            └─► HTML in public/
                                                  │
                                                  └─► GitHub Pages (free hosting)
                                                        │
                                                        └─► https://blog.datendenken.de
                                                              ▲
                                                              │ CNAME record
                                                              │
                                                          DNS at your registrar
```

## Repo layout — what lives where

| Path | Purpose | When you touch it |
|---|---|---|
| `hugo.toml` | Site-wide config: title, baseURL, language, menu, theme params, author block | Change site title, menu, dark/light default, author bio |
| `content/_index.md` | Home-page intro text | Edit homepage copy |
| `content/about.md` | About page | Edit about copy |
| `content/posts/` | Blog posts, one `.md` per post | **Write new posts here** |
| `content/posts/_index.md` | Title of the post list page | Rename "Beiträge" |
| `static/` | Files copied verbatim to site root | Drop in images, `robots.txt`, `favicon.ico`, etc. |
| `static/CNAME` | Binds the custom domain on GH Pages | Don't touch unless changing domain |
| `assets/` *(create if needed)* | Avatar image (`avatar.jpg`), custom JS/CSS that Hugo processes | Add author avatar, custom scripts |
| `layouts/partials/custom-head.html` | Extra `<head>` content — currently the CSP | Add analytics tags, extra meta tags, tighten CSP |
| `layouts/` *(other files)* | Per-site template overrides; theme is the fallback | Override theme rendering on a single template without forking |
| `themes/hugo-blog-awesome/` | Theme submodule (do not edit) | Run `git submodule update --remote` to pull theme updates |
| `.github/workflows/hugo.yml` | CI: builds & deploys on push to `main` | Bump Hugo version, change build flags |
| `.gitignore` / `.gitmodules` | Standard plumbing | — |

## Common tasks

### Write a new post

1. Create `content/posts/my-post.md` with front matter:
   ```yaml
   ---
   title: "Titel"
   date: 2026-05-14
   draft: false
   tags: ["foo"]
   ---
   ```
2. Commit to `main` (or a branch + PR). Push → workflow rebuilds → live in ~1 min.
3. `draft: true` keeps it out of production.

### Preview locally

Requires Hugo extended ≥ 0.87:

```
hugo server -D
```

→ http://localhost:1313 (the `-D` shows drafts).

### Add your avatar

Drop `avatar.jpg` (square, ~256×256) into `assets/` (not `static/`). The theme
reads it via Hugo's asset pipeline.

### Tweak the theme (colors, fonts, menu, social links, web manifest)

Everything is in `hugo.toml` under `[params]`, `[[menu.main]]`, `[params.author]`,
`[[params.socialIcons]]`. Reference:
[`themes/hugo-blog-awesome/exampleSite/hugo.toml`](themes/hugo-blog-awesome/exampleSite/hugo.toml).

### Override a single theme template

Copy the file you want to override out of `themes/hugo-blog-awesome/layouts/...`
into `layouts/...` at the same path. Hugo picks site `layouts/` over theme
`layouts/`.

### Update the theme

```
git submodule update --remote themes/hugo-blog-awesome
```

Test the build, then commit the new submodule SHA.

## Deploy pipeline (`.github/workflows/hugo.yml`)

- Trigger: push to `main` or manual "Run workflow" in the Actions tab.
- Steps: install Hugo extended → install Dart Sass → checkout (with submodules)
  → configure Pages → `hugo --minify` → upload artifact → deploy to Pages.
- baseURL comes from `hugo.toml`, not from the workflow.
- To bump Hugo, change `HUGO_VERSION` at the top of the workflow.

## Hosting & DNS layer

- **GitHub Pages** (free) serves `public/` from the workflow artifact.
- `static/CNAME` tells Pages which domain to bind → `blog.datendenken.de`.
- **DNS** (at your registrar): `CNAME blog → fabian-ko.github.io.` — that's the
  only record needed.
- **HTTPS**: GH Pages auto-provisions a Let's Encrypt cert once DNS resolves.
  Then tick *Enforce HTTPS* in repo Settings → Pages.

## Security knobs

- `layouts/partials/custom-head.html` — CSP meta tag. Tighten/loosen as you add
  features (analytics, embeds).
- `markup.goldmark.renderer.unsafe = true` in `hugo.toml` — lets raw HTML in
  markdown render. Fine because you're the only author; CSP is the backstop.
- Theme submodule is pinned to a commit SHA — supply-chain mitigation.
  Re-pinning is a conscious step.

## Quick "where do I…" cheatsheet

| I want to… | Go to |
|---|---|
| …write a post | `content/posts/<slug>.md` |
| …change site title / menu / colors | `hugo.toml` |
| …add an image to a post | put it next to the post `.md` and reference relatively, or in `static/img/` |
| …add analytics | extend `layouts/partials/custom-head.html` **and** loosen CSP for the provider |
| …override theme styling | copy the file from `themes/hugo-blog-awesome/assets/` to local `assets/` |
| …change the deploy branch or Hugo version | `.github/workflows/hugo.yml` |
| …add a new top-level page (e.g. /impressum) | `content/impressum.md` + add to menu in `hugo.toml` |
| …point a different subdomain at the site | edit `static/CNAME` + update DNS |
