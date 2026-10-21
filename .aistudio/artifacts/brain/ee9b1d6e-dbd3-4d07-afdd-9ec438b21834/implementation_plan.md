# MazeShift — Advanced Procedural Puzzle Game

MazeShift is a high-performance, polished browser puzzle game where players navigate procedurally generated labyrinths, solve multi-tier spatial puzzle mechanics (keys, color-coded locked gates, teleporters, pressure switches, timed traps, and moving obstacles), race countdown timers, and climb local leaderboards.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> Architectural and design decisions confirmed with the user:
> - **Architecture**: Standalone self-contained `index.html` containing all HTML, CSS, and vanilla JavaScript for 100% offline and static deployment capability, while seamlessly supported by the local Vite dev server.
> - **Camera & Viewport**: Dynamic smooth-scrolling camera centered on the player for large labyrinths (25×25, 31×31) with an interactive mini-map overview radar in the HUD.
> - **Audio Engine**: Synthesized cyber-pulse SFX suite powered by the native browser Web Audio API (zero external audio file dependencies; unlocked upon initial user gesture).

---

## 1. Overview & Core Concept

- **What It Does**: MazeShift generates infinite unique, mathematically guaranteed solvable labyrinths across four distinct difficulty tiers (Easy, Normal, Hard, Nightmare). Players guide an avatar through complex mazes containing collectible keys, pressure switches, laser/spike traps with visual tell cycles, moving patrolling drones, and teleporters. Includes full fog-of-war exploration mode, a BFS pathfinding hint engine, seed-reproducible daily challenges, and persistent local achievements.
- **Target Audience / Persona**: Puzzle game fans, speedrunners, and casual gamers looking for snappy, responsive labyrinth navigation with tactical puzzle depth on desktop and mobile touch devices.
- **Key Value**: Delivers zero-latency, high-frame-rate Canvas 2D gameplay with deep procedural mechanics, rich cyber-pulse audio, and complete offline portability without external runtime dependencies.

---

## 2. User Experience & Visual Design

### Key User Flows
1. **Title Menu & Level Select**: Sleek futuristic entry screen with "Play", "Select Difficulty" (Easy, Normal, Hard, Nightmare), "Daily Challenge" (deterministic date seed), "How to Play" tutorial modal, "Settings", and "Local Leaderboard & Stats".
2. **Active Labyrinth Gameplay**:
   - Canvas 2D viewport with smooth sub-pixel camera interpolation centered on the glowing player avatar.
   - Dynamic Mini-Map radar in the corner showing overall maze structure and player coordinates.
   - Top HUD displaying: Countdown Timer / Stopwatch (Practice mode), Move Counter, Score, Current Level / Seed, Key Inventory slots, and Quick Action buttons (Pause `Esc`, Hint `H`, Restart `R`).
   - Touch controls for mobile: Responsive on-screen virtual D-pad and swipe detection, with prevent-default scroll handling.
3. **Puzzle Interaction Cycles**:
   - Picking up color-coded keys (Cyan, Amber, Violet) with sound cues and visual inventory feedback.
   - Stepping on pressure switches to invert gate barriers.
   - Stepping into teleporter portals to instantaneously jump to linked coordinates with warp particle effects.
   - Avoiding trap hazards timed to rhythmic charge/discharge animations.
   - Navigating around predictable moving obstacle patrols.
4. **Fog-of-War (Nightmare & Configurable Mode)**:
   - Dynamic radial visibility mask revealing explored tiles in dim memory tone and currently visible line-of-sight tiles in full luminance.
   - Reveal power-ups or toggle in practice settings.
5. **Hint System (BFS Pathfinding)**:
   - Calculates the shortest legal path to the exit (taking into account key/door possession state).
   - Reveals the immediate next step or draws an ethereal glowing breadcrumb trail, applying a fair score deduction.
6. **Victory & Game Over**:
   - Victory overlay showing completion time, moves, remaining time bonus, hint penalty, total score, new personal record badge, and "Next Level" / "New Maze".
   - Game-over overlay when time expires with "Retry" and "Return to Menu".

### Visual Identity & Theme
- **Color Palette & Discipline (60-30-10 Rule)**:
  - `60% Neutral Canvas`: Deep space navy and abyss blacks (`#070b14`, `#0d1322`, `#111827`).
  - `30% Structural Surfaces`: Dark slate corridors, frosted glass panels (`rgba(17, 24, 39, 0.85)`), crisp hairline borders (`rgba(56, 189, 248, 0.2)`).
  - `10% Vibrant Accents`: Neon Cyan (`#06b6d4`, `#22d3ee`), Electric Emerald (`#10b981`), Radiant Amber (`#f59e0b`), and Magenta Violet (`#a855f7`) for player, exit, keys, gates, and teleporters.
- **Typography & Layout**:
  - Display Font: Modern geometric sans (`Plus Jakarta Sans`, `Outfit`, `-apple-system, system-ui`).
  - HUD & Metric Figures: Monospace tabular numerals (`tabular-nums font-mono`) for precision timers and score readouts.
  - Zero-pill metadata: Level headers and stats separated by subtle typographic bullets (`·`).
  - Accessibility: Respects `prefers-reduced-motion` and provides toggleable high-contrast walls and reduced particles.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Self-Contained Index.html vs Multipage Bundler**
  - *Chosen Approach*: Deliver the complete game within `index.html` using clean modular JavaScript structures, Canvas 2D engine, and responsive CSS.
  - *Why*: Conforms strictly to user instructions, enables direct drag-and-drop / Vercel static deployment, and runs flawlessly offline while remaining compatible with Vite dev server.
