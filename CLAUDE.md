# CLAUDE.md — 7반 서바이벌 (7survive)

This file provides guidance for AI assistants working on this repository.

---

## Project Overview

**7반 서바이벌** ("7th Grade Survival") is a browser-based roguelike survivor game with a school theme, written in Korean. It is a fully self-contained client-side application with no build process, no framework, and no external dependencies beyond Google Fonts.

- **Current version:** V16.2 (balance patch)
- **Language:** Korean (UI text, item names, enemy names)
- **Save key:** `localStorage["7ban_save_v16_2"]`

---

## Repository Structure

```
7survive/
├── index.html      # Entire game (HTML + inline CSS + all JavaScript, ~871 lines)
├── style.css       # External stylesheet (~187 lines)
└── CLAUDE.md       # This file
```

There are only two source files. All game logic lives inside `index.html` as inline `<script>` tags.

---

## Tech Stack

| Concern          | Technology                          |
|------------------|-------------------------------------|
| Rendering        | HTML5 Canvas 2D API                 |
| Game logic       | Vanilla JavaScript (ES6+)           |
| Styling          | CSS3 + external `style.css`         |
| Persistence      | `localStorage`                      |
| Mobile input     | Touch Events API (virtual joystick) |
| Fonts            | Google Fonts (Black Han Sans, Noto Sans KR) |
| Build system     | None — open `index.html` in a browser |

There is no npm, no TypeScript, no bundler, no transpiler, and no server required.

---

## Running the Game

Open `index.html` directly in any modern browser. No installation or build step is needed.

For local development with live-reload you can use any static file server, e.g.:

```bash
npx serve .
# or
python3 -m http.server 8080
```

Mobile users can use the on-screen virtual joystick; desktop users use **WASD** or **Arrow keys**.

---

## Code Architecture (inside `index.html`)

### Constants & Configuration

| Variable        | Purpose |
|-----------------|---------|
| `GAME_CONFIG`   | Core game parameters: `endTime` (10 min), `baseHp` (160), `baseSpeed` (3.4), `maxEnemies` (250), `maxProjectiles` (60), `maxParticles` (80) |
| `WAVES`         | Array of 14 wave definitions mapping timestamps → enemy type & count |
| `BOSS_SCHEDULE` | 8 named bosses scheduled at specific seconds (180 s, 300 s, …) |
| `DATABASE`      | All item data: base weapons, evolved weapons, passive items |
| `ENEMY_NAMES`   | 19 enemy types with Korean display names |

### Classes

| Class      | Role |
|------------|------|
| `Particle` | Visual-effects particle (position, velocity, color, lifetime) |
| `Player`   | Player state: position, HP, EXP, level, inventory, derived stats |
| `Enemy`    | Enemy state: position, type, HP, speed, damage, knockback physics |

### Key Systems

**Weapon system (`WeaponSys`)** — four weapon archetypes:
- **Shooter** — timed projectile spray (e.g., `cane` → evolves to `bat`)
- **Aura** — continuous area damage around the player (e.g., `jacket` → `analysis`)
- **Orbital** — rotating hitbox orbiting the player (e.g., `hand` → `burning`)
- **Zone** — stationary damage zone placed near player (e.g., `pizza` → `pig`)

**Synergy / evolution** — each base weapon pairs with one passive item:
```
cane  (📏) + book    (📘) → bat      (🚀)
hand  (🖐️) + smoke   (🌫️) → burning  (👹)
jacket(💩) + glasses (👓) → analysis (🤯)
pizza (🍕) + burger  (🍔) → pig      (🐷)
```
Evolution is triggered when both items are in the inventory and the player reaches the required level.

**Passive items** — seven items that modify player stats:

| Item      | Effect |
|-----------|--------|
| `book`    | Cooldown −8% |
| `smoke`   | Area +10% |
| `glasses` | Magnet +25% |
| `burger`  | Max HP +30 |
| `protein` | Damage +8% |
| `slippers`| Speed +10% |
| `juice`   | HP recovery +0.7/sec |

**Save / Load** — game state is JSON-serialized to `localStorage` with the key `7ban_save_v16_2`. The save includes: game time, player stats, inventory, seen enemies, and boss-spawn flags. Increment the version suffix in this key whenever a save-breaking change is made.

