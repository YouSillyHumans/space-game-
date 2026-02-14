# CLAUDE.md - Space Survivor Arcade

## Project Overview

**Space Survivor Arcade** is a self-contained HTML5 Canvas arcade space shooter game. The entire game lives in a single file (`index.html`) with no external dependencies, no build system, and no package manager. It runs directly in any modern browser.

- **Language:** Vanilla JavaScript (no frameworks, no TypeScript)
- **Rendering:** HTML5 Canvas 2D API
- **Audio:** Web Audio API (all sounds synthesized at runtime)
- **Storage:** LocalStorage for high score persistence
- **Input:** Mouse (desktop) and Touch (mobile)
- **Total size:** ~1,060 lines in one file

## Repository Structure

```
/
├── index.html    # Complete game (HTML + CSS + JavaScript)
├── CLAUDE.md     # This file
└── .git/         # Git metadata
```

There is no build step, no test framework, no linter, and no CI/CD pipeline. To run the game, open `index.html` in a browser or serve it from any HTTP server.

## File Organization (index.html)

The single file is organized into clearly commented sections:

| Lines (approx) | Section | Description |
|---|---|---|
| 1-54 | HTML Structure | Canvas element, UI overlays (menu, game over), buttons |
| 7-29 | CSS | Dark space theme, neon cyan/magenta palette, responsive sizing |
| 61-87 | Canvas Setup | Dynamic DPR scaling, resize handler, localStorage init |
| 98-244 | Audio Engine | Web Audio synth: 12+ sound effects, dynamic background music |
| 249-388 | Narrator System | ZYX-9 AI narrator with portrait, typing animation, story arcs |
| 391-405 | Background | 5-layer parallax nebula clouds |
| 410-426 | Game State Init | `init()` - resets player, arrays, stars, wave counter |
| 431-446 | Utilities | `pts()` particle spawner, `ftx()` floating text, boss scaling |
| 451-493 | Wave Spawner | Progressive difficulty, boss every 5 waves, enemy selection |
| 498-514 | Shooting System | Bullet generation, spread shot, homing missiles, cooldowns |
| 519-727 | Update Loop | `update()` - physics, AI, collisions, powerups, state management |
| 730-982 | Rendering | `draw()` - all rendering: background, entities, HUD, narrator |
| 987-1012 | Input System | Mouse and touch event handlers |
| 1017-1048 | Game Flow | `startGame()`, `gameOver()`, button listeners, audio init |
| 1052-1057 | Main Loop | `requestAnimationFrame` loop: update -> draw at 60 FPS |

## Key Variable Naming Conventions

The codebase uses terse, abbreviated variable names throughout:

### Global Singletons
- `g` - main game state object
- `ctx` - canvas 2D rendering context
- `p` - player object (alias for `g.p`)
- `W`, `H` - canvas width/height
- `mouse`, `touch` - input state
- `playing` - game active flag

### Constants
- `ET` - enemy types dictionary (`drone`, `fighter`, `tank`, `swarm`, `sniper`)
- `PT` - powerup types dictionary (`rapidfire`, `spread`, `shield`, `nuke`, `homing`, `timeslow`)
- `BOSSES` - array of 4 boss definitions
- `NARRATOR` - narrator story data and state

### Common Abbreviations in Objects
- `sz` = size, `hp` = health, `mhp` = max HP
- `vx`/`vy` = velocity, `sc` = score, `col` = color
- `sr` = shoot rate, `cd` = cooldown, `inv` = invulnerability frames
- `pu` = powerup, `bul` = bullets, `eb` = enemy bullets
- `en` = enemies, `pt` = particles, `tx` = floating text
- `ct` = combo timer, `w` = wave, `t` = game tick counter
- `dpr` = device pixel ratio

## Game Architecture

### Main Loop

```
loop() -> if playing: update() -> draw() -> requestAnimationFrame(loop)
```

- `update()` returns `'dead'` when player loses all lives, triggering `gameOver()`
- Frame-based timing (no delta-time), targets 60 FPS via `requestAnimationFrame`

### Entity Management

All entities (bullets, enemies, particles, powerups, floating text) are stored in plain arrays on the `g` object and filtered each frame to remove dead/expired entities:

```js
g.bul = g.bul.filter(b => { /* move, collide, return alive */ });
g.en  = g.en.filter(e  => { /* AI, collide, return alive */ });
```

