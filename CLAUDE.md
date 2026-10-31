# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris-style game in vanilla JavaScript on HTML5 Canvas. There are no dependencies, no `package.json`, no build step, no linter and no tests. The README, UI strings and code comments are in Spanish. Keep new user-facing text in Spanish to match.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically and visit `http://localhost:8000`:

```bash
python -m http.server 8000
# or: npx serve .
```

Verify changes by playing in the browser. There is no automated test suite.

## Architecture

There are three files. `index.html` holds the DOM, `style.css` the dark theme, and `game.js` all the logic, loaded as a classic (non-module) `'use strict'` script.

- **Global mutable state.** `game.js` keeps all state in top-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`). `init()` resets all of them and is also the restart handler, so any new state must be reset there.
- **Cell values are piece type IDs.** Each `PIECES[type]` matrix is filled with its own type number (1–7). `merge()` copies those numbers into `board`, and `drawBlock` uses them as indices into `COLORS`. Index 0 / `null` means empty. Adding a piece type therefore means adding entries at the same index in both `PIECES` and `COLORS`, and updating the `* 7` in `randomPiece()`.
- **Collision.** `collide(shape, x, y)` is the single gate for every move, rotation, ghost projection and spawn check. Rows above the board (`y < 0`) are allowed, but the side walls and the floor are not.
- **Rotation.** `rotateCW` (transpose plus reverse) is wrapped by `tryRotate`, which tries horizontal kicks `[0, -1, 1, -2, 2]`. These are simple kicks, not SRS.
- **Game loop.** `loop` is driven by `requestAnimationFrame`. It accumulates `dt` into `dropAccum` and applies gravity once it passes `dropInterval`. Locking always goes through `lockPiece()`, which runs `merge()`, then `clearLines()`, then `spawn()`. `spawn()` calls `endGame()` when the new piece collides immediately. Pause and game over stop the loop with `cancelAnimationFrame(animId)`.
- **Scoring and speed** are set in `clearLines()`, `softDrop()` and `hardDrop()`:
  - Each line clear scores `LINE_SCORES[n] * level`.
  - A soft drop scores +1 per row and a hard drop +2 per row.
  - Level is `floor(lines / 10) + 1`.
  - The drop interval is `max(100, 1000 - (level - 1) * 90)` ms.
- **The HUD** is updated with plain DOM text through `updateHUD()`. The pause and game-over screens share a single `#overlay`, whose title and score text are rewritten by `togglePause()` and `endGame()`.

## Coupled constants

`COLS`, `ROWS` and `BLOCK` in `game.js` must match the `<canvas id="board">` size in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`, currently 300×600). The next-piece canvas is 120×120. That size assumes a 4×4 preview grid at 30px, hard-coded as `NB = 30` and the `4` in `drawNext()`.