- **Decision 2: Camera Viewport with Mini-Map**
  - *Chosen Approach*: Dynamic player-following camera with smooth lerping on larger mazes (25×25, 31×31), paired with a compact semi-transparent mini-map radar. On smaller grids (11×11), the camera naturally frames the full maze.
  - *Why*: Prevents microscopic tiles on high-density grids and mobile screens while retaining global orientation awareness.
- **Decision 3: Procedural Solvability Guarantee**
  - *Chosen Approach*: Randomized DFS with backtracking for maze corridors + reachability validation via BFS. Key placement occurs on accessible branches before locking gates to guarantee 100% solvable puzzles without soft-locks.
  - *Why*: Eliminates frustration from uncompletable runs.
- **Decision 4: Pure Web Audio Synthesis**
  - *Chosen Approach*: Custom procedural sound synthesis using AudioContext (oscillators, noise bursts, gain envelopes, filter sweeps).
  - *Why*: Eliminates 404s, CORS issues, external asset loading lags, and keeps the bundle 100% self-contained.

---

## 4. Technical Architecture & Data Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                          MAZESHIFT HUD                          │
│  [Timer: 01:24.8]  [Score: 1,450]  [Moves: 42]  [Keys: 🔑 🔑]   │
│  [Buttons: Hint (H) | Pause (Esc) | Restart (R) | Sound (M)]    │
└────────────────────────────────┬────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────┐
│                       CANVAS 2D VIEWPORT                        │
│  ┌─────────────────────────┐        ┌────────────────────────┐  │
│  │ World / Grid State      │        │ Dynamic Camera Lerp    │  │
│  │ - Corridors & Walls     │  ───>  │ - Smooth tracking      │  │
│  │ - Keys, Doors, Gates    │        │ - Viewport clipping    │  │
│  │ - Traps & Hazard Cycles │        │ - Mini-map projection  │  │
│  │ - Teleporters & Drones  │        │ - Particle burst pool  │  │
│  │ - Fog-of-War Mask       │        └────────────────────────┘  │
│  └─────────────────────────┘                                    │
└────────────────────────────────┬────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────┐
│                        ENGINE & STORAGE                         │
│  ┌───────────────────────┐  ┌────────────────────────────────┐  │
│  │ Maze Generator (DFS)  │  │ Audio Engine (Web Audio API)   │  │
│  │ Solver / Hint (BFS)   │  │ Synth blips, sweeps, chords    │  │
│  ├───────────────────────┤  ├────────────────────────────────┤  │
│  │ Input Manager         │  │ LocalStorage Service           │  │
│  │ Keyboard, Touch, DPad │  │ High scores, Daily, Settings   │  │
│  └───────────────────────┘  └────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### State Model
- **Game State**: `MENU | PLAYING | PAUSED | VICTORY | GAMEOVER`
- **Maze Configuration**:
  - `gridSize`: 11×11 (Easy), 17×17 (Normal), 25×25 (Hard), 31×31 (Nightmare)
  - `timeLimit`: Generous (Easy: 120s), Standard (Normal: 90s), Tight (Hard: 75s), Strict (Nightmare: 60s), or Infinite (Practice)
  - `fogOfWar`: Configurable (Default ON in Nightmare)
  - `hazards`: Keys & Doors, Switches, Teleporters, Periodic Traps, Moving Obstacles
- **Daily Challenge**: Seed derived from `YYYY-MM-DD` integer hashing for 100% uniform daily runs across players.
- **Persistence Schema (`localStorage`)**:
  - `mazeshift_records`: Personal best time & score per difficulty
  - `mazeshift_achievements`: Set of unlocked achievement IDs (e.g., First Run, Speed Demon, Key Master, Maze Veteran, Night Owl)
  - `mazeshift_settings`: Sound volume, SFX mute, reduced motion, fog-of-war override, virtual D-pad toggle.

---

## 5. Verification Plan

1. **Maze Generation**: Confirm procedural DFS generates connected mazes with start `(1,1)` and end `(N-2, N-2)`. Automatic BFS reachability check verifies solvability before gameplay starts.
2. **Mechanics Test**:
   - Verify walls block movement without clipping.
   - Verify doors block passage until matching colored key is gathered.
   - Verify pressure switch toggles target barrier.
   - Verify teleporter moves avatar to counterpart coordinates.
   - Verify trap cycle triggers reset or penalty if stepped on while active.
3. **Viewport & Camera**:
   - Small maze: perfectly centered.
   - Large maze: player stays centered with smooth camera boundaries.
   - Mini-map accurately displays relative position.
4. **Time & Scoring**: Countdown decrements accurately, pauses during pause menu, triggers Game Over at zero, computes score based on speed, moves, and key bonuses.
5. **Persistence**: Reload test verifies stored high scores and settings remain intact.
6. **Responsive Input**: Test Arrow/WASD keys, prevent page scroll, test on-screen mobile D-pad and swipe gestures.
7. **Build & Lint**: Verify app builds and runs cleanly via `compile_applet`.
