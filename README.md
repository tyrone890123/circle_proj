# Circle Battle Arena

A single-file, static, browser-based auto-battler. Two weapon-wielding circles fight
using physics on an HTML canvas — seeded so the same seed always reproduces the same
fight. Pure vanilla JS + Canvas 2D, no build step, no dependencies, no backend.

**[Open `index.html` in a browser]** — that's the whole app.

## Features

- **Physics arena** — circles slide, bounce off the walls *and each other*, and never
  stall (a minimum-speed floor keeps the action going, like the reference).
- **Weapon motion toggle** — per fighter, each weapon either *follows the circle's
  direction* (a Verlet-simulated tethered flail that swings as the body moves and
  bounces) or *spins* at a constant, tunable angular speed.
- **Combat** — circle-vs-capsule hit detection, one hit per contact (re-arms on
  separation), damage, self-heal-on-hit (capped at start HP), and knockback along the
  contact normal.
- **HP bars + live readout** — each fighter shows an HP bar, numeric HP, current weapon
  damage, plus a stat panel (total damage, hits, knockback, speed).
- **Epic juice** — glow, motion trails, particle bursts, floating damage numbers,
  screen shake, and a winner overlay that shows the seed.
- **Smooth at any refresh rate** — physics runs at a fixed 60 Hz logical timestep
  while rendering happens every `requestAnimationFrame` (your display's native
  60/90/120 Hz) with interpolation, so motion is fluid and the fight runs at the
  same speed on every device.
- **Live editor panel** — plain HTML inputs beside the canvas, grouped to mirror
  `CONFIG`. Most edits apply live; start position / velocity / HP / seed apply on Reset.
  Includes display toggles to hide fighter names and the win-screen seed.
- **Deterministic seeding** — a seeded PRNG (mulberry32) drives all randomness. A number
  in the seed field repeats that exact fight; blank rolls a fresh seed each Start and
  writes it back so a good random run is shareable.
- **Export / Import JSON** — the save & share mechanism (no browser storage).

## Controls

- **▶ Start** — run the sim from the current config (no-op while running).
- **⟲ Reset** — stop, restore fighters to their start state, hold at frame 0.
- **🎲 Reroll** — write a new random seed and auto-restart.
- **Export / Import / Copy** — dump or load a `CONFIG` as JSON.

## Design

`CONFIG` is the single source of truth: the simulation only reads it, the editor only
writes it. See `SPEC.md` for the full design and `CLAUDE.md` for the working rules.

## Hosting

Push and enable GitHub Pages on the branch root — it's a static file.
