# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step required. Open directly or via a local server:

```bash
open index.html                # macOS — direct open
python3 -m http.server 8000    # then visit http://localhost:8000
```

## Architecture

Three files, no dependencies, no bundler:

- **`index.html`** — DOM structure: a 300×600 `<canvas id="board">` for the playfield, a 120×120 `<canvas id="next-canvas">` for the piece preview, a side panel with score/lines/level HUD, and a single `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro theme using CSS variables; `backdrop-filter: blur` on the overlay.
- **`game.js`** — All game logic (~300 lines, strict mode, no classes).

### `game.js` internals

**State** lives in module-level `let` variables: `board` (2D array `ROWS×COLS`, cells are `0` or a color index 1–7), `current`/`next` (piece objects `{type, shape, x, y}`), and timing vars (`lastTime`, `dropAccum`, `dropInterval`, `animId`).

**Key functions and their roles:**
- `collide(shape, ox, oy)` — boundary + overlap check; called before every move and rotation 
- `rotateCW(shape)` — transpose + reverse rows; pure, returns new matrix
- `tryRotate()` — applies `rotateCW` then tries wall-kick offsets `[0, -1, 1, -2, 2]`
- `clearLines()` — iterates bottom-up, splices full rows and unshifts empty ones; updates score/level/`dropInterval`
- `ghostY()` — projects `current.y` downward until collision; used for ghost rendering and `hardDrop`
- `loop(ts)` — `requestAnimationFrame` callback; accumulates `dropAccum` and calls `lockPiece()` or advances `current.y`
- `spawn()` — promotes `next` to `current`, generates new `next`; triggers `endGame()` if immediate collision
- `init()` — full reset; called on page load and restart button

**Rendering** uses two canvases. `draw()` clears, redraws the grid lines, the locked board, the ghost (via `ghostY`, `globalAlpha = 0.2`), then the active piece. `drawNext()` centers the preview shape in the 4×4 next-canvas grid.

**Scoring:** `LINE_SCORES = [0, 100, 300, 500, 800]` × level. Hard drop adds 2 pts/row, soft drop adds 1 pt/row. Level = `floor(lines / 10) + 1`. Drop speed = `max(100, 1000 − (level − 1) × 90)` ms.

## Tunable constants (top of `game.js`)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Update canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` / `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index 1–7 maps to piece types I–L |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points per 1–4 cleared lines |
