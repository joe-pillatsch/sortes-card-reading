# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

There is no build step. Open `index.html` directly in a browser (`open index.html` on macOS). All code — HTML, CSS, and JS — lives in that single file.

The app is hosted via GitHub Pages at https://sortes.joepillatsch.com (repo `joe-pillatsch/sortes-card-reading`). The `CNAME` file at the repo root sets that custom domain; pushing to `main` redeploys.

## Architecture

The entire app is `index.html`. There is no framework, no bundler, and no external dependencies. The card back image is an inline SVG data URI (~2.4 KB), so the file is self-contained apart from the 43 front-face images in `cards/`.

This is a plain static web app (intended to be hosted on a subdomain via GitHub Pages). It is **not** inscribed on Bitcoin Ordinals — the old `PREVIEW_MODE`/`inscriptionId`/`/content/` dual-mode has been removed; `imageUrl()` just returns the local file path.

### Data model

- **`CARDS`** — static array of 43 objects: `{ name, file }`. Index in this array is a card's permanent identity.
- **`deck`** — runtime array of `{ idx, reversed }` objects. `idx` points into `CARDS`; `reversed` is set at shuffle/cut time. Fisher-Yates shuffled.
- **`selected`** — array of positions within `deck` (not `CARDS` indices) for the up-to-6 chosen cards.

### Two-view flow

1. **Pile view** (`#pileView`): all 43 cards rendered face-down in an absolute-positioned responsive grid with deterministic per-slot jitter. Card size and column count are computed from `clientWidth` on every render and resize.
2. **Spread view** (`#spreadView`): shown automatically when 6 cards are selected. Displays the 6 chosen cards face-up (reversed cards are CSS `rotate(180deg)`), then shows a "New Reading" button to return to pile view.

`showPile()` must always be called *before* `renderPile()` so the container has a measurable `clientWidth`; the reset handler enforces this order.

### Fitting both views to one screen (desktop)

Both views are sized to fit the viewport on desktop with no vertical scroll, by measuring the actual surrounding chrome (header/controls/footer/etc.) at render time rather than hard-coding a reserve:

- **Pile view** (`renderPile`): the disc diameter is capped by `window.innerHeight − chromeH`, where `chromeH` = measured header + controls + hint + footer heights + a small constant.
- **Spread view** (`fitSpread`): on desktop (`>700px`) the 6 faces are a single non-wrapping row; `fitSpread()` sets the `--spread-card-w` CSS var to `min(width-fit, height-fit)` so all 6 fit horizontally *and* the row + names fit vertically. It's called from `onCardClick` right after `showSpread()` (the row must be visible to measure its width) and from the resize handler. Below `700px` it removes the var and CSS wraps to 2 columns (vertical scroll is acceptable on mobile).

`.spread-card .face` is `width: 100%` (sized by its parent + `aspect-ratio: 5/8`). Because the header height feeds these calculations, changing the logo size or header padding automatically reflows both views.

### Card assets

- `cards/` — 43 optimized WebP front images, ~700px wide, named in kebab-case after each card (e.g. `the-fire.webp`, `mother-moon.webp`). Total ~2.5 MB. These were down-sized/recompressed from the original 1350×2250 PNGs (which were ~27 MB and have been removed).
- `tarot-logo/` — all logo assets live here. `moon_gold.svg` is the gold-tinted moon/card mark the app uses for both the header logo and the favicon (`<link rel="icon">`); it was recolored from the black hand-built source `moon_handbuilt.svg`/`.png` in the same folder.

### Credits

The footer (`.credit`) credits the artist Ashleigh York, linking to her shop (ashleighyorkart.com) and Instagram.
