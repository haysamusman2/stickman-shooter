# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

Open any `.html` file directly in a browser — there is no build step, server, or package manager.

```
start zombie_shooter.html   # Windows
start tictactoe.html
```

## Git workflow

Every meaningful change should be committed and pushed:

```bash
git add <file>
git commit -m "short imperative description"
git push
```

Remote: `https://github.com/haysamusman2/stickman-shooter` (main branch)

---

## zombie_shooter.html — architecture

### Coordinate system

The world is unbounded on the X axis. The player (`P.x`) is always centered on screen. `cameraX = P.x - CW/2` is recomputed every frame. `ws(wx)` converts a world X to a screen X for drawing.

### Game loop structure

`requestAnimationFrame` → `gameLoop(ts)` drives everything. It checks `gameState` (`'start'` | `'playing'` | `'dead'`) and calls update functions then draw functions in order:

```
updateInput → updateBullets → updateZombies → updateSpawner →
updatePowerups → updatePopups → updateHits → updateContact
```

Draw order: `drawBackground` → `drawPowerups` → `drawBullets` → zombies → `drawPlayer` → `drawNukeFlash` → `drawHUD` → `drawPopups`

Update functions must not draw; draw functions must not mutate state.

### Key state

| Variable | Purpose |
|---|---|
| `P` | Player object — position, timers, `gunLevel` (0–3), `rapidFireTimer`, `multiShotTimer` |
| `bullets[]` | `{x, y, vx, vy, age, damage}` — `vx`/`vy` support angled spread shots |
| `zombies[]` | `{x, y, speed, hp, dying, dyingTimer, walkCycle, dir}` |
| `powerups[]` | `{type, x, y, age}` — max one on the ground at a time |
| `popups[]` | Floating pickup text `{text, color, x, y, timer}` |
| `elapsed` | Seconds since game start; `tier = Math.floor(elapsed / 10)` drives wave scaling |

### Gun system

`GUN_LEVELS` (index = `P.gunLevel`) controls fire cooldown, bullet count per shot, and damage per bullet:

- 0 PISTOL: CD 0.22s, 1 bullet, 1 damage (3 hits to kill)
- 1 CARBINE: CD 0.16s, 1 bullet, 2 damage (2 hits to kill)
- 2 SMG: CD 0.13s, 2 bullets spread, 2 damage (2 hits to kill)
- 3 MINIGUN: CD 0.10s, 3 bullets spread, 3 damage (1 hit to kill)

`getGunCD()` divides the base cooldown by 2.5 when Rapid Fire is active. `SPREAD_ANGLES` maps bullet count → array of degree offsets from horizontal.

### Power-up system

At most **one power-up exists on the ground at a time** — `spawnPowerup()` returns early if `powerups.length > 0`. Sources: auto-spawn every 14 s, and 8% drop chance from zombie kills.

`POWERUP_DEFS` holds display config. `collectPowerup()` applies effects:
- `rapidfire` / `multishot`: set timer to 5 s (not additive)
- `medkit`: clears `P.contactTimer` instantly
- `nuke`: kills all non-dying zombies, sets `nukeFlash`
- `gunupgrade`: increments `P.gunLevel` (max 3); only enters the spawn pool at waves that are multiples of 7

---

## tictactoe.html — architecture

DOM-based (no canvas). `board` is a flat 9-element array (`null` | `'X'` | `'O'`). The bot uses full minimax (unbeatable). `mode` is `'2p'` or `'bot'`. Bot move is triggered with a 300 ms `setTimeout` for UX.
