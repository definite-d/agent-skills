---
name: specular
description: |-
  Build a zero-build viewer for a folder of markdown: grouped sidebar navigation, filter search,
  light/dark themes, syntax highlighting, copy buttons, a raw-source toggle, and scroll restore that
  only applies to reloads. Ships the finished viewer. Use when someone wants to read specs,
  ADRs, RFCs, prompts, runbooks, or notes in a decent UI, or asks for a "spec browser", "docs
  viewer", "markdown reader", "documentation portal", or "single file, no build". Skip it for public
  sites that need SEO, real routing, auth, or multi-user editing.

  Examples:
  - user: "make these specs readable" → copy the viewer, fill in the docs arrays, inline the libraries
  - user: "docs viewer with no build step" → one HTML file with libraries and font inlined
  - user: "I want it as a normal site folder" → split it into index.html, style.css, app.js, vendor/
  - user: "the viewer flashes white / sidebar collapses weird" → check the traps below
---

# Specular

The viewer is already built and styled: `references/viewer.html`. Copy it, point it at the
documents, add the libraries. Do not redesign it.

## 1. Ask one question

One file or folder?

- **One file (default).** A single HTML with marked, marked-highlight, highlight.js, and the font
  inlined. About 640KB, mostly the font.
- **Folder.** `index.html`, `style.css`, `app.js`, and `vendor/` for people who want to inspect or
  edit the parts.

Both come from the same file.

## 2. Edit four things

1. `<title>` and `<span class="brand">`.
2. `SPECS` and `PROMPTS`: one `{ num, name, file }` per document, paths relative to the viewer root.
3. Optional: the color tokens in `:root` and `.light`.
4. Optional: the sidebar group labels in the markup.

## 3. Libraries

The app script expects three UMD builds loaded before it: marked 12
(`window.marked.marked`), marked-highlight 2.1 (`window.markedHighlight.markedHighlight`), and the
highlight.js 11 common build (global `hljs`). Inter 4 variable is the webfont the CSS points at,
`./vendor/fonts/InterVariable.woff2`. Later versions work if they keep those globals.

One file: inline the three scripts where the `<!-- specular:libs -->` marker sits, embed the font
as base64, and drop the preload link. Folder: copy the files to `vendor/` and `vendor/fonts/`,
link them at the marker in the same order, and move the `<style>` block to `style.css` and the app
script to `app.js`.

## 4. Serve

Any static server works, for example `python3 -m http.server 8000`. There are no module imports,
and the viewer fetches with `cache: "no-cache"`, so revalidation is covered. Opening the file from
`file://` will not load documents.

## Traps

The viewer already handles these. If you rewrite one, keep the reason.

- **Sidebar collapse:** animate `width` on the aside and keep the grid at `auto 1fr`. `display: none`
  on a grid child auto-places the reader into the empty column.
- **Theme flash:** set the class from an inline `<head>` script, before the stylesheet. Doing it on
  `load` flashes.
- **Code colors:** keep the `--code-*` palette. Third-party hljs themes are tuned to another
  background and look faded here. Every token clears 4.5:1 against `--code-bg`.
- **Highlighting:** the common build only, `langPrefix: "hljs language-"`, no `highlightAuto()`.
  Unlabeled fences stay plain on purpose.
- **Markdown in markdown:** four backticks around the outer block, labelled `markdown`. CommonMark
  closes at the first fence of equal length, so three backticks truncate silently.
- **Scroll:** restore only on the initial load from `sessionStorage`, clear it on navigation, and
  null the current file before swapping `innerHTML`.
- **Stale documents:** fetch with `cache: "no-cache"`. Servers send `Last-Modified` without
  `Cache-Control`, so browsers otherwise serve an hour-old copy.
- **Motion:** nothing over 300ms, and respect `prefers-reduced-motion`.

## Ship

Open it in a browser: both themes, collapse the sidebar, search, copy a block, toggle the source
view, reload mid-document. That is the test.
