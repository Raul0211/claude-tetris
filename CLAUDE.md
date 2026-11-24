# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. UI text and the README are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder statically (recommended):

```bash
python -m http.server 8000   # then open http://localhost:8000
```

There are no automated tests; verify changes by playing in the browser.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`, `#next-canvas`, and a shared `#overlay` used for both PAUSA and GAME OVER), `style.css`, and `game.js` (all logic, loaded as a plain classic script, `'use strict'`, everything in global scope).

Key points in `game.js` that span multiple pieces:

- **Cell values double as color indices.** `PIECES[type]` matrices are filled with the piece's own type number (1–8; 8 is the non-standard "tuerca", a 3×3 ring with an empty center), which is also its index into `COLORS`. The board stores these same numbers (`0` = empty). Adding or reordering a piece means keeping `PIECES` and `COLORS` aligned; `randomPiece()` picks uniformly from `PIECES.length - 1` types, so it needs no change.
- **Game state is module-level `let` globals** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`), all reset by `init()`, which is also the restart-button handler.
- **Piece lifecycle:** `spawn()` promotes `next` → `current` and triggers `endGame()` if the new piece already collides. Locking always goes through `lockPiece()` → `merge()` → `clearLines()` → `spawn()` (used by gravity in `loop`, `softDrop`, and `hardDrop`).
- **Rotation** is `rotateCW` (matrix rotation) plus naive horizontal kicks `[0, -1, 1, -2, 2]` in `tryRotate` — not SRS.
- **Game loop** (`loop`) is `requestAnimationFrame`-driven; gravity uses `dropAccum` vs `dropInterval`. Pause/game over stop it via `cancelAnimationFrame(animId)`; unpausing resets `lastTime` and calls `loop` directly. `draw()` redraws the whole board every frame (grid, locked cells, ghost at `globalAlpha` 0.2, current piece). The next-piece preview is only redrawn in `spawn()`.
- **Scoring/levels** live in `clearLines()`: `LINE_SCORES[cleared] * level`, level = `floor(lines / 10) + 1`, `dropInterval = max(100, 1000 - (level - 1) * 90)`. Soft drop +1/row, hard drop +2/cell.

## GitHub workflows

All use `anthropics/claude-code-action@v1` with the `CLAUDE_CODE_OAUTH_TOKEN` secret (no API key): `claude.yml` (`@claude` mentions), `claude-code-review.yml` (PR review), and `claude-issue-triage.yml` (on issue opened / title-or-body edited: applies existing type/`area:*`/`priority:*` labels and posts a single diagnosis comment, found by the `<!-- claude-issue-diagnosis -->` marker and updated in place on later edits).

## Gotchas

- Canvas sizes in `index.html` are hardcoded: `#board` must be `COLS × BLOCK` by `ROWS × BLOCK` (300×600), and `#next-canvas` (120×120) assumes a 4×4 grid of 30px cells (`NB` in `drawNext`). Changing `COLS`/`ROWS`/`BLOCK` requires updating the HTML.
- The controls list in `index.html` and the controls table in `README.md` duplicate the key bindings in the `keydown` handler; update all three together.
