# Specular

Turn a folder of markdown into a viewer that opens instantly, works offline, and needs no build
step.

## When to Use

- A pile of specs, ADRs, RFCs, prompts, runbooks, or notes needs a readable UI.
- The user asks for a spec browser, docs viewer, markdown reader, or documentation portal with no
  framework.
- The result has to run from a static server on a machine with no network.

Skip it for public sites that need SEO, deep routing, auth, or multi-user editing.

## What It Produces

Grouped sidebar navigation, filter search with `Ctrl/⌘K`, light and dark themes, a collapsible
sidebar, syntax highlighting with a hand-tuned palette, copy buttons on code blocks, a raw-source
toggle, and scroll restore that applies to reloads only.

One file by default, or a folder with `index.html`, `style.css`, `app.js`, and `vendor/`.

## Structure

- `SKILL.md` - The question to ask, the four things to edit, the libraries and their globals, the
  traps, and how to ship it.
- `references/viewer.html` - The viewer. This is the spec; copy it rather than rebuilding it.
