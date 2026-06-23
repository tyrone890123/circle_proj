# CLAUDE.md

Operating instructions for Claude Code when working in this repository. This is **not**
the spec — see `SPEC.md` for what to build. This file is how to behave while building it.

## What this project is

A single-file, static, browser-based auto-battler: two (later N) weapon-wielding circles
fight using physics on an HTML canvas. Pure vanilla JS, no build step, hosted on GitHub
Pages. Read `SPEC.md` before making changes.

## Hard constraints — do not violate

- **One file.** All HTML, CSS, and JS live in `index.html`. Do not split into modules, do
  not add a bundler, do not introduce a build step.
- **No dependencies.** No npm, no frameworks (no React/Vue/etc.), no CDN libraries. Vanilla
  JS and Canvas 2D only. If you think something needs a library, stop and ask first.
- **No backend.** Nothing that requires a server. The output must work as a static file
  opened directly or served by GitHub Pages.
- **`CONFIG` is the single source of truth.** The simulation reads from `CONFIG` only. The
  editor panel writes to `CONFIG` only. Do not create a second parallel state store, and do
  not let the sim hold its own copies of tunable values — read them live from `CONFIG`.
- **No `Math.random()` in the simulation.** Every random value must come from the seeded
  PRNG. A single stray `Math.random()` breaks reproducibility silently. If you add anything
  random, route it through the seeded generator.
- **No `localStorage` / `sessionStorage`.** Persistence is via Export/Import JSON only.
  Browser storage is unreliable in embedded contexts and is deliberately excluded.

## Conventions

- Plain shapes first, sprites later. Don't block playable behavior on art assets.
- Start positions are stored as fractions of the arena (0–1), not pixels. Keep them that way.
- Keep the editor panel as real HTML inputs beside the canvas — do not draw controls onto
  the canvas.
- Comments should explain *why*, not restate *what* the code does. Skip comments that just
  narrate the line below them.
- When fixing a bug, explain the cause, not just the patch.

## Decisions already made (don't relitigate without being asked)

These were chosen deliberately and are recorded in `SPEC.md §3`:

- Live editor panel (not code-only editing).
- Knockback stat is a cumulative sum (not escalating-over-time).
- The 🎲 reroll button rerolls the seed **and** auto-restarts.
- Heal-on-hit is capped at `startHP`.

If a change would reverse one of these, flag it and confirm first rather than silently
switching.

## When extending

- `fighters` is an array by design. Adding fighters means adding array entries and rendering
  another panel column — not rewriting the loop. Preserve that.
- Board obstacles / pickups are **not** in scope yet. They would be a second entity type.
  Do not add them speculatively; confirm first.

## Definition of done for a change

- Still a single `index.html` with no added dependencies or build step.
- Opens and runs by loading the file directly in a browser.
- `CONFIG` remains the only state; Export produces valid JSON that Import round-trips.
- Reproducibility holds: the same seed yields the same fight.
- No console errors on load or during a match.

## Testing

There is no test framework (and don't add one). Verify by opening `index.html` in a browser
and checking: a match runs, Start/Reset/🎲 behave, edits apply at the right time (live vs.
on-Reset per `SPEC.md §7`), and a fixed seed reproduces an identical fight.
