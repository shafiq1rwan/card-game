# CLAUDE.md — Dice Card Battle

3D dice-battler card game (player vs AI) in **single self-contained HTML files** using three.js r128 (UMD, from cdnjs CDN — the only external dependency; works over `file://`). No build step, no runtime asset loading: dice/coin/table art is drawn on 2D canvases at runtime, and in `explorer-style.html` the card faces composite embedded explorer-character artwork from `assets/card-character.png` (a 3×3 sprite sheet, embedded whole as one base64 JPEG data URI in `CARD_SHEET_SRC`, preloaded by `loadCardArt()` before `main()` runs) — a data URI is used because WebGL/canvas can't read local image files over `file://`. Regenerate the embed with `sips -s format jpeg -s formatOptions 80`. Cell rects were pixel-scanned: cols start at [26, 296, 566], rows at [35, 434, 829], each cell 257×384; that card canvas is 512×765 and its card plane is 2.7×4.03 to match. Each cell's art includes its own ornate frame and name plaque, so its `drawCard` only overlays the HP badge and a special/stats plaque; if the sheet fails to decode it falls back to emoji rendering. `assets/Tarot Cats/` (22 tarot-cat PNGs) is an unused earlier art set.

## Files

- **`index.html` — the canonical game** as of Jul 2026 (renamed from `cosmicon-style.html`): Honkai Star Rail "Cosmicon, Roll On!" pop-art style, modeled on the screenshots in `reference/` — blue/yellow arena, floating pixel cubes, Titan One display font, chip-style UI. **The user prefers this look; new features go here.**
- **`explorer-style.html` — preserved alternative skin** (formerly the canonical `index.html`): "explorer's guild" brass/parchment UI, Georgia serif + Cinzel, leather-desk table, embedded character sprite art. Keep it working, don't delete it without asking.
- Feature gap: per-card attack effects (fx kinds fireball/stone/slash/sparkle/poison/crystal, recolored per character), shield dome defense, turn-sweep banner, ivory-vs-crimson dice themes, numbered dice faces, and the embedded character card art exist **only in `explorer-style.html`** — candidates to port into `index.html` if asked. Both files have: all screens, rebalanced roster, Sudden Death, coin fixes, full mobile support, sound + tap-to-skip, hit-stop/ghost-bars/punch numbers, wipes/parallax/dust/sway/emotes, embedded display fonts.
- Card display names in `explorer-style.html` are explorer characters but card `id`s and all stats are identical in both files: dragon = Beatrix the Alchemist, golem = Alistair the Mystical Librarian, wolf = Zane the Celestial Navigator, fairy = Luna the Moongazer, serpent = Rowe the Chronomancer, tiger = Kai the Deep Diver. Sheet characters Silas, Elara, and Lyra are unused (roster expansion would need new stats + a balance re-simulation). `index.html` uses the monster names. Balance notes below refer to cards by `id`.

## Game rules (implemented)

Pick a card → coin flip decides first attacker → each round: attacker rolls 5 dice, keeps exactly `atkDice` of them (rerolls of selected dice allowed, `rerolls` chances), defender does the same with `defDice`; damage = `max(0, ATK − DEF)`; specials give bonuses when the *kept* dice meet the condition; roles swap each round until HP ≤ 0. **Sudden Death: from round 7, attacks gain +1 ATK per extra round** — exists to cap tank-mirror stalemates; do not remove without re-simulating balance.

## Architecture (same in both files)

One classic `<script>` (no modules) — everything is top-level, so test drivers can reach globals directly. Key pieces: `CARDS` data array (stats + `special` + `fx`) → promise-based tween engine driven by one rAF loop (`tween({dur, delay, ease, update})`) → async/await game flow (`main()` → screens → `playerStage`/`aiStage` → `resolveAttack`) → player input via `waitButton()`/`pressButton()` resolver pattern and raycast dice clicks. AI keeps its mathematically best combo (`bestCombo` enumerates 5-choose-N including special bonuses) and rerolls dice ≤ 3.

Mobile: `portrait = aspect < 0.9` switches camera preset and `cardPos()` (cards move from corners to top/bottom); CSS media queries at `max-width: 700px` and `max-height: 470px`; `touch-action` locked on canvas and buttons; safe-area insets on bottom/right UI.

Sound (both files): all SFX are WebAudio-synthesized in the `SFX` object — no audio files. Guarded by `sndOn` (persisted to localStorage key `dcb-sound`, toggled by `#btn-sound`) and try/catch so headless runs never break; audio unlocks on first pointerdown; menu idle rolls are silenced via the `idleActive` check inside `SFX.diceRoll`/`diceLand`. `skippableWait()` + the capture-phase pointerdown listener implement tap-to-skip on the turn sweep, resolve compare, coin result, and KO pauses.

Juice (both files): display fonts are embedded as base64 WOFF2 `@font-face` at the top of `<style>` (index = Titan One, explorer-style = Cinzel 700; both SIL OFL, sourced from jsdelivr @fontsource). Damaging hits trigger `hitStop(90)` — a freeze-frame implemented as a `freezeUntil` early-return at the top of `loop()`. Damage/BLOCKED popups use the `dmgPunch` scale-punch keyframes with text-stroke outlines. HP bars have a lagging "ghost" fill (`.ghost` layer with a slower, delayed width transition; `updatePanels` sets both widths). Screen changes use the `#wipe` diagonal panel (`wipeCover()`/`wipeReveal()`; reveal is a no-op unless covered). Living background: `dust` Points drift in `loop()`, pointer parallax via `parTX/parTY` lerped into the camera position, and idle card sway writes `rotation.z`/`rotation.x` every frame — sway is skipped when `hp <= 0` so it can't fight `deathAnim`'s pose (any new animation that tweens card rotation needs the same guard). Emote speech bubbles: `emoteFrom(pool, side)` with pools in `EMOTES`, triggered on attack/hit/block/special/sudden-death.

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
- Effects must remove themselves from the scene and dispose geometry/materials (shared `dotGeo` in explorer-style.html is never disposed).
- Dice face mapping: box materials `[+x,−x,+y,−y,+z,−z]` = faces `[3,4,1,6,2,5]` (opposites sum to 7); `quatForValue` yaw-randomizes, so numbered faces (explorer-style) need the underline under "6".