### Collision Detection

Simple distance-based (circle-circle) collision checks between:
- Player bullets <-> enemies
- Enemy bullets <-> player
- Enemies <-> player (contact damage)
- Powerups <-> player (pickup)

### Audio System

All sounds are synthesized using Web Audio oscillators and noise nodes - no audio files. The music dynamically shifts frequencies when a boss is present. Audio context must be resumed on first user interaction (browser autoplay policy).

## Game Content

### Enemy Types (5)
| Type | HP | Speed | Score | Notes |
|---|---|---|---|---|
| drone | 1 | 1.8 | 100 | Basic enemy |
| fighter | 2 | 1.3 | 200 | Moderate |
| tank | 5 | 0.7 | 500 | Slow, tough |
| swarm | 1 | 2.5 | 80 | Aggressive tracking |
| sniper | 2 | 0.5 | 350 | Fast shooter |

### Bosses (4, every 5th wave)
| Wave | Name | Title | HP | Score | Pattern |
|---|---|---|---|---|---|
| 5 | KRAGOTH | Void Sentinel | 45 | 5,000 | sweep |
| 10 | ZYLTHRA | Nebula Witch | 65 | 8,000 | zigzag |
| 15 | OMNIVEX | The Devourer | 95 | 12,000 | chase |
| 20 | VOID KING | Lord of the Swarm | 140 | 20,000 | all patterns |

Boss HP scales by +50% on repeat encounters (wave 25+ cycles back).

### Powerups (6)
| Type | Color | Duration (frames) | Effect |
|---|---|---|---|
| rapidfire | Yellow | 300 | Fire rate: 10 -> 4 frames |
| spread | Cyan | 400 | Fires 5 bullets in a fan |
| shield | Green | 500 | Blocks next hit |
| nuke | Red | Instant | Destroys all on-screen enemies |
| homing | Magenta | 350 | Auto-targeting missiles |
| timeslow | Blue | 250 | 0.4x game speed |

### Scoring
- Base: enemy score value x combo multiplier (max 10x)
- Combo builds on consecutive kills, resets after timeout
- High score saved to localStorage key `"hiscore"`

## Development Guidelines

### Making Changes

Since the entire game is one file, any change involves editing `index.html`. When modifying:

1. **Identify the correct section** using the section comments and the table above
2. **Preserve the abbreviated naming style** - use short variable names consistent with existing code
3. **Test in browser** - open `index.html` directly or via a local server
4. **Test both inputs** - verify mouse (desktop) and touch (mobile) behavior
5. **Check audio** - Web Audio requires user interaction to start; test sound effects and music

### Adding New Enemy Types

Add to the `ET` object with properties: `hp`, `sz`, `sp` (speed), `col` (color), `sc` (score), `sr` (shoot rate). Then update the wave spawner logic to include the new type at the appropriate wave threshold.

### Adding New Powerups

Add to the `PT` object with properties: `col` (color), `dur` (duration in frames). Then add handling logic in `update()` for activation and in `draw()` for the HUD timer display.

### Adding New Bosses

Add to the `BOSSES` array with: `name`, `title`, `hp`, `sp`, `sz`, `sc`, `pattern`. Add a new drawing case in `drawBossBody()` and optionally add narrator dialogue in the `NARRATOR` story data.

### Key Numbers / Magic Values
- 3 lives per game
- 90 frames invulnerability after damage
- 80 frames between waves
- 10 frames base fire cooldown (4 with rapidfire)
- 0.96 particle friction
- 0.4 time slow factor
- Boss health bar: 80px wide, scaled by `hp / mhp`

## Testing

There is no automated test suite. Testing is manual:

1. **Open `index.html`** in a browser
2. **Play through waves** to verify enemy spawning and difficulty scaling
3. **Collect each powerup type** and verify effects
4. **Reach wave 5** to test the first boss encounter
5. **Die and retry** to verify game over flow and high score persistence
6. **Resize the window** to check responsive canvas scaling
7. **Test on mobile** (or emulator) for touch input

## Browser Compatibility

Requires modern browsers with support for:
- HTML5 Canvas 2D
- Web Audio API
- LocalStorage
- `requestAnimationFrame`
- Touch events (for mobile)
- CSS `clamp()` function

Works in Chrome, Firefox, Safari, and Edge (all modern versions).
