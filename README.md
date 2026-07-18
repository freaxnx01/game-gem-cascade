# Handoff: Gem Cascade — match-3 browser game

## Overview
Gem Cascade is a complete, original match-3 tile-swapper (Bejeweled-style): swap adjacent gems to make lines of 3+, with special gems, cascade multipliers, level progression, synthesized sound, and saved progress. Single screen, works on desktop (mouse) and mobile (touch).

## Task for Claude Code: publish to GitHub Pages
`index.html` is a **self-contained, production-ready static file** (all JS, fonts, and runtime inlined — no build step, no server, no external requests needed). Publishing it is enough to ship the game.

Suggested steps:
1. Create a new public repo (e.g. `gem-cascade`): `gh repo create gem-cascade --public`
2. Put `index.html` at the repo root (keep the name `index.html`). Optionally add this README.
3. Commit and push to `main`.
4. Enable GitHub Pages, serving from `main` / root:
   `gh api repos/{owner}/gem-cascade/pages -X POST -f "source[branch]=main" -f "source[path]=/"`
   (or repo Settings → Pages → Deploy from branch → main / root)
5. Game will be live at `https://<owner>.github.io/gem-cascade/`

No framework port is required. Only recreate the game natively (React/canvas/etc.) if the user asks for a maintainable codebase — in that case use the spec below and `source/Gem Cascade.dc.html` as the reference implementation.

## About the files
- `index.html` — deployable standalone build. Do not hand-edit (compiled/inlined); it is the artifact to publish.
- `source/Gem Cascade.dc.html` — authoring source (a "Design Component" HTML format from the design tool). Treat as **readable reference** for all game logic, rendering, and styling; it contains the full template (HUD markup) and a `Component` class with the complete game implementation. It requires the design tool's runtime (`support.js`) so it won't run standalone as-is — that's what `index.html` is for.

## Fidelity
**High-fidelity and fully functional.** The game logic, visuals, sounds, and persistence are final. Any reimplementation should match behavior and look 1:1.

## Game spec (for reference or native reimplementation)

### Board & core loop
- 8×8 grid, 7 gem types (both tweakable; see props). Board generates with no pre-existing matches and at least one valid move.
- Swap two orthogonally adjacent gems (click-select then click neighbor, or drag/swipe ≥30% of a cell toward a neighbor). Invalid swaps animate there-and-back with an error buzz.
- Matches of 3+ in a row/column clear; gems above fall with gravity + landing squash; new gems spawn from above; cascades repeat until stable.
- Cascade multiplier: wave 1 = ×1, each subsequent wave +1. Base score 10/gem × multiplier. Floating score popups appear at each run's centroid (colored per gem).
- No fail state: if no valid moves remain, board auto-reshuffles (toast: "NO MOVES — RESHUFFLING"), preserving the gem multiset.

### Special gems
- **Match 4 → Flame gem** (keeps color, pulsing orange glow + bright core): when cleared, explodes 3×3. Creation bonus 100×multiplier.
- **L/T intersection → Lightning gem** (keeps color, white glow + horizontal/vertical gleam bars): when cleared, clears its full row + column. Bonus 150×multiplier.
- **Match 5 → Color cube** (colorless dark orb, rotating rainbow arcs, pulsing white core): swap with any gem to destroy all gems of that color (+250). Two cubes swapped = clear entire board (+1000). If destroyed passively (by explosion), clears a random color. Bonus on creation 300×multiplier.
- Specials created at the swap position (or run middle during cascades). Chain reactions: any special destroyed by another special triggers its own effect.

### Levels & scoring HUD
- Level target: `900 + (level−1)×600` points; progress bar fills, overflow carries; "LEVEL N" banner + arpeggio fanfare on level-up.
- HUD: SCORE (gold), BEST, LVL badge + progress bar, buttons HINT / ♪ ON-OFF / NEW.

### Helpers
- Hint: after 7s idle (or HINT button), a valid move pulses with white glows.
- Selection: white rounded-square glow, slight pulse.

### Sound (WebAudio, no files)
Synth only: select blip (520→660 triangle), swap (300→520 sine), invalid (180→110 saw), land thock, match chime rising through C-major pentatonic-ish notes `[523,587,659,784,880,988,1047,1175]` indexed by cascade wave, flame boom (bandpass noise 240Hz + 120→50 sine), lightning zap (noise 2800Hz + 1500→300 square), cube sweep (220→1500), level-up arpeggio, shuffle rattle. AudioContext created/resumed on first pointerdown. Mute persisted.

### Persistence
`localStorage` key `gem-cascade-save-v2`: `{v:2, n, types, score, best, level, levelScore, muted, grid:[[colorIdx, special|0]…]}`. Saved when the board settles / on new game / mute toggle / pagehide. Restored on load if dims match.

### Tweakable props (root component)
- `boardSize` int 6–10, default 8
- `gemTypes` int 5–8, default 7
- `particles` boolean, default true

## Design tokens
- Font: **Fredoka** (Google Fonts, 400–700), fallback system-ui/sans-serif.
- Page bg: `#0b0718` + `radial-gradient(120% 90% at 50% -10%, #2a1550, #160c30 45%, #0b0718)`.
- Text: `#e9e4ff`; muted labels `rgba(233,228,255,.55)`, 10px, letter-spacing 2px.
- Gold accent (score/title/NEW): `#ffd257`, title gradient `#ffe9a3 → #ffb84d → #ff8f3d`.
- Cyan accent (LVL, links): `#7fd7ff`; level bar gradient `#3ee0ff → #a06bff`.
- Panels: `rgba(255,255,255,.05)` bg, `1px solid rgba(255,255,255,.1)` border, radius 14px. Buttons: pill (999px), `rgba(255,255,255,.07)` bg, hover `.15`.
- Gem palette (indices 0–7): red `#ff3b5c`, orange `#ff9330`, yellow `#ffd42a`, green `#3ad463`, blue `#2fb4ff`, purple `#b465ff`, silver `#e9edf7`, pink `#ff5fd0`.
- Gem shapes by index: rounded square, pentagon, rhombus, hexagon, circle, triangle, fat 4-point star, octagon. Rendered on canvas: dark outline, radial gloss gradient (white→base→dark), inner rim light, bottom bounce light, dual specular ellipses + sparkle dots, ambient colored glow halo, periodic diagonal shine sweep.
- Board: canvas, rounded 18px panel `rgba(16,10,38,.6)`, checkered cells `rgba(255,255,255,.022/.05)`.

### Animation timings
Swap 160ms ease-out (swap-back 150ms), clear 230ms (scale/fade + white flash + particle burst), gravity `36×cellSize px/s²`, landing squash decay, reshuffle 200ms out/in, popups rise 0.95s, level banner 1.7s, hint pulse sine 6rad/s.

## Assets
None — everything is drawn programmatically (canvas) or synthesized (WebAudio). Only external dependency at design time was the Fredoka font, already inlined into `index.html`.
