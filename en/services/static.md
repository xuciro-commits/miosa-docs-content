---
slug: "services/static"
title: "Static sites from Git"
description: "Build a static site from a repository and serve it from the static lane, with an optional build command and output directory."
section: "Services & projects"
order: 1007
---

A static service takes a git source like a web service does, and is served from
the static lane instead of a container.

```bash
miosa service create shop site --type static \
  --repo https://github.com/acme/site --branch main
miosa service update shop site --build-command "npm run build" --output-dir dist
```

## How the site is built

The build runs on a build cell: Railpack detects the framework, or a Dockerfile
in the repository is used when there is one. `build_command` replaces the build
command Railpack would use (Railpack builds only), and `output_dir` says which
folder to serve.

With neither set, the output folder is discovered: Railpack's own static root
first, then `dist`, `build`, `out`, `public` and `_site`, taking the first one
with an `index.html` at its top. A Dockerfile build also tries the image's own
document root (`nginx`, `httpd`, `caddy`).

## When the build fails

- `reason_code: build_failed` with `reason_detail` starting
  `static_output_missing:` means no `index.html` was found in any candidate
  folder. Set `output_dir` to the folder your build writes.
- `static_site_too_large:` means the site is over 500 MiB or over 20,000 files.

> **Note: Framework example**
>
> A Vite site needs nothing: Railpack detects Vite, builds it, and the output is
> `dist`. Single-page-app fallback routing is a separate change; until then a deep
> link into a client-side route should be handled by the app.
