# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone built with plain HTML5 Canvas and vanilla JavaScript (ES6+). No build step, no bundler, no dependencies, no package.json. Everything lives in three files: `index.html` (canvas + styles), `game.js` (all game logic), `favicon.svg`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test tooling in this repo — changes are verified by loading the page in a browser and playing.

## Architecture

`game.js` is a single file organized into clear sections (marked with `── Section ──` comments), executed top to bottom:

- **Input**: raw keyboard state is tracked in two objects — `keys` (held) and `justPressed` (edge-triggered, consumed via `pressed(code)`). `ArrowLeft/Right/Up` and `Space` are the only inputs.
- **Entity classes**: `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has its own `update(dt)` and `draw()`; there is no shared base class or entity manager. All movement wraps toroidally at the canvas edges via `wrap(v, max)`.
- **Asteroid sizing**: size is an integer 1 (small) to 3 (large), indexing parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` produces two asteroids one size smaller (size 1 splits into nothing).
- **Global mutable game state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` are module-level `let` bindings, not encapsulated in a class/object. `state` is one of `'playing' | 'dead' | 'gameover'` and gates behavior in `update()`.
- **Game flow functions**: `initGame()` (fresh run), `nextLevel()` (clears bullets/particles, respawns ship, adds more asteroids scaled by level), `killShip()` (handles death/respawn/game-over transitions), `spawnAsteroids(count)` (spawns large asteroids away from ship start position).
- **Main loop**: `update(dt)` then `draw()`, driven by `requestAnimationFrame`. `dt` is clamped to 0.05s max to avoid large jumps after tab-switch/lag. Collision detection is simple circle-distance checks (`dist`), done as O(n·m) nested loops (bullets×asteroids, ship×asteroids) — fine at this entity count, don't over-engineer if extending it.
- Coordinate system is fixed at `W = 800`, `H = 600` (matches the canvas element's width/height in `index.html`); nothing is currently responsive to viewport size.

## Adding features

New entity types should follow the existing class shape (`constructor`, `update(dt)`, `draw()`, a `dead` flag for cleanup) and be pushed into/filtered out of the relevant global array the same way `bullets`/`asteroids`/`particles` are. Note `README.md` describes power-ups and a "shooting star" asteroid variant as features, but these are not present in `game.js` — treat them as aspirational/future work, not existing behavior to preserve.
