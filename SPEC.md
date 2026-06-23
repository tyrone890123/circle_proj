# Circle Battle Arena — Spec

A static, single-file web page where two (later N) weapon-wielding circles auto-fight
using physics. Everything tunable lives in one config object **and** is editable live
via an on-screen panel. Hostable as-is on GitHub Pages.

---

## 1. Goal & constraints

- **One file:** `index.html` containing markup, `<canvas>`, `<style>`, `<script>`. No build step, no npm, no backend.
- **Config-driven:** a single `CONFIG` object is the only source of truth. The sim reads it; the editor panel writes it. Nothing else holds state.
- **Live-tunable:** speed, placement, damage, colors, weapons, etc. are editable while it runs (or on Reset, where mid-fight editing is meaningless).
- **Feel over realism:** "looks like the reference video" beats physically correct.
- **GitHub Pages:** push, enable Pages on branch root, done.

## 2. Non-goals

- No player-controlled movement. Only interaction is tuning + Start/Reset/reroll.
- No physics accuracy.
- No framework, no bundler, no server.

## 3. Decided defaults

These were open questions, now locked. All are revisitable — changing any is a small edit, not a rewrite.

| Decision | Chosen | Alternative (if you change your mind) |
|---|---|---|
| Editing UX | **Live editor panel** (HTML inputs beside canvas) | Code-only `CONFIG` editing |
| Knockback stat | **Cumulative sum** of all knockback applied | Escalating-over-time multiplier |
| Reroll (🎲) button | **Rerolls seed + auto-restarts** the fight | Loads seed only; user starts manually |
| Start position units | **Fractions of arena (0–1)**, resolution-independent | Raw pixels |
| Heal-on-hit | Per-fighter, capped at `startHP` | Allow overheal via a flag |

## 4. The config object

The single source of truth. Editor reads/writes this; sim only reads it.

```js
const CONFIG = {
  arena:   { width: 600, height: 740, wallThickness: 8, restitution: 0.98 },
  physics: { friction: 0.992, maxSpeed: 14, knockbackScale: 1.0 },
  match:   {
    startHP: 80,
    winOnZero: true,
    seed: null,        // null/blank = roll a fresh random seed on each Start
    lastSeed: null     // the seed actually used this run; written back so random runs are recoverable
  },
  fighters: [
    {
      name: "Light", color: "#f5d020",
      start: { x: 0.30, y: 0.20 },     // fractions of arena (0–1)
      velocity: { x: 4, y: 3 },         // initial only
      radius: 34,
      weapon: {
        type: "hammer",                 // preset key -> visual + (later) sprite
        length: 70, width: 24,          // tether length + hit size
        damage: 15, heal: 0,            // heal applies to SELF on hit
        knockback: 5700
      }
    },
    {
      name: "Plant", color: "#33cc33",
      start: { x: 0.60, y: 0.70 },
      velocity: { x: -3, y: -4 },
      radius: 34,
      weapon: { type: "blade", length: 70, width: 20, damage: 11, heal: 11, knockback: 5700 }
    }
  ]
};
```

`fighters` is an **array** so going from 2 to N fighters is additive — add an entry, the
panel renders another column. Build for 2 first; keep the structure.

## 5. Simulation rules

**Per fighter, each frame:**
1. `position += velocity`
2. `velocity *= physics.friction`; clamp magnitude to `maxSpeed`
3. Wall contact → reflect that axis, multiply by `restitution`
4. Weapon trails the circle: anchored at the circle, swung outward along the circle's
   velocity direction (a tethered point, not rigid). This produces the flail motion.

**Collision (weapon ↔ enemy circle), each frame:**
- Test enemy circle against the attacker's weapon hit region (circle-vs-capsule).
- On hit, **once per contact** (cooldown flag that resets when the bodies separate — prevents 60 hits/sec while overlapping):
  - enemy HP `-= weapon.damage`
  - self HP `+= weapon.heal`, capped at `startHP`
  - knockback: `enemyVelocity += contactNormal * weapon.knockback * knockbackScale * dt`
- Accumulate running totals (total damage per fighter, total knockback) for the readout.

**Win:** if `winOnZero` and a fighter's HP ≤ 0 → freeze sim, show winner + the seed that
produced the fight.

## 6. Determinism / seeding

- A small seeded PRNG (e.g. mulberry32, ~5 lines). **Iron rule: nothing in the sim calls
  `Math.random()` directly** — every random value (initial velocities, any jitter) pulls
  from the seeded generator. A single stray `Math.random()` silently breaks "same seed →
  same fight."
- Blank/null seed → roll a fresh random seed each Start (random every run — the default).
- A number in the seed field → that exact fight repeats.
- The seed actually used is written into `match.lastSeed` and shown in the field, so even a
  random run that looked great is recoverable and shareable.

## 7. Editor panel

Collapsible panel of plain HTML `<input>`s **beside** the canvas (not drawn on canvas).
Grouped to mirror `CONFIG`:

- **Arena:** width, height, wall restitution — sliders.
- **Physics:** friction, max speed, knockback scale — sliders.
- **Match:** start HP, win-on-zero toggle, seed (text field).
- **Per fighter (one column each):** name, color picker, start x/y, initial velocity x/y,
  radius; per-weapon: type dropdown, damage, heal, knockback, length, width.

**Apply timing:**
- Live where safe: damage, heal, colors, speeds, knockback, knockbackScale, friction.
- On **Reset** only: start position, initial velocity, start HP, seed (mid-fight changes
  are meaningless).

**Buttons:**
- **Start** — runs the sim from current config; no-op while already running.
- **Reset** — stops, restores fighters to start state, clears totals, holds at frame 0.
- **🎲 Reroll seed** — writes a new random seed and auto-restarts (decided default).
- **Export** — dumps current `CONFIG` to a JSON textbox (this is the save mechanism).
- **Import** — paste a `CONFIG` JSON back in to load a matchup.

No `localStorage` (unreliable in embedded/iframe contexts). Export-to-text is how you save
and share.

## 8. Rendering

- **Phase 1 (playable):** plain shapes — filled circles, HP as centered text, weapon as a
  line + rectangle/polygon. Stat readout as HTML text under the canvas.
- **Phase 2 (cosmetic, deferred):** swap weapon shapes for sprite PNGs via a
  `type → image URL` map in `/sprites/`. No engine change — purely a draw swap.

## 9. File layout

```
index.html      # everything
/sprites/       # optional, phase 2
SPEC.md
CLAUDE.md
README.md
```

## 10. Build order

1. Canvas + seeded PRNG wired in + one bouncing circle reading from `CONFIG`.
2. Two circles, walls, friction, HP text.
3. Tethered weapons.
4. Collision → damage / heal / knockback + per-contact cooldown.
5. Stat readout + win state.
6. Editor panel wired to `CONFIG` (live + Reset-gated edits); Start / Reset / 🎲 buttons.
7. Export / Import JSON.
8. *(Optional)* sprites; *(optional)* N>2 fighters; *(optional)* board obstacles (a second
   entity type — meaningful scope increase, deferred).

## 11. Known open items

- **Board obstacles / pickups** ("items on board" may have meant this) — not specced.
  Would be a second entity type. Confirm before building.
- **Sprite assets** — the reference uses pixel-art PNGs; not included. Phase 2.
- **Overheal** — currently capped at `startHP`. Add a flag if uncapped healing is wanted.
