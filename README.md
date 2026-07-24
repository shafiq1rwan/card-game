# 🎴 Dice Card Battle

A 3D dice-battler card game in a **single HTML file**, built with [three.js](https://threejs.org/). Pick your favourite card, flip the coin, and out-roll an AI opponent until one side hits 0 HP.

No build step, no assets, no install — all card art, dice, coin, and board textures are generated at runtime on `<canvas>`.

## ▶ How to Run

**Desktop:** open `index.html` in any modern browser (double-click it). An internet connection is needed once per session for the three.js CDN script.

**Mobile:** serve the folder and open it from your phone on the same network:

```bash
npx serve .
# then browse to http://<your-pc-ip>:3000 on the phone
```

The game is fully touch-enabled and has a dedicated portrait layout — cards move to the top/bottom of the board and the HUD reflows so everything stays tappable.

## 🎮 How to Play

1. **Main lobby** → hit *Bring it on!* (the dice roll themselves while you decide; *Rules* has the full how-to).
2. **VS screen** — tap *Edit* to open the card collection, pick your card in the big preview, *Deploy*, then *Time to settle this!*
3. A 3D **coin flip** decides who attacks first (👑 you / 🤖 AI). Attacker and defender swap every round.
4. On your turn, **throw 5 dice — three six-sided cubes and two four-sided chips** — then pick exactly the number your card needs in the center dice picker (⚔️ ATK dice when attacking, 🛡️ DEF dice when defending). Their sum is your combat value.
5. Not happy? Select any dice and **↻ reroll** them instead — each card has 2–3 reroll chances per turn.
6. Land your card's **✨ special condition** with the dice you keep for a bonus.
7. After both sides confirm, the big **ATK vs DEF compare** shows the numbers (crown goes to the exchange winner) — **damage = ATK − DEF**.
8. Reduce the enemy's HP to 0 → **KO!** and the victory screen crowns the winner.

🔥 **Sudden Death:** from Round 7 onward every attack gains +1 ATK per extra round, so duels always reach a finish.

## 🃏 The Cards

| Card | HP | ⚔️ ATK dice | 🛡️ DEF dice | ↻ Rerolls | ✨ Special |
|---|---|---|---|---|---|
| 🐉 Blaze Dragon | 32 | 3 | 2 | 2 | 3 of a kind → **+8 ATK** |
| 🗿 Stone Golem | 36 | 3 | 2 | 2 | Pair → **+6 DEF** |
| 🐺 Thunder Wolf | 28 | 3 | 2 | 3 | Keep a 6 → **+4 ATK** |
| 🧚 Mystic Fairy | 26 | 3 | 2 | 3 | Pair → **+3 ATK & DEF** |
| 🐍 Shadow Serpent | 24 | 4 | 2 | 3 | Odd total → **+3 ATK** |
| 🐯 Crystal Tiger | 30 | 3 | 2 | 2 | Even total → **+4 DEF** |

Stats were tuned by Monte-Carlo simulation (600 optimal-vs-optimal games per matchup): every matchup averages 4–16 rounds and every card can win.

## ✨ Features

- **3D arena** — rounded white stadium floor with red (AI) and blue (you) halves, a dark rim, and floating pixel cubes drifting around it — straight out of the reference screenshots
- **Animated everything** — tumbling dice with staggered bounces, coin flip tossed in with spin & settle, particles, camera shake, HP counting down live on the card art, death topple
- **Full screen flow** — lobby with idle dice, blue-vs-red VS screen, card collection with big preview, Match Start splash, red-vs-blue ATK/DEF compare with a 👑 on the exchange winner, a giant KO slam, and a victory screen with the loser's card in grayscale (plus a snarky quip)
- **Sound effects** — dice clatter, coin ping, whooshes, impacts, shield shatter, victory/defeat jingles — all synthesized live with WebAudio (no audio files); mute with the 🔊 button
- **Snappy pacing** — shorter pauses everywhere, and any banner/result overlay can be **tapped to skip**
- **Game-feel polish** — embedded display typography (Cinzel / Titan One, still single-file), a freeze-frame **hit-stop** on every damaging blow, outlined damage numbers that punch in with overshoot, HP bars that drain with a lagging ghost trail, and haptic vibration on hits (mobile)
- **Living presentation** — diagonal **screen wipes** between menus, drifting ambient dust over the board, pointer **parallax** on the camera, cards that idle-sway like they're breathing, and **emote speech bubbles** ("💢 OUCH!", "😎 Too slow!") as the cards react to hits, blocks, and perfect rolls
- **ATK vs DEF compare** — red-vs-blue number panel with a 👑 on the exchange winner
- **KO splash + Victory screen** — winner's card crowned, loser grayscaled with a snarky quip
- **Smart AI** — rerolls weak dice, chases its special condition, and always keeps its mathematically best dice combination (watch its picks glow red)
- **Mobile-ready** — portrait board layout, responsive HUD, touch-safe controls, notch-aware safe areas

## 📁 Files

| File | What it is |
|---|---|
| `index.html` | **The game** — "Cosmicon, Roll On!" pop-art style, modeled on the screenshots in `reference/` |
| `explorer-style.html` | Alternate "explorer's guild" skin (brass/parchment, embedded character art) with extra combat flair not yet in the main game: per-card signature attacks, shield dome defense, turn-sweep banners, color-coded numbered dice. Same rules & balance. |
| `reference/` | Visual reference screenshots used for the alternative skin |

## 🛠 Tech Notes

- Single classic `<script>` — no modules, no framework; three.js r128 (UMD) from cdnjs
- All art is procedural: card faces, dice, coin faces, and the arena are drawn on 2D canvases and used as `CanvasTexture`s (the explorer skin additionally embeds a character sprite sheet as a data URI)
- Promise-based tween engine drives every animation from one `requestAnimationFrame` loop
- Dice values are decided up front; the tumble animation converges exactly onto the rolled face's quaternion
- The AI evaluates all keep-combinations (5-choose-N) including special-condition bonuses before confirming
