# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (watches Sass + Eleventy with live reload)
npm start

# Production build (minifies HTML/CSS/JS)
npm run build

# Individual steps
npm run get-theme        # Downloads theme CSS from THEME env var URL
npm run build:sass       # Compiles Sass → dist/styles/
npm run build:eleventy   # Builds static site → dist/
```

No test suite exists in this project.

## Architecture

This is "Deb's Hive Mind" — a static digital garden built with [Eleventy](https://www.11ty.dev/), designed to publish notes from an Obsidian vault via the [Digital Garden plugin](https://github.com/oleeskild/Obsidian-Digital-Garden).

### Data flow

1. **Obsidian plugin** pushes Markdown notes into `src/site/notes/`
2. **`npm run get-theme`** fetches the Obsidian theme CSS (URL from `THEME` env var) and writes it to `src/site/styles/_theme.<hash>.css`
3. **Eleventy** reads `src/site/` and outputs to `dist/`
4. **Sass** compiles `src/site/styles/` into `dist/styles/`

### Key configuration points

- **`.env`** — All site behaviour is driven by env vars (site name, theme URL, feature flags, Obsidian CSS settings). See existing `.env` for the full list.
- **`.eleventy.js`** — Core config: markdown-it pipeline, image optimisation transforms, callout/table/Mermaid/PlantUML rendering, HTML minification in prod.
- **`src/helpers/userSetup.js`** — Intentional extension point: add custom markdown-it plugins (`userMarkdownSetup`) or Eleventy plugins (`userEleventySetup`) here instead of touching `.eleventy.js`.
- **`src/helpers/userUtils.js`** — Add custom Eleventy computed data here via `userComputed`.
- **`src/site/_data/meta.js`** — Reads all env vars and exposes them as Eleventy global data (`meta.*`) available in every template.
- **`src/site/notes/notes.11tydata.js`** — Directory data file that assigns layouts and computes per-note `settings` from env vars, with per-note frontmatter able to override globals.

### Note settings system

Per-note feature flags (defined in `src/helpers/constants.js` as `ALL_NOTE_SETTINGS`) are resolved in `notes.11tydata.js`: the note's own frontmatter wins; otherwise the env var is used. Flags include `dgShowBacklinks`, `dgShowLocalGraph`, `dgShowFileTree`, `dgEnableSearch`, `dgShowToc`, `dgLinkPreview`, `dgShowTags`, etc.

### Markdown rendering pipeline (`.eleventy.js`)

markdown-it is configured with these extensions in order: anchors → mark → footnote → hashtag links → MathJax (inline `$…$`) → attrs → task checkboxes → PlantUML → named headings → user setup. The fence renderer additionally handles `mermaid`, `transclusion`, and `ad-*` (Obsidian admonition) code blocks.

Post-render HTML transforms (in order): dataview JS links → callout blocks → image `<picture>` elements → table wrappers → HTML minification (prod only).

### Internal link resolution

`src/helpers/linkUtils.js` resolves `[[wikilinks]]` to URLs by reading the target note's frontmatter from `src/site/notes/`. Broken links render as `.is-unresolved` anchors pointing to `/404`. The same file builds the graph data structure (`getGraph`) used by the interactive note graph.

### Layouts & templates

- `src/site/_includes/layouts/note.njk` — standard note layout
- `src/site/_includes/layouts/index.njk` — home/garden-entry layout (tag: `gardenEntry`)
- `src/site/_includes/components/` — Nunjucks partials for sidebar, search, graph, file tree, callout scripts, etc.

### Deployment

Configured for both Netlify (`netlify.toml`) and Vercel (`vercel.json`). Build command is `npm run build`; publish directory is `dist`.
