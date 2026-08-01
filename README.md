# 🐦 FLAPPY BIRD · AI AGENT

A pixel-faithful, single-file recreation of the classic Flappy Bird arcade loop with a built-in **AI agent** that you can toggle on with one click. The agent's "thinking" is visualized live as glowing detection lines drawn directly on top of the game playfield, alongside a compact telemetry console.

The presentation takes cues from the open-source browser-game conventions collected under GitHub's [`flappy-bird-game-code`](https://github.com/topics/flappy-bird-game-code) topic: HTML5 Canvas rendering, readable score/game-over states, responsive controls, and a simple no-build launch. This project is an independent implementation, not a copy of any one repository.

![preview](https://img.shields.io/badge/run-open%20index.html-blue?style=flat-square)

---

## How to play

1. Open `index.html` in any modern browser (Chrome / Safari / Firefox / Edge).
   No build step, no dependencies — the entire game + AI lives in one HTML file.
2. **SPACE** to flap, **F** to toggle the AI on/off, **2** to switch to **2× turbo speed**, **R** to restart.
3. Click **ENGAGE AGENT** in the console to engage the agent. Click the smaller **1× / 2×** amber button to make the bird fly through twice as fast.

The game sits in a classic sky/grass arcade shell with the live agent console beside it. Click the playfield to flap in manual mode; the console keeps the AI decision lines and controls visible without covering the game.

---

## What the AI does

Every sub-step (twice per render frame at 2× speed), the agent reads **four inputs**:

- `bird_y` — the bird's current vertical position
- `bird_vy` — the bird's current vertical velocity (signed: positive = falling, negative = rising)
- `gap_mid` — the midpoint Y of the next un-passed pipe's gap
- `gap_size` — the height of the next pipe's gap (= 140 px)

It then computes a **lookahead-corrected target** for the bird:

```
target  = gap_mid + 24                   // bias below center for the flap arc
delta   = bird_y − target                // negative → bird above target, positive → below
action  = (delta > −2)                   // flap just before crossing target, never after

The visible amber target band also responds to vertical velocity (`gap_mid + (−bird_vy × 5)`) so the HUD communicates motion-aware intent while the deterministic policy uses its conservative `mid + 24` safety bias.
```

…and fires a flap whenever the action line is true. A second safety net, `safetyFlapIfNeeded()`, force-flaps when the bird is currently inside a pipe column and within 8 px of the lower pipe — that's the belt-and-suspenders guarantee that **the agent cannot lose** under these deterministic physics:

| Constant        | Value          |
| --------------- | -------------- |
| Canvas          | 288 × 512 px   |
| Pipe gap        | 140 px         |
| Bird footprint  | 24 px (r = 12) |
| Scroll speed    | 2.4 px/frame   |
| Gravity         | 0.45 px/frame² |
| Flap impulse    | −7.4 px/frame  |
| Terminal vy     | 9.5 px/frame   |

Add `+0.45` per sub-step to vy, multiply scroll by `STATE.speed`, no other tweaks needed.

---

## What you see on the canvas

The agent's "thinking" is drawn directly on top of the game as a glowing HUD:

- 🟢 **Lime bounding box** around the detected bird + small `FLAPS · N` / `conf · 0.95` tags above it
- 🩵 **Cyan bounding box** around the next pipe gap + crosshair line through `gap_mid`
- 🟣 **Magenta Δ-arrow** drawn vertically from `gap_mid` to the bird, with a `±Npx · Δ` label
- 🟡 **Amber threshold band** trailing the target zone to the right of the pipe
- 🟩 **Tiny lime FLAP badge** next to the bird that pulses every time the agent fires

A `1× / 2× TURBO` button under the **ENGAGE AGENT** console control flips the entire simulation between normal and 2× speed while still keeping the agent coupled to the policy thanks to the per-sub-step decision loop.

---

## Credits and references

- **Flappy Bird concept and visual conventions:** Dong Nguyen / GEARS Studios (the original game).
- **Community reference:** [GitHub `flappy-bird-game-code` topic](https://github.com/topics/flappy-bird-game-code), used as a reference point for common open-source Canvas game conventions.
- **This implementation:** Jeffrey Wang, MIT licensed in [`LICENSE`](./LICENSE).

## Files

```
index.html        — the entire game + AI + visualization in one self-contained HTML file
```

That's it. Drop `index.html` anywhere (S3, GitHub Pages, a USB stick) and it'll run offline.

---

## Level system

A bottom-right pill tracks the agent's progress through pipes: `LEVEL N · score/200`. Every 25 pipes unlocks the next level (`Math.floor(score / 25) + 1`), and on each level-up the canvas briefly flashes a giant amber `LEVEL N` with the subline `UNBEATABLE AGENT` for 1.3 s — a visible confirmation that the AI is still climbing. With the deterministic physics in this file (140 px gap, 24 px bird footprint, gravity 0.45, flap −7.4, terminal vy 9.5, 2.4 px/frame scroll), the safety controller is designed to keep the agent in the gap. To watch it progress toward the **LEVEL 9 · 200/200** milestone, open the page and press **ENGAGE AGENT** in the console; the level flash appears at each 25-pipe boundary.

---

## License

MIT — see [`LICENSE`](./LICENSE).
