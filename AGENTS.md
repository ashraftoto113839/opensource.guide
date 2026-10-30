# Working in this repo (Base44 sandbox)

This is the `github/opensource.guide` Jekyll site. It is a static site: no API, no database,
no external credentials.

## Running it

```
docker compose -f docker-compose.base44.yml up -d --build
```

- Web entry point: http://localhost:3000 (Jekyll dev server on container port 4000).
- `jekyll serve --watch` rebuilds on save; rebuilds are incremental and land in `_site/`
  (gitignored) inside the bind-mounted checkout.
- After a compose/Dockerfile/dependency change: `docker compose -f docker-compose.base44.yml up -d --build`.

## Non-obvious constraints

- **Ruby 2.6 is required.** `Gemfile.lock` pins `github-pages (191)` / `jekyll 3.7.3` /
  `nokogiri 1.8.4` from 2018; those gems do not build or run on modern Ruby. The image is
  `ruby:2.6-buster` (`docker/base44/Dockerfile`).
- **Debian buster apt is archived.** The Dockerfile rewrites `/etc/apt/sources.list` to
  `archive.debian.org` (with `Acquire::Check-Valid-Until=false`) to install
  `libxml2-dev libxslt1-dev zlib1g-dev cmake curl` needed by nokogiri/commonmarker.
- **No `npm install` needed.** `node_modules/` (primer-core, primer-marketing) is committed
  to git and is the Sass `load_paths` target for `assets/css/index.scss`, so `scripts/bootstrap`
  must not be used as-is.
- Sass compiles with the Ruby `sass` gem from `jekyll-sass-converter 1.5.2`; new Primer
  versions or `dart-sass` are not drop-in compatible with these `@import` paths.
- `jekyll-github-metadata` logs "No GitHub API authentication" — expected offline, not an error.
- Ignore `script/bootstrap` (installs into `vendor/gems`, unnecessary here).

## Verifying

- `curl -fsS http://localhost:3000/` should return the rendered home page; page titles come
  from the `_articles` front matter (e.g. `/starting-a-project/`).
- CSS sanity check: `/assets/css/index.css` returns compiled Primer CSS (non-empty, ~100 KB).
- Tests (lint/prose/html-proofer) run in the container, not on the sandbox host:
  `docker compose -f docker-compose.base44.yml exec -T site script/test`
  (needs the `test/` npm deps: `cd test && npm install`, plus the `kindlegen` binary for ebooks).
