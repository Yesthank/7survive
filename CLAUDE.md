# CLAUDE.md — AI Assistant Guide for 7반 서바이벌

## Project Overview

**7반 서바이벌 (Class 7 Survival)** is a browser-based HTML5 survival action game, version V16.1. It is a single-player, wave-based arcade game inspired by the "Vampire Survivors" genre, themed around a Korean high school setting. All in-game text is in Korean.

The player survives 10 minutes of increasingly difficult enemy waves, collecting weapons and passive items, leveling up, and evolving weapons through synergy combinations.

## Repository Structure

```
7survive/
├── index.html      # Main game file (~747 lines) — HTML structure, inline styles, and all JavaScript logic
├── style.css       # External stylesheet — advanced styles (animations, Glassmorphism, etc.)
├── character.png   # (Optional) Custom player character sprite loaded at runtime
└── CLAUDE.md       # This file
```

This is a **Browser-Native** application with zero build steps. Open `index.html` directly in a browser or serve the directory with any static HTTP server.

---

## 1. Tech Stack & Versions (기술 스택과 버전)

This project targets a **Browser-Native** environment — immediately runnable with no build process.

| Technology | Version / Source | Purpose |
|---|---|---|
| HTML5 | — | Page structure, semantic layout, UI overlays |
| CSS3 | — | Styling, animations, Glassmorphism, responsive layout |
| Vanilla JavaScript | ES6+ | All game logic, rendering, input handling — no frameworks |
| Tailwind CSS | v3.4.x (CDN Script) | Fast layout composition, consistent typography, responsive utilities |
| Lucide Icons | Latest (Development CDN) | Clean, modern stroke-based icons suited for data dashboards |
| Google Fonts | 'Inter' / 'JetBrains Mono' | High readability, professional tabular number support |
| Canvas 2D API | — | Game world rendering (entities, particles, effects) |
| LocalStorage | — | Save/load game state (key: `7ban_save_v16_1`) |

**No package.json, no npm, no bundler, no transpiler.**

---

## 2. Development Rules (개발 규칙)

These rules are **strictly enforced** for maintainability and design consistency.

### File Structure Separation Principle (파일 구조 분리 원칙)

- **`index.html`** — Structure and logic (HTML markup + JavaScript)
- **`style.css`** — Advanced styles (animations, Glassmorphism, custom gradients, complex visual effects)
- Simple utility styling via Tailwind classes inline; complex/custom CSS goes in `style.css`

### Design System: Slate & Indigo Theme

| Element | Specification |
|---|---|
| **Background** | Slate-900 (Dark/Deep) base — reduces eye strain |
| **Accent** | Indigo-500 ~ 600 — emphasizes data importance |
| **Text** | High-Contrast (Slate-50, Slate-300) — maximizes readability |
| **Contrast Ratio** | Minimum 4.5:1 for all text on dark backgrounds |

### Component Styling

- **Glassmorphism**: Use `backdrop-filter: blur()` with semi-transparent borders to create depth
- **Charts**: Implement with **Pure CSS/SVG only** — no external chart libraries (Chart.js, etc.) to optimize loading speed and maximize customization freedom
- **Hover Effects**: Card levitation effects with shadow transitions on hover

### Responsiveness

- **Desktop-First** design (dashboard-oriented), with media queries for graceful vertical stacking on mobile
- Information should stack elegantly on smaller screens without losing hierarchy

---

## 3. Development Workflow (개발 워크플로우)

### Step 1: Scaffolding & Layout
- Build skeleton with HTML5 semantic tags (`header`, `main`, `section`, `footer`)
- Connect Tailwind CDN and configure base color palette

### Step 2: Core UI Implementation
- Implement Sticky Sidebar/Header (fixed on scroll without obscuring content)
- Grid system for KPI cards: 4-column desktop → 1-column mobile

### Step 3: Visual Enhancement (style.css)
- Apply custom gradients and Glassmorphism effects (`backdrop-filter`)
- Implement card levitation on hover with shadow transitions

### Step 4: Data Visualization
- Zebra-striped tables for row differentiation and readability
- CSS Flexbox/Grid-based Bar Charts and SVG Line Charts

### Step 5: Final Polish
- Run Lucide icon rendering: `lucide.createIcons()`
- Verify GitHub Pages deployment path compatibility

---

## 4. Special Precautions (특별 주의사항)

### GitHub Pages Deployment Compatibility
- **Always use relative paths** (`./style.css`) — never absolute paths (`/style.css`)
- This prevents broken styles when deployed in subdirectories

### Cross-Origin Isolation (CORS)
- Use only trusted CDNs: `unpkg`, `jsdelivr`
- Prevents security issues from untrusted CDN sources

### Performance Optimization (No Bloat)
- **No heavy chart libraries** — use CSS/SVG for all visualizations
- Game loop runs at 60fps with up to 400 enemies — avoid expensive operations in update/render path