**Game loop** — driven by `requestAnimationFrame`. Each frame:
1. Read input (keyboard/joystick)
2. Move player
3. Spawn enemies (per `WAVES` schedule)
4. Update enemy positions (chase player)
5. Fire weapons (per cooldowns)
6. Check collisions
7. Spawn/collect item drops
8. Update particles
9. Render canvas
10. Update DOM UI

**Game modes:**
- **Normal** — survive 10 minutes (600 seconds) to graduate
- **Endless** — unlocked after completing Normal; wave escalation continues indefinitely

---

## Style Conventions

### HTML / JavaScript (inside `index.html`)

- All game logic is written in a single `<script>` block at the bottom of `<body>`.
- Global variables and classes are declared at the top of the script.
- Constants (`GAME_CONFIG`, `WAVES`, `DATABASE`, …) are `const` objects defined before class definitions.
- The game uses emoji literals for item and enemy icons throughout.
- Korean string literals are used for all player-visible text.
- Anti-tamper guards block DevTools shortcuts (F12, Ctrl+Shift+I/J/C) and right-click.

### CSS (`style.css`)

- Mobile-first with `clamp()` for fluid font and spacing sizes.
- Dark theme: backgrounds `#111` – `#1f2937`, accent color `#fbbf24` (amber/gold).
- Responsive breakpoint at `480px` (`@media (max-width: 480px)`).
- Inventory slot border colors indicate item type: **red** = weapon, **blue** = passive, **gold** = evolution.
- Animation keyframes defined: `pulseText`, `pop`, `slideUp`.

---

## Making Changes

### Adding a New Weapon

1. Add an entry to `DATABASE.weapons` with fields: `name`, `icon`, `type` (`"weapon"` or `"evo"`), `cd`, `baseDmg`, `perDmg`.
2. If it evolves, add the pairing to `DATABASE.evolution` (`{ baseWeapon: "passiveItem" }`).
3. Implement the firing logic in `WeaponSys` (follow the pattern of an existing weapon of the same archetype).
4. Add a level-up card description in the card-generation logic.

### Adding a New Enemy

1. Add the enemy type string to `ENEMY_NAMES`.
2. Add spawn entries to `WAVES` at the desired timestamps.
3. Define the enemy stats (HP, speed, damage, radius, resist) in the enemy-spawn logic.
4. If it is a boss, add it to `BOSS_SCHEDULE`.

### Changing Balance

- Edit constants in `GAME_CONFIG` for global tuning.
- Edit per-item values in `DATABASE` for weapon/passive balance.
- Edit per-wave spawn counts and enemy stats directly in `WAVES` and the enemy-spawn function.
- **Important:** if a save-incompatible change is made (e.g., item renamed, stat structure altered), bump the localStorage key version (e.g., `7ban_save_v16_2` → `7ban_save_v17_0`) to avoid corrupted saves on reload.

### Modifying the UI

- Structural HTML elements (screens, modals, HUD panels) are in the `<body>` of `index.html`.
- Visual styling belongs in `style.css`.
- Dynamic DOM manipulation (showing/hiding screens, updating text) happens in the JavaScript section of `index.html`.

---

## What to Avoid

- **Do not introduce external JavaScript dependencies.** The game is intentionally zero-dependency.
- **Do not add a build step.** The file must remain openable directly in a browser.
- **Do not rename or restructure the localStorage key** without bumping the version suffix.
- **Do not remove the DevTools-blocking code** without explicit instruction — it is an intentional design choice.
- **Do not translate UI text to English** — the game is intentionally in Korean.

---

## No Automated Tests or CI

There are no automated tests, linters, or CI pipelines. Validation is manual:

1. Open `index.html` in a browser.
2. Verify the title screen loads.
3. Start a new game and confirm the canvas renders.
4. Test any feature you modified end-to-end in-game.

---

## Git Workflow

The repository uses a single `master` branch for stable releases. Feature/AI work is done on `claude/` prefixed branches. Commit messages should be in English and describe the change concisely (e.g., `Add poison passive item`, `Fix boss HP bar overflow on mobile`).
