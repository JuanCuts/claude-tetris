# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris implementation in HTML5 Canvas + CSS + JavaScript. No dependencies, no build step, no package manager, no tests.

## Running

Open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test command — the three source files are loaded as-is by the browser.

## Architecture

Three files, no modules:

- `index.html` — DOM structure: the main `<canvas id="board">` (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), the `<canvas id="next-canvas">` preview, the HUD (score/lines/level), and the pause/game-over overlay.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, structured around a `requestAnimationFrame` loop:
  - Board state is a `ROWS × COLS` matrix (`board`) where each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
  - Pieces (`PIECES`) are defined as square matrices; `rotateCW` rotates via transpose + row-reverse.
  - `collide(shape, ox, oy)` is the single collision check used for movement, rotation, and spawn validation.
  - `tryRotate` implements basic wall kicks: after rotating, it tries offsets `[0, -1, 1, -2, 2]` until one doesn't collide.
  - `loop(ts)` accumulates elapsed time and drops the piece one row once `dropAccum >= dropInterval`; `dropInterval` shrinks as level increases (`max(100, 1000 - (level-1)*90)`).
  - `clearLines` scans bottom-up, splicing full rows out and unshifting empty rows in; scoring uses `LINE_SCORES = [0,100,300,500,800]` multiplied by `level`. Level increases every 10 lines.
  - Ghost piece (`ghostY`) projects the current piece straight down and is drawn at `globalAlpha = 0.2`.
  - Game over is triggered when a freshly spawned piece immediately collides (checked in `spawn()`).

When changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK`, `ROWS×BLOCK`).
