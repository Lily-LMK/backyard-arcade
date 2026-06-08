# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML5 canvas web app that turns iNaturalist observation data into mini-games. No build system, no dependencies, no framework — open `index.html` directly in a browser.

To run: `open index.html` (or drag it into a browser). To test changes, reload the page.

## Data

`observations-745770.csv` is an iNaturalist export for user `lily_kumpe`. The app can also load data live from the iNaturalist API (research-grade, with photos). Key CSV columns used: `image_url`, `scientific_name`, `common_name`, `iconic_taxon_name`, `observed_on`, `place_guess`, `taxon_*` hierarchy fields.

## Architecture

Everything lives in one `<script>` block inside `index.html` (~4300 lines). Sections are marked with `// ---------- Name ----------` comments — use these as landmarks.

**Scene system** — `scene` is a string (`'home'|'quiz'|'match'|'night'|'invaders'|'snake'`). `goto(name)` transitions between scenes. Each scene has its own module object (`quiz`, `match`, `night`, `invaders`, `snake`) with `reset()` and a draw function called each frame.

**Render loop** — `requestAnimationFrame` loop in `// ---------- Main loop ----------`. Each frame: clear canvas, draw current scene, draw top bar, draw toasts, draw help modal. Canvas is DPR-aware (`dpr` variable, `setCanvasSize()` on resize).

**Data model** — `data` object holds:
- `data.rows` — array of CSV row objects
- `data.cols` — column name → index map (`getCol(row, name)` helper)
- `data.pool` — filtered indices (rows with valid photo URLs)
- `data.badImage` — Set of URLs that failed to load (skipped on retry)

**Image loading** — `loadImage(url)` returns a record `{img, ready, ok, src}` and caches in `imgCache` Map. `circleCache` (LRU, max 200) stores pre-rendered circle-clipped offscreen canvases to avoid per-frame clip operations.

**UI** — `ui.buttons` array populated each frame by `addButton(x,y,w,h,label,fn,opts)`. Buttons are hit-tested in `pointerDown()`. Pointer state in `ui.pointer`. Drag state in `ui.drag`.

**Persistence** — `storage` object wraps `localStorage` with JSON serialization. Stores last-used data, game scores, settings.

**Color palette** — `palette` maps scene names to hex colors. `sceneAccent(alpha)` returns the current scene's color at a given opacity.

## Keyboard shortcuts (don't break these)

`Escape` → home, `p` → pause, `?` → help, `f` → fullscreen.  
Scene-specific: `n`/`m` in quiz, `i`/`r` in invaders, `r` in snake, arrow keys in night walk.

## iNaturalist API loader

`// ---------- iNaturalist API loader ----------` fetches paginated observations via `https://api.inaturalist.org/v1/observations`. Supports user login or project slug. Builds the same `data` structure as CSV parsing.
