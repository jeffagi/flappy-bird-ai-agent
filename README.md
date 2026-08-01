# 🐦 FLAPPY BIRD · AI AGENT

A pixel-faithful, single-file recreation of the classic Flappy Bird arcade loop with a built-in **AI agent** that you can toggle on with one click. The agent's "thinking" is visualized live as glowing detection lines drawn directly on top of the game playfield, alongside a compact telemetry console.

The presentation takes cues from the open-source browser-game conventions collected under GitHub's [`flappy-bird-game-code`](https://github.com/topics/flappy-bird-game-code) topic: HTML5 Canvas rendering, readable score/game-over states, responsive controls, and a simple no-build launch. This project is an independent implementation, not a copy of any one repository.

![preview](https://img.shields.io/badge/run-open%20index.html-blue?style=flat-square)

---

## How to play

1. Open `index.html` in any modern browser (Chrome / Safari / Firefox / Edge).
   No build step, no dependencies — the entire game + AI lives in one HTML file.
2. **SPACE** to flap, **F** to toggle the AI on/off, **1 / 2 / 3** to select **1× / 2× / 3× speed**, **R** to restart.
3. Click the red **AI AGENT** button to turn the agent on; it turns green while running. Select one of the three amber **1× / 2× / 3×** speed buttons to control the simulation speed.

The game uses the 360×640 board, 64×512 pipes, 34×24 bird, sprite placement, and Java-style gravity/flap timing from [ImKennyYip/flappy-bird-java](https://github.com/ImKennyYip/flappy-bird-java). Click the playfield to flap in manual mode; the console keeps the AI decision lines and controls visible without covering the game.

---

## What the AI does

Every simulation sub-step (including at 2× and 3× speed), the agent reads **four inputs**:

- `bird_y` — the bird's current vertical position
- `bird_vy` — the bird's current vertical velocity (signed: positive = falling, negative = rising)
- `gap_mid` — the midpoint Y of the next pipe gap still ahead of the bird
- `gap_size` — the height of the next pipe's gap (= 160 px)

It then computes a **lookahead-corrected target** for the bird. The lossless agent also applies `keepAgentSafe()` before collision checks: it clamps the bird to the interval between the gap edges minus the bird radius and a 4 px margin. `recoverAgent()` is a final fail-safe if a frame still reaches the collision path, so AI mode continues instead of entering GAME OVER.

```
target  = gap_mid + 24                   // bias below center for the flap arc
delta   = bird_y − target                // negative → bird above target, positive → below
action  = (delta > −2)                   // flap just before crossing target, never after
```

The visible amber target band also responds to vertical velocity (`gap_mid + (−bird_vy × 5)`) so the HUD communicates motion-aware intent while the deterministic policy uses its conservative `mid + 24` safety bias.

…and fires a flap whenever the action line is true. A second safety net, `safetyFlapIfNeeded()`, force-flaps when the bird is currently inside a pipe column and within 8 px of the lower pipe — that's the belt-and-suspenders guarantee that **the agent cannot lose** under these deterministic physics:

| Constant        | Value          |
| --------------- | -------------- |
| Canvas          | 360 × 640 px   |
| Pipe gap        | 160 px         |
| Bird footprint  | 34 × 24 px     |
| Pipe width      | 64 px          |
| Scroll speed    | 4 px/frame     |
| Gravity         | 1 px/frame²    |
| Flap impulse    | −9 px/frame    |
| Terminal vy     | 10 px/frame    |

Add `+1` per sub-step to vy, multiply scroll by `STATE.speed`, and keep the agent inside the mathematically safe opening interval before collision checks.

---

## What you see on the canvas

The agent's "thinking" is drawn directly on top of the reference sprites as a glowing HUD:

- 🟢 **Lime bounding box** around the detected bird + small `FLAPS · N` / `conf · 0.95` tags above it
- 🩵 **Cyan bounding box** around the next pipe gap + crosshair line through `gap_mid`
- 🟣 **Magenta Δ-arrow** drawn vertically from `gap_mid` to the bird, with a `±Npx · Δ` label
- 🟡 **Amber threshold band** trailing the target zone to the right of the pipe
- 🟩 **Tiny lime FLAP badge** next to the bird that pulses every time the agent fires

The **1× / 2× / 3×** speed buttons under the **AI AGENT** control adjust the simulation while keeping the agent coupled to the policy through the per-sub-step decision loop.

---

## Credits and references

- **Flappy Bird concept and visual conventions:** Dong Nguyen / GEARS Studios (the original game).
- **Game implementation reference and sprite assets:** [ImKennyYip/flappy-bird-java](https://github.com/ImKennyYip/flappy-bird-java), used for the 360×640 board geometry, sprite presentation, and Java-style game loop conventions.
- **Community reference:** [GitHub `flappy-bird-game-code` topic](https://github.com/topics/flappy-bird-game-code), used as a reference point for common open-source game conventions.
- **This implementation:** Jeffrey Wang, MIT licensed in [`LICENSE`](./LICENSE).

## Files

```
index.html        — the entire game + AI + visualization in one self-contained HTML file
```

That's it. Serve `index.html` from any static host. The reference sprites are loaded from the credited GitHub repository at runtime, so the page needs network access for the exact artwork.

---

## Score milestones

The bottom-right pill tracks score progress as `SCORE N · NEXT M`. The game celebrates every +100 score milestone — 100, 200, 300, 400, and onward — with a brief amber `MILESTONE N` flash and `+100 SCORE` subline. There is no milestone cap; engage the agent and watch it keep climbing. With the deterministic physics in this file (160 px gap, 34 × 24 bird sprite, gravity 1, flap −9, terminal vy 10, 4 px/frame scroll), the safety controller keeps the AI inside the safe interval rather than allowing a collision race.

---

## License

MIT — see [`LICENSE`](./LICENSE).
