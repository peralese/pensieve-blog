# Digital Pensieve

> The Digital Pensieve is not merely a tool for recollection; it's a portal to the past and a
> window into the present. It is a place where memories can be examined and understood.

Erick's personal blog — book reviews, travel writeups, gardening notes, recipes, and whatever
else is on his mind. Built with [Hugo](https://gohugo.io/) on the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, customized into a warm
editorial/magazine layout.

Live at [pensieve.peraleslab.com](https://pensieve.peraleslab.com/).

## Tech stack

- **[Hugo](https://gohugo.io/)** (extended) — static site generator
- **[PaperMod](https://github.com/adityatelange/hugo-PaperMod)** — base theme, included as a
  git submodule at `themes/PaperMod`. It's never edited directly — all customization lives in
  project-level `layouts/` and `assets/css/extended/` files that Hugo's lookup order lets
  shadow the theme's own, so the theme stays cleanly upgradeable.
- **GitHub Actions → S3 + CloudFront** — pushing to `main` builds the site (`hugo --minify`)
  and syncs it to S3 behind CloudFront (see `.github/workflows/deploy.yml`). There's no
  separate build/deploy step to run by hand.

## Content model

Everything lives under `content/posts/` as a single unified feed — that's the one section,
regardless of topic. Two taxonomies layer on top of it instead of splitting content into
separate sections:

- **`categories`** — content type: `Reviews`, `Recipes`, `Gardening`, `Travel`, or `Interests`. Filing a
  post under one of these makes it show up on that nav tab (e.g. `/categories/travel/`) in
  addition to the main Posts feed. Leave it unset for a general post — it just shows up in
  Posts with an "Essay" badge.
- **`tags`** — freeform genre/topic keywords (e.g. `Sci-Fi`, `Warhammer 40k`, `Salado`).
  These power the "Browse Topics" sidebar widget.

  **Don't reuse a category name as a tag** (e.g. a tag literally called `Travel`) — Hugo's
  `tags` and `categories` taxonomies share the same URL/slug space, so a tag and a category
  with the same name collide and break the build.

Each post is a Hugo **page bundle** — a folder (`content/posts/<slug>/index.md`) with any
images living right alongside it, so `![alt](photo.jpg)` in the body just works with no path.

### Front matter reference

```toml
+++
title = 'My Post Title'
date = '2026-07-12T10:00:00-05:00'
draft = false                      # false = published
categories = ['Travel']            # optional — Reviews/Recipes/Gardening/Travel/Interests
tags = ['Salado', 'Texas']         # optional — freeform
coverImage = 'hero-photo.jpg'      # optional — see below
+++

Body content here, in Markdown.
```

- **`coverImage`** (optional) — explicitly picks which bundle image is "the" main one for
  this post. It's used consistently for the post's own hero image, its thumbnail on every
  listing page (home, `/posts/`, category pages), and the `{{< gallery >}}` shortcode's main
  image. Without it, whichever image happens to sort first alphabetically wins by default.

### Photo galleries

For a post with several images, drop the `gallery` shortcode into the body:

```
{{< gallery >}}
```

Renders a large main image (the `coverImage` if set, otherwise the first one found) followed
by a thumbnail strip of the rest; each thumbnail opens its full-resolution original in a new
tab. To control the exact set/order yourself instead of auto-discovering everything in the
bundle:

```
{{< gallery "photo2.jpg" "photo4.jpg" >}}
```

**Note:** the shortcode has to actually be typed into the body — just having images in the
folder isn't enough to make them appear on the post page itself (though the *thumbnail* on
listing pages works automatically either way, since that's driven separately by `coverImage`
or the first bundle image).

## Local development

```bash
git clone --recurse-submodules <repo-url>
cd pensieve-blog
hugo server -D          # -D includes drafts; open http://localhost:1313
```

If you cloned without `--recurse-submodules`, run `git submodule update --init` to pull in
the PaperMod theme.

To create a new post by hand:

```bash
hugo new content posts/my-new-post/index.md
```

This uses `archetypes/default.md` to pre-fill `title`/`date`/`draft`/`tags`. Add `categories`
and `coverImage` yourself as needed (see above).

### Local upload tool (optional, not in this repo)

There's a small local-only helper — a single-file Python script serving a form at
`http://127.0.0.1:8787` — that scaffolds a new post (title, category, tags, cover image,
additional images, draft/publish toggle) without touching TOML by hand. It lives in
`local-tools/`, which is gitignored on purpose (an admin tool with filesystem write access
has no business being in a public repo, and it only exists on whichever machine you run it
from — see `.gitignore`).

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds with `hugo --minify`
and syncs `public/` to S3 (via OIDC-assumed AWS credentials), then invalidates the CloudFront
cache. There's no need to build or deploy locally — just commit and push.

Note: `public/` is also committed in this repo as a leftover from earlier setup, but it isn't
actually the deploy source of truth (the Action rebuilds fresh every time) — don't worry about
keeping it in sync when working locally.

## Project structure

```
content/
  posts/<slug>/index.md    # every post lives here (page bundles, images alongside)
  categories/               # Reviews/Recipes/Gardening/Travel/Interests taxonomy landing pages
  about/                     # About page (custom layout, has its own avatar image)
  search.md                  # activates PaperMod's built-in Fuse.js search
  archive.md                 # activates PaperMod's built-in year-grouped archive
layouts/
  list.html                  # project override: hero + card grid + sidebar (home/section/tag pages)
  about.html                 # custom About page layout
  _partials/
    footer.html                # project override: adds the 3-column magazine footer
    sidebar.html                # About/Browse Topics/Recent Reviews/Archive widgets
    post-card.html               # renders one post as a card (used by list.html)
  _shortcodes/
    gallery.html                  # the {{< gallery >}} shortcode
assets/css/extended/
  magazine.css                    # all custom styling (auto-loaded by PaperMod, no config needed)
archetypes/default.md              # front matter template for `hugo new content`
hugo.toml                          # site config, nav menu, params
```

## Author
Erick Perales - IT Architect, Cloud Migration Specialist
https://github.com/peralese