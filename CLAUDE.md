# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
# then visit http://localhost:3000
```

There are no tests, no linter, and no package manager.

## Architecture

The entire game lives in two files: `index.html` (canvas setup + style) and `game.js` (all logic).

**Classes** — each has `update(dt)` and `draw()` methods:
- `Ship` — player-controlled, resets to center on respawn; `tryShoot()` returns new `Bullet` instances
- `Asteroid` — size 1/2/3 (small/medium/large); `split()` returns two size-1 children or empties on size 1
- `Bullet` — expires via `ttl`; `dead` flag triggers removal
- `Particle` — short-lived spark used for explosion effects; fades out via `ttl / life` alpha

**Constants** `RADII`, `SPEEDS`, `POINTS` are arrays indexed by asteroid size (`[0, small, medium, large]`).

**Global game state** — flat module-level variables: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`.

**Game states**: `'playing'` | `'dead'` | `'gameover'`. The `update()` function branches on `state` at the top; `'dead'` waits `deadTimer` seconds before returning to `'playing'`.

**Input** — two maps: `keys` (held) and `justPressed` (consumed once via `pressed()`). `pressed()` clears the flag on read so fire/restart only trigger once per keydown.

**Main loop** — `requestAnimationFrame` calls `loop(ts)`, which computes `dt = min(elapsed, 50ms)` to cap on tab-blur spikes, then calls `update(dt)` → `draw()`.

**Wrapping** — the `wrap(v, max)` utility gives toroidal (edge-wrap) movement for all entities.

## Controls

| Key | Action |
|-----|--------|
| `←` `→` | Rotate ship |
| `↑` | Thrust |
| `Space` | Shoot / restart |
