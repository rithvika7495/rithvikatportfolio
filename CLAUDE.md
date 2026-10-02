# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal portfolio for Rithvika Tiruveedhula: a static site hosted on GitHub Pages from the `main` branch at https://rithvika7495.github.io/rithvikatportfolio/. Pushing to `main` deploys it.

There is no build step, package manager, linter or test suite. The whole site is one file, `index.html`, which holds the markup, a `<style>` block and a `<script>` block. Images and `CV-RithvikaT.pdf` sit at the repo root and are referenced by relative path; some filenames contain spaces, e.g. `TCSOptumera-Fine Grained.jpg`. The CV is updated by deleting and re-uploading the PDF under the same name.

## Running locally

```sh
python3 -m http.server 8000   # then open http://localhost:8000
```

External dependencies:
- Geist and Geist Mono from Google Fonts
- Font Awesome from cdnjs
- GSAP 3.12.5 and ScrollTrigger from cdnjs
- Lenis 1.1.13 (smooth scroll) from unpkg

When testing scroll motion, keep the browser tab visible. Hidden tabs pause `requestAnimationFrame`, which freezes Lenis and GSAP.

## Architecture of index.html

### Design
Dark only. The look blends two inspiration sites, sawad.framer.website and redoyanulhaque.me. All colours, fonts and sizes live as CSS custom properties on `:root`:

| Token | Value / use |
|---|---|
| `--bg` | `#0b080c` |
| `--accent` | lavender `#c2a4ff` |
| `--sans` | Geist |
| `--mono` | Geist Mono |
| `--photo-w` | width of the hero photo |

Reuse these tokens rather than hard-coding colours.

### Scroll-motion concept (Redoyan-style)
`#photo-stage` holds the portrait. It is the one element that stays fixed while the page moves around it. It only becomes `position: fixed` under `.has-motion` at widths of 1025px and above.

The script adds `.has-motion` to `<html>` only when GSAP and ScrollTrigger have loaded and the visitor hasn't asked for reduced motion. Without it, the page falls back to a plain static layout. Any CSS that depends on the motion must be scoped under `.has-motion`.

Desktop timelines live in `gsap.matchMedia('(min-width: 1025px)')`:

| Trigger | What moves |
|---|---|
| `#home` | the photo glides left (`x: -24vw`) while the hero text fades out |
| `#workexp` | the photo lifts and fades |
| `#projects` | the section is pinned and `.work-track` scrolls sideways; the pin length is the track's overflow width |

On every screen size:
- `.career` timelines scrub `.career-progress` (`scaleY`) and move `.career-dot` (`top`).
- `.big-title .ln` lines, `[data-reveal]` elements and `[data-count]` stats animate in as they enter the screen.

### Gotchas
- **Lenis class names:** Lenis rewrites `<html>`'s class list and strips anything matching `lenis*`, so never put "lenis" in your own class names. The script adds `smooth-js` when Lenis is active. Without that class, CSS `scroll-behavior: smooth` stays on and fights Lenis.
- **Jumping to a pinned section:** the pinned `#projects` element is `position: fixed` while active. Anchor jumps and the post-filter reset therefore target its `.pin-spacer` parent.
- **Height-changing content:** call `ScrollTrigger.refresh()` after anything that changes layout height. `showTab()` already does this.

### Hero layout
`.landing` is a three-column grid. Because the fixed photo leaves grid flow, `.landing-left`, `.photo-stage` and `.landing-right` are pinned to explicit columns 1, 2 and 3.

### Project cards and modal
Each `.project-card` inside `.work-track#projects-grid` has two parts:
- **Visible summary:** `.project-arrow`, `.project-img`, `.project-cat`, `.project-title`, `.project-description`, `.project-tech` and `.project-github-link`
- **Hidden write-up:** a `.floating-window-text` div holding the full text

When a card is clicked, the script copies that div's `innerHTML` into the shared `#modal-backdrop` and pauses Lenis while the modal is open.

To add a project, copy an existing card. Card numbers come from a CSS counter.

### Filtering
`.filter-btn[data-filter]` buttons toggle each card's `hidden` attribute based on the space-separated tokens in its `data-category`. Every card must also include `all`. The valid tokens are `work_at_tcs`, `capstone_projects`, `summer_internships` and `coursework_projects`.

### Stats
The About stats (internships, projects, papers) are hard-coded in `data-count`. Update them when content changes.
