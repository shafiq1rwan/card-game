# CLAUDE.md — Dice Card Battle

3D dice-battler card game (player vs AI) in **single self-contained HTML files** using three.js r128 (UMD, from cdnjs CDN — the only external dependency; works over `file://`). No build step, no assets: all art (card faces, dice, coin, table) is drawn on 2D canvases at runtime and used as `CanvasTexture`s.

## Files

- **`index.html` — the canonical game.** Dark navy felt-table style. The user explicitly prefers this style; do NOT restyle it toward the Cosmicon look.
- **`cosmicon-style.html` — alternative skin** (Honkai Star Rail "Cosmicon, Roll On!" pop-art look, modeled on the screenshots in `reference/`). Same rules, balance, and core logic. Keep it working, don't delete it without asking.
- Feature gap: per-card attack effects (fireball/boulder/claws/sparkles/poison/ice-shards), shield dome defense, turn-sweep banner, ivory-vs-crimson dice themes, numbered dice faces, and special-text-under-name card layout exist **only in `index.html`**. Both files have: all screens, rebalanced roster, Sudden Death, coin fixes, full mobile support.

## Game rules (implemented)

Pick a card → coin flip decides first attacker → each round: attacker rolls 5 dice, keeps exactly `atkDice` of them (rerolls of selected dice allowed, `rerolls` chances), defender does the same with `defDice`; damage = `max(0, ATK − DEF)`; specials give bonuses when the *kept* dice meet the condition; roles swap each round until HP ≤ 0. **Sudden Death: from round 7, attacks gain +1 ATK per extra round** — exists to cap tank-mirror stalemates; do not remove without re-simulating balance.

## Architecture (same in both files)

One classic `<script>` (no modules) — everything is top-level, so test drivers can reach globals directly. Key pieces: `CARDS` data array (stats + `special` + `fx`) → promise-based tween engine driven by one rAF loop (`tween({dur, delay, ease, update})`) → async/await game flow (`main()` → screens → `playerStage`/`aiStage` → `resolveAttack`) → player input via `waitButton()`/`pressButton()` resolver pattern and raycast dice clicks. AI keeps its mathematically best combo (`bestCombo` enumerates 5-choose-N including special bonuses) and rerolls dice ≤ 3.

Mobile: `portrait = aspect < 0.9` switches camera preset and `cardPos()` (cards move from corners to top/bottom); CSS media queries at `max-width: 700px` and `max-height: 470px`; `touch-action` locked on canvas and buttons; safe-area insets on bottom/right UI.

## Verifying changes (important — this works on the dev machine)

1. **Syntax:** extract the inline script and `node --check` it.
2. **Logic-only tests:** slice the script from start to the `TWEEN ENGINE` comment, `eval` it, export via `globalThis` (top-level `const` in an eval'd strict script doesn't leak otherwise).
3. **Headless e2e:** headless Edge **cannot reach the CDN** from the sandboxed shell (curl can) → build a test copy with three.min.js **inlined**. Under `--virtual-time-budget` rAF never fires → shim it *before* the game script: `window.requestAnimationFrame = cb => setTimeout(() => cb(performance.now()), 50)`. Inject `window.onerror`/`unhandledrejection` collectors and an auto-player interval that drives `pressButton`/`selection`/`bestCombo`/`refreshChooseUI`, write results into a DOM node, then run:
   `msedge --headless=new --disable-gpu --enable-unsafe-swiftshader --window-size=390,844 --virtual-time-budget=~400000 --dump-dom <file>` and grep the dump. A full game ≈ 2–4 min real time; use small windows (portrait 390×844 exercises the mobile code paths).
4. **FX/leak checks:** call effect functions directly from the driver and compare `scene.children.length` before/after (all effects must self-dispose).

## Balance

Tuned by Monte-Carlo simulating optimal-vs-optimal matchups in Node (600 games per pair; simulate `stageTotal` with the AI reroll policy + `bestCombo`). Targets: every matchup averages < 20 rounds, per-card overall win rates ~40–85%. **Re-run the simulation after any change to card stats, specials, or the damage formula.** Current roster identity: Golem/Serpent are the strong "easy" picks, Dragon/Tiger the challenge picks.

## Conventions

- Keep everything single-file; no frameworks, no build tooling.
- Shared logic changes (rules, balance, AI) should be ported to **both** HTML files; presentation can diverge.
- The user cares about mobile playability — after UI changes, re-check portrait phone layout.
- Effects must remove themselves from the scene and dispose geometry/materials (shared `dotGeo` in index.html is never disposed).
- Dice face mapping: box materials `[+x,−x,+y,−y,+z,−z]` = faces `[3,4,1,6,2,5]` (opposites sum to 7); `quatForValue` yaw-randomizes, so numbered faces need the underline under "6".
