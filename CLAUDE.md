# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio for Rithvika Tiruveedhula: a static site hosted on GitHub Pages from the `main` branch at https://rithvika7495.github.io/rithvikatportfolio/. Pushing to `main` deploys it.

There is no build step, package manager, linter or test suite. The whole site is one file, `index.html`, which holds the markup, a `<style>` block and a `<script>` block. Images and `CV-RithvikaT.pdf` sit at the repo root and are referenced by relative path; some filenames contain spaces, e.g. `TCSOptumera-Fine Grained.jpg`. The CV is updated by deleting and re-uploading the PDF under the same name.

## Running locally

```sh
python3 -m http.server 8000   # then open http://localhost:8000
```

Opening `index.html` directly with `file://` also works. External dependencies are Geist and Geist Mono from Google Fonts, and Font Awesome from cdnjs.

## Architecture of index.html

### Design
Dark only. The look blends two inspiration sites, sawad.framer.website and redoyanulhaque.me. All colours, fonts and the radius live as CSS custom properties on `:root`:

| Token | Value / use |
|---|---|
| `--bg` | `#0b080c` |
| `--surface`, `--surface-2` | card backgrounds |
| `--accent` | lavender `#c2a4ff` |
| `--muted`, `--dim` | secondary text |
| `--sans` | Geist |
| `--mono` | Geist Mono |
| `--radius` | shared corner radius |

Reuse these tokens rather than hard-coding colours.

### Layout
`.page` is a two-column grid:
- a sticky `<aside class="profile">` card holding the photo, contact details and CV button
- `<main>` holding the sections `#home`, `#about`, `#workexp`, `#projects`, `#publications` and `#contact`

Below 1024px the grid collapses to one column. Below 720px the pill nav shows icons only.

### Headings
Section headings use `<h2 class="big-title">First <span>Second</span></h2>`. The `span` renders as a muted second line.

### Project cards and modal
Each `.project-card` has two parts:
- **Visible summary:** `.project-arrow`, `.project-img`, `.project-cat`, `.project-title`, `.project-description`, `.project-tech` and `.project-github-link`
- **Hidden write-up:** a `.floating-window-text` div holding the full text

When a card is clicked, the script copies that div's `innerHTML` and the card title into the shared `#modal-backdrop` dialog. Clicks on `.project-github-link` are excluded. The modal's CSS (`.modal-body h3`, `ul`, `table` and so on) styles whatever the write-up contains.

To add a project, copy an existing card. The `data-modal-id` attribute is not used by the JS.

### Filtering
`.filter-btn[data-filter]` buttons toggle each card's `hidden` attribute based on the space-separated tokens in its `data-category`. Every card must also include `all`. The valid tokens are `work_at_tcs`, `capstone_projects`, `summer_internships` and `coursework_projects`.

### Script features
- a loader overlay, removed on `load` with a 1.5s fallback
- a typewriter rotating the `roles` array
- a scroll-spy that highlights the active nav link
- reveal-on-scroll, where JS adds `.reveal`/`.in` so the content stays visible without JS
- a `#bg-canvas` particle-network background

Motion respects `prefers-reduced-motion`.

### Stats
The hero stats (internships, projects, papers) are hard-coded. Update them when experience, project cards or publications change.
