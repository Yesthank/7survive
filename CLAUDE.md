# CLAUDE.md — AI Assistant Guide for 7반 서바이벌

## Project Overview

**7반 서바이벌 (Class 7 Survival)** is a browser-based HTML5 survival action game, version V16.1. It is a single-player, wave-based arcade game inspired by the "Vampire Survivors" genre, themed around a Korean high school setting. All in-game text is in Korean.

The player survives 10 minutes of increasingly difficult enemy waves, collecting weapons and passive items, leveling up, and evolving weapons through synergy combinations.

## Repository Structure

```
7survive/
├── index.html      # Main game file (~747 lines) — contains all HTML, inline CSS, and JavaScript
├── style.css       # External stylesheet (~51 lines) — base UI styling, animations, responsive layout
├── character.png   # (Optional) Custom player character sprite loaded at runtime
└── CLAUDE.md       # This file
```

This is a **monolithic single-page application** with zero build steps. The entire game logic, data, and rendering code lives inside a single `<script>` tag in `index.html`.

## Tech Stack

| Technology | Usage |
|---|---|
| HTML5 | Page structure, UI overlays (modals, HUD) |
| CSS3 | Styling, animations (`@keyframes`), responsive layout, z-index layering |
| Vanilla JavaScript | All game logic, rendering, input handling — no frameworks |
| Canvas 2D API | Game world rendering (entities, particles, effects) |
| LocalStorage | Save/load game state (key: `7ban_save_v16_1`) |
| Google Fonts | "Black Han Sans" (titles), "Noto Sans KR" (body text) |

**No external dependencies.** No package.json, no npm, no bundler, no transpiler.

## Build & Run

There is **no build step**. To run the game:

- Open `index.html` directly in a browser, or
- Serve the directory with any static HTTP server (e.g., `python3 -m http.server`)

## Testing

There are **no automated tests**. The project has no test framework, no test files, and no CI/CD pipeline. All testing is manual through browser play-testing.

## Linting & Formatting

There are **no linting or formatting tools** configured. No ESLint, Prettier, or similar. Code style is manually maintained.

## Architecture

### Game State Management

Global state objects defined at the top of the script:

- `GAME` — Core game state: `state` (title/running/paused/gameover), `frame`, `time`, `score`, `freezeTimer`, `screenShake`
- `GAME_CONFIG` — Constants: `endTime` (600s), `baseHp` (130), `baseSpeed` (3.4), `maxEnemies` (400)
- `player` — Single `Player` instance with HP, inventory, stats, position

Entity arrays managed as flat lists:
- `enemies[]`, `items[]`, `projectiles[]`, `damageTexts[]`, `particles[]`

### Core Classes

- **`Player`** — Player character with HP, exp/level, inventory (weapons + passives), computed stats. Methods: `reset()`, `updateStats()`
- **`Enemy`** — Enemy entities with type-based stat setup. Types: `normal`, `fast`, `swarm`, `tank`, `boss_minsang`, `boss_taeho`, `boss_hanki`, `boss_final`. Methods: `setupStats()`, `update()`
- **`Particle`** — Visual effect particles with position, velocity, lifetime. Methods: `update()`, `draw()`

### Data System

All game content is defined in a single `DATABASE` object:

- `DATABASE.weapons` — 5 base weapons + 5 evolution weapons, each with name, icon, type, description, cooldown, base damage, per-level damage
- `DATABASE.passives` — 7 passive items with stat effects and synergy targets
- `DATABASE.evolution` — Maps base weapons to their required passive for evolution (e.g., `cane` → `book`)

Wave progression is defined in `WAVES[]` (10 entries, 0–600 seconds) and `BOSS_SCHEDULE[]` (3 timed boss spawns at 3, 6, 9 minutes).

### Weapon System

Weapons are fired by a central `WeaponSys` object with type-specific methods:

| Fire Method | Weapon Type | Behavior |
|---|---|---|
| `fireShooter()` | Cane/Bat | Projectile-based, penetrating |
| `fireOrbital()` | Hand/Burning | Spinning orbits around player |
| `fireAura()` | Jacket/Analysis | Radial area damage + knockback |
| `fireZone()` | Pizza/Pig | Area placement, homing |
| `fireSoccer()` | Soccer/Dream | Bouncing projectiles |

Evolution requires weapon level 8 + paired passive level 1.

### Key Game Systems

