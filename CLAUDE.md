# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio for Rithvika Tiruveedhula: a static site hosted on GitHub Pages from the `main` branch at https://rithvika7495.github.io/rithvikatportfolio/. Pushing to `main` deploys it.

There is no build step, package manager, linter or test suite. The whole site is one file, `index.html`, which holds the markup, a `<style>` block and a `<script>` block. Images and `CV-RithvikaT.pdf` sit at the repo root and are referenced by relative path. Project card images are full-size PNG originals in `Images/`, converted to 1000px-wide JPGs at the root (`sips -s format jpeg -s formatOptions 82 --resampleWidth 1000`). They are light coral artwork, so each `.project-img` carries the `tinted` class, which multiplies the image over a deeper peach so it stands out from the cream page. The CV is updated by deleting and re-uploading the PDF under the same name.

## Running locally

```sh
python3 -m http.server 8000   # then open http://localhost:8000
```

External dependencies:
- Inter from Google Fonts
- Font Awesome from cdnjs
- GSAP 3.12.5 and ScrollTrigger from cdnjs
- Lenis 1.1.13 (smooth scroll) from unpkg

When testing scroll motion, keep the browser tab visible. Hidden tabs pause `requestAnimationFrame`, which freezes Lenis and GSAP.

## Architecture of index.html

### Design
Light only: a warm peach/cream page with a coral accent, modelled on a clean designer-portfolio template. It uses rounded cards (`--radius: 20px`), pill buttons, soft blurred shadows, frosted-glass panels and large coral glows (`.bg-glow`) behind the page. All colours, fonts and sizes live as CSS custom properties on `:root`:

| Token | Value / use |
|---|---|
| `--bg` | `#fdf4ef` peach cream |
| `--accent` | coral `#ef6f4c` (`--accent-hover`, `--accent-soft` for tints) |
| `--glow` | coral used by the background glows |
| `--sans` | Inter; headings use weight 400 with tight negative letter-spacing |
| `--radius`, `--radius-sm` | corner radius for cards and small panels |
| `--shadow`, `--shadow-sm` | soft, warm drop shadows |

Reuse these tokens rather than hard-coding colours.

### Page order
1. Sticky `.topbar`, which gets a frosted background (`.scrolled`) once the page scrolls
2. `#home` hero, with a circular photo, then the `.logos` "Highlights & Focus" strip
3. `#about`, with stats and core skill chips
4. `#workexp`, with `.xp-card` lists
5. `#publications`
6. `#projects`, the selected-work grid
7. `#hackathons`, reusing the `.xp-card` layout (result in the left column)
8. `#skills` ("Tech Stack"), a two-column grid of `.stack-card`s. Each card has a pastel `--tint-*` panel that shows all of its chips
9. `.quote-band`
10. `#contact`, the contact row plus the footer

### Motion
The script adds `.has-motion` to `<html>` only when GSAP and ScrollTrigger have loaded and the visitor hasn't asked for reduced motion. Motion is deliberately gentle:
- the hero fades up on load
- `.section-head`, `[data-reveal]`, chips, experience cards and papers fade up as they enter the screen
- `[data-count]` stats count up

Nothing is pinned or fixed.

### Gotchas
- **Lenis class names:** Lenis rewrites `<html>`'s class list and strips anything matching `lenis*`, so never put "lenis" in your own class names. The script adds `smooth-js` when Lenis is active. Without that class, CSS `scroll-behavior: smooth` stays on and fights Lenis.
- **Anchor jumps:** these are offset by the sticky header's height.
- **Height-changing content:** call `ScrollTrigger.refresh()` after anything that changes layout height. `showTab()` already does this.

### Project cards and modal
Each `.project-card` inside `.project-grid#projects-grid` (4 columns on wide screens, then 3, 2 and 1) has two parts:
- **Visible summary:** `.project-arrow`, `.project-img`, `.project-cat`, `.project-title`, `.project-description`, `.project-tech` and `.project-github-link`
  - `.project-cat` is positioned as a pill over the image.
  - `.project-tech` is kept in the markup but hidden.
  - The arrow and GitHub circles sit at the bottom right.
- **Hidden write-up:** a `.floating-window-text` div holding the full text

When a card is clicked, the script copies that div's `innerHTML` into the shared `#modal-backdrop` and pauses Lenis while the modal is open.

To add a project, copy an existing card.

### Filtering
`.filter-btn[data-filter]` buttons toggle each card's `hidden` attribute based on the space-separated tokens in its `data-category`. Every card must also include `all`. The valid tokens are `featured` (the two projects shown first), `work_at_tcs`, `capstone_projects`, `summer_internships` and `coursework_projects`.

### Stats
The About stats (internships, projects, papers) are hard-coded in `data-count`. Update them when content changes.
