# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## What this repository is

This repo is based on the **GitHub Skills "GitHub Pages" course** — an
interactive, self-guided tutorial that teaches you to publish a site or blog
from a GitHub repository using **GitHub Pages** (Jekyll). It began as a template
repo (`skills/github-pages`) and now also contains a small learner-added static
page (`Map.html`).

There are two distinct things living here, and they are easy to confuse:

1. **The course machinery** — `README.md`, `.github/steps/`, and
   `.github/workflows/` implement a guided lesson that advances step-by-step as
   you push commits and configure the repo. The `README.md` is *not* normal
   project documentation; it is the current lesson page and is rewritten
   automatically by the workflows.
2. **The actual site content** — the Jekyll site (created on a `my-pages`
   branch during the course) plus `Map.html`, a standalone Leaflet map.

## Repository layout

```
.
├── README.md              # The live course lesson page (auto-updated by CI — do not hand-edit for docs)
├── Map.html               # Standalone Leaflet + OpenStreetMap page with browser geolocation
├── LICENSE                # MIT
├── .gitignore             # Ignores compiled artifacts, archives, logs, OS files
└── .github/
    ├── dependabot.yml     # Monthly updates for github-actions
    ├── steps/             # Markdown for each lesson step (0..5, X-finish) + -step.txt (current step)
    └── workflows/         # One workflow per step; each advances the course to the next step
```

Note: The Jekyll site files the course produces — `_config.yml`, `index.md`,
`_posts/*.md` — are created on the **`my-pages`** branch by the Step 0 workflow,
not committed on `main`. Do not expect them on `main` unless a PR has been
merged.

## How the course automation works

Each `.github/workflows/N-*.yml` follows the same pattern:

- It reads the current step number from `.github/steps/-step.txt`.
- A `get_current_step` job gates the real job so it only runs when the repo is
  on the matching step (and is not the template repo itself).
- On the triggering event, it uses `skills/action-update-step@v2` to rewrite
  `README.md` from `.github/steps/<from>.md` to `.github/steps/<to>.md` and bump
  `-step.txt`.

Triggers by step:

| Workflow | Trigger | Advances |
|----------|---------|----------|
| `0-welcome.yml` | push to `main` | creates `my-pages` branch + `_config.yml`/`index.md`, 0 → 1 |
| `1-enable-github-pages.yml` | `page_build` | 1 → 2 |
| `2-configure-your-site.yml` | push to `my-pages` touching `_config.yml` | 2 → 3 |
| `3-customize-your-homepage.yml` | push to `my-pages` touching `index.md` | 3 → 4 |
| `4-create-a-blog-post.yml` | push to `my-pages` touching `_posts/*.md` | 4 → 5 |
| `5-merge-your-pull-request.yml` | push to `main` (PR merge) | 5 → X (finish) |

All workflows also support `workflow_dispatch` for manual runs.

**Implication for edits:** `README.md` is machine-managed lesson state. If you
need to document the *project*, prefer adding a separate doc (or this file)
rather than editing `README.md`, which CI may overwrite.

## Map.html

A single self-contained HTML file — no build step, no dependencies to install.

- Loads **Leaflet 1.7.1** CSS/JS from the unpkg CDN.
- Renders a full-viewport `#map`, centered provisionally on Asunción, Paraguay
  (`[-25.2637, -57.5759]`, zoom 13).
- Uses the browser **Geolocation API** to recenter on the user's location
  (zoom 15) and drop an "Estás aquí" marker; logs an error to the console if
  geolocation is unavailable or denied.
- UI strings are in **Spanish**. Keep new user-facing strings in Spanish for
  consistency.

To view it: open `Map.html` directly in a browser, or serve the repo root with
any static server (e.g. `python3 -m http.server`) and browse to `/Map.html`.
Geolocation requires a secure context, so browsers grant it on `localhost` and
over HTTPS (as when served by GitHub Pages), but not from a plain `file://` URL
in some browsers.

## Local development

There is no package manager, build, or test setup — this is static
HTML/Markdown plus GitHub Actions YAML.

- **Preview static files:** `python3 -m http.server 8000` then open
  `http://localhost:8000/Map.html`.
- **Preview the Jekyll site** (only relevant on `my-pages` once site files
  exist): `bundle exec jekyll serve` if you have Ruby/Jekyll installed; GitHub
  Pages builds it automatically on push otherwise.
- **Validate workflow YAML** before pushing changes to `.github/workflows/`.

## Conventions and guardrails

- **Branch for this work:** develop on `claude/claude-md-docs-d5njyb`; do not
  push to `main` or `my-pages` without explicit permission. Create the branch
  from the latest `main` if it doesn't exist.
- **Don't hand-edit `README.md` as documentation** — it is the CI-managed
  lesson page keyed to `.github/steps/-step.txt`. Changing one without the other
  desyncs the course.
- **Keep `Map.html` self-contained** (inline CSS/JS, CDN for Leaflet). Match the
  existing Spanish-language, comment-annotated style.
- **Commit messages:** short and descriptive; the existing history uses concise
  present/imperative summaries (e.g. `Map OpenStreetMap`).
- Do not create pull requests unless explicitly asked.

## Deployment

The site is published via **GitHub Pages**. Once Pages is enabled (course
Step 1) and content is merged to the Pages branch, GitHub builds and serves it
automatically — there is no manual deploy step. `Map.html` is served as a
static file at its path.