- **`spawnManager()`** — Wave-based enemy spawning + boss scheduling
- **`updateItems()`** — XP gem collection with magnet pull mechanic
- **`levelUp()`** — Random 3-card selection UI (weapon or passive upgrades)
- **`killEnemy()`** — Death handling, loot drops, boss chest drops
- **`saveGame()` / `loadGame()`** — LocalStorage persistence
- **Game loop** — `requestAnimationFrame`-based update/render cycle

### Input System

- **Keyboard:** WASD or Arrow keys for movement
- **Mobile:** Virtual joystick (touch events: `touchstart`, `touchmove`, `touchend`)
- **Mouse:** Click-based UI interactions (menus, cards, buttons)

### UI Layer Architecture

Z-index layering (low to high):
1. `#gameCanvas` (z-index: 1) — Game world
2. `#exp-container` (z-index: 5) — XP bar
3. `#flash-layer` (z-index: 50) — Screen flash effects
4. `#ui-layer` (z-index: 100) — HUD (timer, level, kills, inventory, joystick)
5. `#title-screen`, `#levelup-modal`, `#chest-modal`, `#result-screen` (z-index: 200+) — Modal overlays

### Rendering Pipeline

1. Clear canvas and apply camera offset (centered on player)
2. Draw checkerboard background pattern
3. Render entities: items → enemies → projectiles → player
4. Render particles and floating damage text
5. Apply screen shake effect
6. Draw player invincibility flash

## Key Conventions

### Code Organization

- All JavaScript is in a single `<script>` block in `index.html` (starts around line 133)
- Inline `<style>` block in `<head>` supplements `style.css` for game-specific UI styles
- Data-driven design: game content is configured through `DATABASE`, `WAVES`, and `BOSS_SCHEDULE` objects rather than hardcoded in logic
- Korean comments (`// 데이터`, `// 적 등장 알림 배너`) are used throughout

### Naming Conventions

- Global constants: `UPPER_SNAKE_CASE` (`GAME_CONFIG`, `BOSS_SCHEDULE`, `WAVES`, `DATABASE`)
- Classes: `PascalCase` (`Player`, `Enemy`, `Particle`)
- Functions: `camelCase` (`startGame`, `spawnManager`, `killEnemy`, `levelUp`)
- DOM IDs: `kebab-case` (`game-canvas`, `exp-fill`, `ui-layer`, `levelup-modal`)
- CSS classes: `kebab-case` (`top-bar`, `save-btn`, `item-slot`, `type-badge`)

### Color Scheme

- Weapons: Red accent (`#f87171`)
- Passives: Blue accent (`#60a5fa`)
- Gold/highlight: `#fbbf24`
- UI background: Dark grays (`#1f2937`, `#111827`)
- HP bar: Green-to-red gradient based on percentage

### Save System

- LocalStorage key: `7ban_save_v16_1`
- Stores: game time, player stats, inventory, kill count, seen enemy types, boss spawn states
- Version-specific key prevents cross-version save conflicts

## Common Modification Patterns

### Adding a New Weapon

1. Add entry to `DATABASE.weapons` with `name`, `icon`, `type: "weapon"`, `desc`, `cd`, `baseDmg`, `perDmg`
2. Add evolution entry to `DATABASE.weapons` with `type: "evo"`
3. Map weapon → passive in `DATABASE.evolution`
4. Add fire logic method in `WeaponSys`
5. Add weapon handling in the game loop's weapon update section

### Adding a New Enemy Type

1. Add name string to `ENEMY_NAMES`
2. Add stat setup branch in `Enemy.setupStats()`
3. Add to `WAVES[]` or `BOSS_SCHEDULE[]` for spawn timing

### Adding a New Passive Item

1. Add entry to `DATABASE.passives` with `name`, `icon`, `type: "passive"`, `desc`, `valDesc`, optionally `synergyTarget`
2. Add stat computation in `Player.updateStats()`

### Adjusting Game Balance

- Enemy HP/damage/speed: Modify `Enemy.setupStats()` or `WAVES[]` configuration
- Weapon damage: Adjust `baseDmg`/`perDmg` in `DATABASE.weapons`
- Wave timing: Edit `WAVES[]` array entries (start/end times, intervals, amounts)
- Boss timing: Edit `BOSS_SCHEDULE[]` time values
- Player base stats: Modify `GAME_CONFIG`

## Important Notes

- The game is entirely client-side; there is no server component
- All text content is in Korean — maintain Korean for user-facing strings
- Emoji characters are used as game sprites (weapons, enemies, items) — no image assets beyond optional `character.png`
- The game targets both desktop (keyboard) and mobile (touch joystick) input
- Performance-critical: the game loop runs at 60fps with up to 400 enemies on screen — avoid expensive operations in the render/update path
