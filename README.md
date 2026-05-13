# kysonyangs.github.io

Jekyll blog source for `https://kysonyangs.github.io`, using the Chirpy theme.

## Requirements

- Ruby 3.0+
- Bundler

## Run Locally

```bash
bundle exec jekyll serve --host 127.0.0.1 --port 4000
```

Open:

```text
http://127.0.0.1:4000/
```

## Build Check

```bash
bundle exec jekyll build
```

Generated files are written to `_site/`. Do not commit `_site/`; it is ignored.

## Publish

This repository publishes through GitHub Actions:

1. Push changes to `main` or `master`.
2. GitHub Actions runs `.github/workflows/pages.yml`.
3. The workflow builds Jekyll and deploys `_site/` to GitHub Pages.

Repository setting required:

```text
Settings -> Pages -> Build and deployment -> Source -> GitHub Actions
```

## Add A Post

Use the helper script:

```bash
scripts/new-post "Post title" --tags "iOS,Swift"
```

Use `--slug` if you want to control the URL part:

```bash
scripts/new-post "Post title" --slug post-title --tags "iOS,Swift"
```

Preview the generated content without writing a file:

```bash
scripts/new-post "Post title" --tags "iOS" --dry-run
```

Available options:

- `--slug`: custom URL and file slug.
- `--date`: custom post date, for example `2026-05-13 10:00:00 +0800`.
- `--tags`: comma-separated tags.
- `--categories`: comma-separated categories.
- `--dry-run`: preview without creating a file.

The script creates a Markdown file in `_posts/`:

```text
_posts/YYYY-MM-DD-title.md
```

Front matter example:

```yaml
---
layout: post
title: "Post title"
date: 2026-05-13 10:00:00 +0800
tags: ["iOS"]
---
```

Post URLs use:

```yaml
permalink: /:title/
```

## Project Layout

- `_posts/`: blog posts.
- `_tabs/`: Chirpy sidebar pages, including archives, tags, and about.
- `_data/`: Chirpy contact and sharing configuration.
- `_config.yml`: site metadata, theme settings, plugins, and permalink rules.
- `google41eca6f1588fb155.html`: Google site verification file.
- `robots.txt`, `sitemap.xml`, `feed.xml`: generated SEO and subscription files.
