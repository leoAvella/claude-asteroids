# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Asteroids clone in plain HTML5 Canvas + vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no build step, no tests, no linter. All game logic lives in `game.js`; `index.html` only hosts an 800×600 `<canvas id="canvas">` and loads the script.

## Running

Open `index.html` directly in a browser, or serve locally:

```bash
npx serve .   # then visit http://localhost:3000
```

Verify changes by playing in the browser (arrows to rotate/thrust, Space to shoot / restart after game over).

## Architecture (`game.js`)

- **Globals:** `canvas`, `ctx`, and the fixed world size `W`/`H` (800×600) are module-level and used directly by every class's `draw()`. If the canvas size changes in `index.html`, `W`/`H` must change too.
- **Input:** `keys[code]` holds held-down state (used for rotation/thrust); `pressed(code)` is a consume-once edge trigger via `justPressed` (used for Space — shooting and restart). Uses `KeyboardEvent.code` values (`'ArrowLeft'`, `'Space'`, …).
- **Entities:** `Bullet`, `Asteroid`, `Ship`, `Particle` classes, each with `update(dt)` and `draw()`. Entities are removed by setting `this.dead = true` and then filtering the global arrays (`bullets`, `asteroids`, `particles`). Positions of ship, bullets and asteroids are wrapped toroidally with `wrap()`; particles are not.
- **Asteroid sizes:** `size` is 3 (large) → 2 → 1 (small). `RADII`, `SPEEDS`, `POINTS` are arrays indexed by size (index 0 unused). `split()` returns two asteroids of `size - 1`.
- **Game state machine:** global `state` is `'playing' | 'dead' | 'gameover'`. `killShip()` decrements lives and goes to `'dead'` (2s `deadTimer`, then `ship.reset()`) or `'gameover'`. `update(dt)` branches on `state`; only `'playing'` runs ship input, bullets and collisions. `initGame()` resets everything; `nextLevel()` fires when `asteroids` is empty and spawns `3 + level` large asteroids away from the center (`SAFE_DIST`).
- **Collisions:** circle-distance checks in `update()` — bullet vs asteroid (spawns split fragments, score, particles), ship vs asteroid (skipped while `ship.invincible > 0`; asteroid radius scaled by 0.82 for forgiveness).
- **Loop:** `requestAnimationFrame(loop)` with `dt` in seconds, clamped to 0.05. All speeds are in px/s, so any new motion should be multiplied by `dt`.

## Conventions

- UI text, comments and README are in Spanish (e.g. HUD shows `NIVEL`, `PUNTAJE`); keep new user-facing strings and comments in Spanish.
- Code is organized into sections with `// ── Name ───` banner comments; add new systems as a new section in the same style.
- Visual style is monochrome white vector lines on black (`strokeStyle '#fff'`, `lineWidth ~1.5`), except the orange thruster flame.

## Notes

- The README's "Descripción del juego" still mentions power-ups and a shooting-star asteroid, but these were removed from the code (commit `13e713f`). Don't assume they exist.
- Demo is published via GitHub Pages: https://klerith.github.io/claude-asteroids/
