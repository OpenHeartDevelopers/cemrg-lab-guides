# CEMRG Lab Guides

Internal onboarding and reference guides for the CEMRG research group.

The rendered site lives at GitHub Pages (enable Pages on `main` / root once pushed).

## Local preview

```shell
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000>.

## Adding a page

1. Drop a markdown file into the right section under `docs/`.
2. Add front matter so `just-the-docs` knows where it goes:

```yaml
---
title: My new guide
parent: New machine setup     # must match the parent's `title`
nav_order: 5
---
```

3. Commit and push. GitHub Pages rebuilds on push.
