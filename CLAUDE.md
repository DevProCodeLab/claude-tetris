# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no build, lint, or test commands — the project has no tooling configured.

## Architecture

Three files, no modules/bundler: `index.html` loads `style.css` and `game.js` directly.

- **`index.html`** — DOM structure: the `#board` canvas (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), a side panel (score/lines/level/next-piece canvas/controls), and a shared `#overlay` used for both Pause and Game Over.
- **`style.css`** — dark/retro arcade styling only; no logic-relevant classes beyond `.hidden` toggling the overlay.
- **`game.js`** — all game logic in one file (~300 lines), built around a small set of global `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) mutated by top-level functions rather than a class/state object. Key pieces:
  - **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the piece that placed it. Pieces (`PIECES`) are square matrices of the same index scheme.
  - **Collision** (`collide`): checked against both board bounds and locked cells; reused for movement, rotation, ghost-piece projection, and spawn (game-over check).
  - **Rotation** (`rotateCW` + `tryRotate`): rotation is a matrix transpose+reverse; `tryRotate` applies wall-kick offsets `[0, -1, 1, -2, 2]` in order and keeps the first that doesn't collide.
  - **Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates delta time in `dropAccum` and advances the piece one row (or locks it) once `dropInterval` is exceeded. `dropInterval` shrinks as level increases (`max(100, 1000 - (level-1)*90)`).
  - **Locking/clearing** (`lockPiece` → `merge` → `clearLines` → `spawn`): full rows are spliced out and an empty row unshifted at the top; cleared-line count drives scoring (`LINE_SCORES`, multiplied by `level`) and level-up (every 10 lines).
  - **Ghost piece** (`ghostY`): projects `current` straight down until collision, drawn at `globalAlpha = 0.2` in `draw()`.
  - **Rendering**: `draw()` redraws the whole board canvas every frame (grid, locked cells, ghost, current piece); `drawNext()` redraws the separate next-piece canvas whenever `spawn()` runs.
  - **Input**: a single `keydown` listener drives movement/rotation/soft-drop/hard-drop/pause; Pause and Game Over reuse the same `#overlay` element, distinguished by title text.

When changing board dimensions or block size (`COLS`, `ROWS`, `BLOCK` in `game.js`), also update the `#board` canvas `width`/`height` attributes in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`) — they are not computed from JS.
