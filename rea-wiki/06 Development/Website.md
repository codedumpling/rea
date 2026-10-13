---
tags: [rea, development, website, docs]
source:
  - website/README.md
  - website/AGENTS.md
  - website/style-guide.md
  - website/wrangler.toml
  - CONTRIBUTING.md
  - package.json
  - .github/workflows/website-pages.yml
  - .github/workflows/pages.yml
---
# Website

REA has **two** documentation surfaces. Don't confuse them.

| Surface | Source | Tooling | Published |
| --- | --- | --- | --- |
| **rea.tools** (public site) | `website/public/` (English HTML) + `website/locales/` (gettext PO translations) | Plain HTML/CSS and small Python scripts (`scripts/website.py`). No framework, bundler or npm dependency | Cloudflare Workers Static Assets (`website/wrangler.toml`) and/or GitHub Pages via the manual `website-pages.yml` |
| **Docs site** | Markdown in `docs/` | VitePress (`docs/.vitepress/config.ts`) | Not published by `pages.yml`, which is a manual build check only |

## rea.tools

The English site has translated editions for every README language. It includes explanatory figures, worked guides, the dinosaur speed mini-game, and case studies: Aegis (Android login code), DX-Ball (sound pan), Notion (Electron clipboard bridge), TH04 (16-bit DOS bullet ring) and a DownUnderCTF checker. These appear in the vault as [[Showcases]]. The sources for case-study claims live in `website/evidence/`.

A build writes every language edition to `website/_site/` (ignored; never edit it by hand). Language switching needs no JavaScript.

### Local preview

```sh
python3 -m venv website/.venv
source website/.venv/bin/activate
python3 -m pip install -r website/requirements.txt
python3 scripts/website.py serve        # http://127.0.0.1:4173/
python3 scripts/website.py serve --port 4174 --base-path /rea/   # test Pages prefix
```

CairoSVG needs the native Cairo library, and the sharing card uses DejaVu Sans (`sudo apt-get install libcairo2 fonts-dejavu-core` on Debian/Ubuntu). After editing source HTML or a catalog, run `python3 scripts/prepare-website.py`.

### Editing

| Change | Edit | Then run |
| --- | --- | --- |
| Page text, examples, figures | The page HTML, outside the marked shared regions | `python3 scripts/website.py check` |
| Title / description | `<title>` and the first description meta | `sync`, then `check` |
| Main navigation | `NAVIGATION` in `scripts/website.py` | `sync`, then `check` |
| Header / footer / sharing | `website/templates/*.html.tmpl` | `sync`, then `check` |
| Translation | `locales/<language>/messages.po` | `translations check`, then `check` |

- After changing English text, run `python3 scripts/website.py translations extract`. Existing translations and reviewer comments survive extraction.
- Create a new page with `python3 scripts/website.py new-page blog/my-article --title "..." --description "..."`. This also works under `guides/` and `showcase/`. New pages start as a visible draft with `noindex`.
- Optional external link audit: `python3 scripts/website.py links --report /tmp/rea-website-links.json` (uses `gh` for GitHub links).
- `check` works offline. It validates shared regions, translations, links, SEO metadata and publishing config. PR checks (`website-check.yml`) and the publisher run this same command.

> [!tip] Writing style
> Read `website/style-guide.md` first. Each paragraph should explain a new fact or action. Keep code, commands, URLs and Evidence unchanged in translations. When updating setup or support claims, compare them against `docs/installation.md`, the provider guides and the released package.

### Deploying

- **Cloudflare Workers:** Wrangler runs the Python build and verification, then serves `_site/` as static assets. There is no Worker script. You need Node.js and Python 3. A pinned Wrangler version is used without adding it to REA's dependencies.
- **GitHub Pages:** only through the manual `website-pages.yml` workflow on `main` (`target`: `both` or `pages`). The repository's *Settings → Pages → Source* must be set to **GitHub Actions**.

## VitePress docs

```bash
npm run docs:dev       # live editing (runs docs:generate first)
npm run docs:build     # production build + link check
npm run docs:preview   # preview at /rea/
npm run docs:generate  # generated reference documents
npm run docs:check     # PR gate: generated-doc validation + README consistency
```

Keep links between guides **relative**, so they work on both GitHub and the site. Link to repository files outside `docs/` with full GitHub URLs. Site navigation lives in `docs/.vitepress/config.ts`.

See also [[Doc Index]], [[CI Workflows]], [[Contributing]].

## Source
- https://github.com/morluto/rea/blob/main/website/README.md
- https://github.com/morluto/rea/blob/main/CONTRIBUTING.md#documentation-website
