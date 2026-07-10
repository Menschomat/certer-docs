# Certer Documentation

This repository contains the Hugo documentation site for Certer.

## Local Development

Install Hugo Extended, then start the local development server:

```sh
hugo server --bind 127.0.0.1 --port 1313 --buildDrafts
```

Open http://127.0.0.1:1313/ to view the site.

## Build

Build the static site into `public/`:

```sh
hugo --buildDrafts
```

To test the GitHub Pages project URL locally, build with the deployed base URL:

```sh
hugo --buildDrafts --baseURL https://menschomat.github.io/certer-docs/
```

## Deployment

GitHub Pages deployment is handled by `.github/workflows/deploy.yml`.
The workflow uses `actions/configure-pages` and passes the Pages base URL to Hugo, so generated links keep the `/certer-docs/` prefix on GitHub Pages.

## Structure

- `content/` contains the documentation pages.
- `layouts/` contains the custom Hugo templates.
- `static/` contains CSS and image assets copied directly into the built site.
- `hugo.toml` contains site-level configuration.