### Accessibility
- Slate & Indigo theme uses dark backgrounds — maintain **text contrast ratio ≥ 4.5:1**
- Ensure all data remains identifiable and readable

---

## 5. Architecture

### Game State Management

Global state objects defined at the top of the script:

- `GAME` — Core state: `state` (title/running/paused/gameover), `frame`, `time`, `score`, `freezeTimer`, `screenShake`
- `GAME_CONFIG` — Constants: `endTime` (600s), `baseHp` (130), `baseSpeed` (3.4), `maxEnemies` (400)
- `player` — Single `Player` instance with HP, inventory, stats, position

Entity arrays as flat lists:
- `enemies[]`, `items[]`, `projectiles[]`, `damageTexts[]`, `particles[]`

### Core Classes

- **`Player`** — HP, exp/level, inventory (weapons + passives), computed stats. Methods: `reset()`, `updateStats()`
- **`Enemy`** — Type-based stat setup. Types: `normal`, `fast`, `swarm`, `tank`, `boss_minsang`, `boss_taeho`, `boss_hanki`, `boss_final`. Methods: `setupStats()`, `update()`
- **`Particle`** — Visual effects with position, velocity, lifetime. Methods: `update()`, `draw()`

### Data System

All game content in the `DATABASE` object:

- `DATABASE.weapons` — 5 base + 5 evolution weapons (name, icon, type, desc, cd, baseDmg, perDmg)
- `DATABASE.passives` — 7 passive items with stat effects and synergy targets
- `DATABASE.evolution` — Maps base weapon → required passive for evolution

Wave progression: `WAVES[]` (10 entries, 0–600s) and `BOSS_SCHEDULE[]` (bosses at 3, 6, 9 min).

### Weapon System

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
- **`updateItems()`** — XP gem collection with magnet pull
- **`levelUp()`** — Random 3-card selection UI
- **`killEnemy()`** — Death handling, loot drops, boss chest drops
- **`saveGame()` / `loadGame()`** — LocalStorage persistence
- **Game loop** — `requestAnimationFrame`-based update/render cycle

### Input System

- **Keyboard:** WASD / Arrow keys
- **Mobile:** Virtual joystick (touch events)
- **Mouse:** Click-based UI interactions

### UI Layer Architecture (z-index)

1. `#gameCanvas` (1) — Game world
2. `#exp-container` (5) — XP bar
3. `#flash-layer` (50) — Screen flash effects
4. `#ui-layer` (100) — HUD (timer, level, kills, inventory, joystick)
5. `#title-screen`, `#levelup-modal`, `#chest-modal`, `#result-screen` (200+) — Modal overlays

---

## 6. Naming Conventions

| Category | Convention | Examples |
|---|---|---|
| Global constants | `UPPER_SNAKE_CASE` | `GAME_CONFIG`, `BOSS_SCHEDULE`, `WAVES`, `DATABASE` |
| Classes | `PascalCase` | `Player`, `Enemy`, `Particle` |
| Functions | `camelCase` | `startGame`, `spawnManager`, `killEnemy`, `levelUp` |
| DOM IDs | `kebab-case` | `game-canvas`, `exp-fill`, `ui-layer`, `levelup-modal` |
| CSS classes | `kebab-case` | `top-bar`, `save-btn`, `item-slot`, `type-badge` |

---

## 7. Common Modification Patterns

### Adding a New Weapon
1. Add entry to `DATABASE.weapons` with `name`, `icon`, `type: "weapon"`, `desc`, `cd`, `baseDmg`, `perDmg`
2. Add evolution entry to `DATABASE.weapons` with `type: "evo"`
3. Map weapon → passive in `DATABASE.evolution`
4. Add fire logic method in `WeaponSys`
5. Add weapon handling in game loop's weapon update section

### Adding a New Enemy Type
1. Add name string to `ENEMY_NAMES`
2. Add stat setup branch in `Enemy.setupStats()`
3. Add to `WAVES[]` or `BOSS_SCHEDULE[]` for spawn timing

### Adding a New Passive Item
1. Add entry to `DATABASE.passives` with `name`, `icon`, `type: "passive"`, `desc`, `valDesc`, optionally `synergyTarget`
2. Add stat computation in `Player.updateStats()`

### Adjusting Game Balance
- Enemy HP/damage/speed: `Enemy.setupStats()` or `WAVES[]`
- Weapon damage: `baseDmg`/`perDmg` in `DATABASE.weapons`
- Wave timing: `WAVES[]` entries (start/end, intervals, amounts)
- Boss timing: `BOSS_SCHEDULE[]` time values
- Player base stats: `GAME_CONFIG`

---

## 8. Important Notes

- Entirely client-side — no server component
- All user-facing text in Korean — maintain Korean for UI strings
- Emoji characters are used as game sprites — no image assets beyond optional `character.png`
- Targets both desktop (keyboard) and mobile (touch joystick) input
- Save system uses versioned LocalStorage key (`7ban_save_v16_1`) to prevent cross-version conflicts
