# Contributing

Pull requests are welcome: translation fixes, new languages, corrections and clearer explanations.
Small fixes can be made directly in the GitHub web editor ("Edit this file" → "Propose changes").

English (`en/`) is the source of truth. The translations were largely machine-assisted, so native speakers
reviewing them are especially welcome. When the English pages change, translations may lag behind; a pull request
bringing a language up to date is always welcome.

## How the pages are organised

Each language has the same five pages under its own path (`en/`, `jp/`, `zh/`, `es/`, ...):
`index.md`, `overview.md`, `roles.md`, `quickstart.md`, `protocol.md`. Each page's front matter:

```yaml
---
layout: page
title: How it works        # translated title (shown in the page list)
permalink: /en/overview/   # /<path>/<ref>/ ; the index is /<path>/
lang: en                   # the language code from _data/languages.yml
ref: overview              # the same for every translation of a page
nav_order: 2               # the same for every translation of a page
---

{% include langnav.html %}
```

- `ref` links translations together: the language switcher and the `hreflang` tags find the other
  languages' versions of a page by it.
- `lang` selects the page list shown at the top, sets `<html lang>`, and (through `_data/languages.yml`) `dir`.
- Internal links point to the same language: `{{ '/es/overview/' | relative_url }}`.
- Keep code blocks, commands, file names, JSON fields, event kinds and URLs unchanged.
- The Japanese pages live under `jp/` (not `ja/`) because older links point there.

## Adding a language

1. Add an entry to `_data/languages.yml`: `code` (BCP 47, e.g. `pt-BR`), `path` (URL prefix, e.g. `pt`),
   `name` (the language's own name), `dir` (`ltr` or `rtl`) and a one-line `tagline` for the root chooser.
2. Copy `en/` to `<path>/`, translate, and set `lang`/`permalink` in each file.
3. Add the language's docs root to the "Languages" section of `llms.txt`.
4. Check locally with the same gems GitHub Pages uses, for example:
   `docker run --rm -v "$PWD":/site -w /site ruby:3.3 sh -c 'gem install github-pages --no-document && jekyll build'`
