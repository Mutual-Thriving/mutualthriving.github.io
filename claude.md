# Mutual Thriving — Repo Guide

Landing page and content site for the Mutual Thriving movement. Static site built
with **Jekyll** and served via **GitHub Pages** (built-in build, no custom Actions
workflow). Custom domain **mutualthriving.org** (see `CNAME`).

> Note: this file (`claude.md`) is internal and excluded from the published site
> via `exclude:` in `_config.yml`.

## Tech / hosting

- **Jekyll** (kramdown markdown, `permalink: pretty`).
- Plugins: `jekyll-seo-tag`, `jekyll-sitemap` (both GitHub Pages supported).
- Built automatically by GitHub Pages from the configured branch/folder. That
  branch/folder choice lives in **GitHub → Settings → Pages**, not in any file.
- Local dev: `bundle exec jekyll serve`. Config changes require a restart.

## What gets published

- Controlled by `_config.yml`, **not** `.gitignore`. The `exclude:` list keeps
  files in the repo but off the live site (currently: Gemfile(.lock), README.md,
  LICENSE, `READMEs/`, `claude.md`).
- Jekyll auto-ignores dotfiles and `_`-prefixed folders.

## Layout of the repo

```
index.html         # Main landing page (hand-written HTML, not markdown)
manifesto.md       # Full manifesto page
styles.css         # All styling; color theme = CSS vars in :root (top of file)
script.js          # Interactive features
CNAME              # Custom domain (mutualthriving.org)
_config.yml        # Site config, collections, excludes
_layouts/          # default.html, page.html, format.html
_includes/         # Reusable partials: nav, footer, callout, highlight,
                   #   three-yeses, value-card, vision
_drafts/           # Unpublished drafts (e.g. mealprep.md)
writings/          # writings/index.html lists the `writings` collection
img/               # Images
READMEs/           # Setup docs (DOMAIN_SETUP.md, GOOGLE_FORMS_SETUP.md) — excluded
```

## Layouts

- **default.html** — base shell (nav + footer).
- **page.html** — generic content page (title/subtitle + `{{ content }}`).
- **format.html** — for MT *Format* pages. Renders title/subtitle/tags, the page
  content, an optional **Three YESes** block driven by front matter
  (`thrive`, `generalizable`, `others`), and a Discord CTA.

## Collections

- Defined in `_config.yml`. Currently: `writings` (`output: true`,
  permalink `/writings/:title/`). `writings/index.html` loops `site.writings`
  to build the index grid.

---

## Planned structure: Formats + Reviews

**Goal:** Formats are events that follow MT guidelines; Reviews each belong to a
specific Format. Both authored as individual `.md` files. At the bottom of a
Format page, automatically list all Reviews of that Format.

This is a standard Jekyll one-to-many relationship between two collections.
GitHub Pages supports all of it natively (collections + `where` filter, no extra
plugins).

### 1. Two collections (`_config.yml`)

```yaml
collections:
  formats:
    output: true
    permalink: /formats/:title/
  reviews:
    output: true        # false if reviews should only appear embedded
    permalink: /reviews/:title/
```

Files live in `_formats/*.md` and `_reviews/*.md`.

### 2. The link (the crux)

Each review declares which format it belongs to via a front-matter key that
matches the format's stable id (simplest: match the format's filename slug, or
give each format an explicit `id:`).

```yaml
# _reviews/jane-on-deep-dialogue.md
---
format: deep-dialogue      # points at the format
reviewer: Jane
rating: 5
---
Review body...
```

```yaml
# _formats/deep-dialogue.md
---
title: Deep Dialogue
id: deep-dialogue
layout: format
---
```

Relationship is just `review.format == format.id` — no database, a shared string
maintained by hand. (Typo = review silently won't appear; keep ids = filenames.)

### 3. Auto-list at the bottom of `_layouts/format.html`

Write the snippet **once** in the layout, not per file. Every format uses
`layout: format`, so new reviews dropped in `_reviews/` show up automatically on
the next build. Conceptually:

```liquid
<h2>Reviews of this format</h2>
{% assign matching = site.reviews | where: "format", page.id %}
{% for review in matching %}
  ... review.reviewer, review.rating, review.url ...
{% empty %}
  <p>No reviews yet.</p>
{% endfor %}
```

Key line: `site.reviews | where: "format", page.id` filters the whole reviews
collection to the current format.

### Notes

- Reverse link (review → its format):
  `site.formats | where: "id", page.format | first`.
- `output: true` vs `false` for reviews = whether each review gets its own URL or
  only appears embedded under its format.
- The link is convention, not enforced — a typo'd `format:` value just won't match.
