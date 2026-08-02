# 15 — Constants

**Status: draft, revision 2 — awaiting Bryan's review.** Vocabulary is fixed by
[`14-CANON.md`](14-CANON.md); this file puts numbers on it.

Every value is one of three kinds:

| Mark | Meaning |
|---|---|
| **Fixed** | Derived from geometry or a decision. Changing it changes the design. |
| **Anchor** | A judgment call everything else is measured against. Changing it rescales a ladder. |
| **Starting point** | A guess, expected to move once M0 or the balance engine exists. |

`data/constants.json` is generated from this file in Phase 2a.

---

## 1. The playfield

```
PLAYFIELD_WIDTH        = 1000    Fixed (D-024)
PLAYFIELD_HEIGHT_REF   = 2200    Fixed — 20:9, the reference device
PLAYFIELD_HEIGHT_MIN   = 2000    Fixed — 18:9, the design floor
READY_LINE_FROM_BOTTOM =  360    Anchor
SPAWN_MARGIN           =  300    Starting point — above the action zone
```

Width is fixed and height varies with the device. That is the correct way round: fixing
height would rescale the ship and every radius between phones, and balance would not
transfer.

| Aspect | Height at 1000 wide | |
|---|---|---|
| 16:9 | 1778 | Letterboxed — see below |
| 18:9 | 2000 | **Design floor** |
| 19.5:9 | 2167 | Very common |
| 20:9 | 2200 | **Reference** |
| 21:9 | 2333 | Extra sky, no downside |

**The ready line is 360 units from the bottom**, not a percentage — a percentage drifts
with aspect and puts the ship under the player's thumb on short screens.

### The action zone

Your rule, and it is the right one:

> **Baddies spawn above the visible area, but every entry and first action happens inside
> the action zone.** Nothing sits on a border. Traversing outside is fine.

```
ACTION_ZONE = the bottom 2000 units
```

So a baddie spawns at 2300, drifts down, and its first telegraph, first shot, or first
turn cannot fire until it is at or below 2000. On a 20:9 screen it has already been
visible for 100 units of travel by then; on the 18:9 floor it becomes visible exactly as
it acts.

This also gives `SPAWN_MARGIN` a real job: it is the runway a baddie needs to be moving at
speed before it becomes the player's problem, rather than materializing at the edge.

### 16:9 letterboxes — O-16 resolved

At 1778, a 16:9 screen is 222 units short of the floor. **Letterbox it** rather than
designing everything against 1778.

Designing the whole game against 16:9 would cost every modern player 10% of their vertical
space for a slice of the market that is now mostly pre-2017 devices and tablets.
Letterboxing costs thin bars on rare hardware and guarantees **identical gameplay
everywhere**, which matters more — a shorter action zone would change every timing and
every band, and balance measured on one device would not hold on another.

---

## 2. Entity sizes

```
VALRUNE_WIDTH    = 80    Anchor — 8% of screen width
BADDIE_MINION    = 64    Starting point
BADDIE_ELITE     = 90    Starting point
BADDIE_MINIBOSS  = 200   Starting point
BADDIE_BOSS      = 400   Starting point
PROJECTILE_WIDTH = 8     Starting point
```

Sanity check against your density targets: 30 minions at 64 units is 1,920 units of total
width, under three screen-widths, comfortable on a 2200-tall screen. A 75-baddie zerg is
roughly seven rows — dense but navigable. **Ship size and density targets are consistent.**

---

## 3. Radius bands

Regions **around** a point. Anchored on `r1 = 120` (D-024).

| Band | Units | % of width | Uses in your spreadsheets |
|---|---|---|---|
| `r_short` | 60 | 6% | 4 |
| `r1` | 120 | 12% | **51** |
| `r2` | 240 | 24% | 29 |
| `r3` | 400 | 40% | 7 |

**Four rungs is right — the authoring proves it.** I counted every radius reference across
your ability, attunement, and status tables. Nothing reaches for a fifth band, and the
distribution is heavily bottom-weighted.

**`r1` is the single most consequential number in the game.** Fifty-one abilities read
against it. Moving it from 120 to 150 silently buffs a fifth of your content. It should
change only deliberately, and the balance engine should report its sensitivity.

### What these actually hit

```
expected_hits = density × (π r² / playfield_area) × CLUSTERING
```

| Band | Playfield covered | At 30 baddies, uniform | At CLUSTERING 2.0 |
|---|---|---|---|
| `r_short` | 0.5% | 0.15 | 0.3 |
| `r1` | 2.1% | 0.6 | 1.2 |
| `r2` | 8.2% | 2.5 | 4.9 |
| `r3` | 22.8% | 6.9 | 13.7 |

That `r1` catches roughly one extra baddie is the correct read, and it is why chain
lightning specifies "the nearest baddie within r1" rather than "all baddies."

---

## 4. Distance bands

Reach **away from** a point. Respaced per your instruction: the old top rung becomes
`dis4`, with real rungs added beneath it.

| Band | Units | Derivation |
|---|---|---|
| `dis_short` | 60 | = `r_short` **Fixed** |
| `dis1` | 120 | = `r1` **Fixed** |
| `dis2` | 400 | Mid-screen |
| `dis3` | 800 | Long, but not maximum |
| `dis4` | 1600 | **Maximum — ready line to the top of the action zone** |
| `dis5` | 2400 | Beyond maximum — wrapped angled shots, the Expanse |

`dis4` keeps the physical meaning: 2000 minus 360 for the ready line leaves 1640, so
**1600 reaches the top of the action zone**, and your solar beam "out to distance_3 (max
distance)" lands exactly where you wrote it.

### What the existing content maps to

Your spreadsheets use three distance rungs, 36 references total:

| You wrote | References | Now means |
|---|---|---|
| `distance_1` | 8 | `dis1` |
| `distance_2` | 19 | `dis2` |
| `distance_3` | 9 | **`dis4`** |

Solar beam annotates `distance_3` as "(max distance)", so the rename matches your intent
rather than reinterpreting it. Phase 2 does the remap mechanically.

**`dis3` at 800 is currently unused** — that is the rung you asked for, sitting empty until
something needs it. Spectral Lance is the likely first customer: it fires to max distance
and pays bonus damage "beyond distance_2", and 800 is a better threshold for that than 400.

---

## 5. Width bands

| Band | Units | Definition | Uses |
|---|---|---|---|
| `w1` | 40 | Half the Valrune | 3 |
| `w2` | 80 | The full Valrune | 2 |
| `w3` | 240 | Two Valrunes plus half a Valrune each side | 0 |
| `w4` | 400 | Five Valrunes — CRYO's wave blast | 0 |

**You were right that `w0` was bad naming, and the fix is to delete it rather than
renumber.** A projectile's width is not a band anybody authors against — nobody writes
`width: w0`. It is a property of the projectile, so it lives in entity sizes as
`PROJECTILE_WIDTH` and the ladder starts at 1 like every other ladder.

`w4` is the CRYO answer: its wave blast trades reach for width, so it needs a rung above
the general-purpose ones rather than a special case.

---

## 6. Push, pull, and gravity

**Both are `(distance, time)`** — reverting my earlier split, because you are right that
the symmetry is real. One is displacement away from a source, the other toward it, and
**neither stops the target's own movement.** Each contributes a velocity that adds to
whatever the target was already doing.

**Gravity is the status that disables self-movement** (D-055). Only then does a pull become
the target's entire motion. Nothing arbitrates, because nothing competes.

| Band | Distance | Time | Effective | Feel |
|---|---|---|---|---|
| `push_1` | 40 | 0.08s | 500 u/s | The Forge knock — a stutter, not a launch |
| `push_2` | 100 | 0.12s | 833 u/s | A real shove |
| `push_3` | 220 | 0.20s | 1100 u/s | A detonation |
| `pull_1` | 80 | 1.0s | 80 u/s | Slowed but still advancing |
| `pull_2` | 200 | 1.0s | 200 u/s | Overpowered; it comes to you |
| `pull_3` | 450 | 1.0s | 450 u/s | Inescapable under gravity |

Push windows are short because a push is one impulse. Pull windows are 1.0s because a pull
repeats for as long as the status lasts, making the pair read as a rate.

**Read against a minion at 120 u/s:** `pull_1` at 80 slows an advancing baddie without
stopping it, which is your `misc_ideas` line 15 exactly.

**One thing to watch:** these are calibrated against *baddies*. Pulling the **Valrune** at
1000 u/s is a different scale entirely — `pull_3` would barely inconvenience the player.
Anything that pulls the player has to pair with gravity to matter at all.

`pull_stop_radius` defaults to `null` (all the way to the point); a value stops the pull at
that ring while gravity keeps holding the target.

---

## 7. Speeds

```
VALRUNE_SPEED_BASE   = 1000 u/s    Anchor — 1.0s to cross screen width
VALRUNE_SPEED_RANKS  = 20 × +40    → 1800 u/s at max
BADDIE_SPEED_MINION  =  120 u/s    Anchor — what pull bands read against
PROJECTILE_SPEED     = 2400 u/s    Starting point
PROJECTILE_RANKS     = 5 × +160    → 3200 u/s at max
ROTATION_BASE        =  420 °/s    Starting point
ROTATION_RANKS       = 20 × +15    → 720 °/s at max
```

**Speed ranks are now flat per rank, not percentages.** v0 had thrusters and gyros as
`+0.15× base` and velocity as `+8%`, none of which are on D-014's allowlist. Flat u/s and
flat °/s fix that and are easier to reason about.

**Rotation starts faster and tops out lower**, as you asked. v0's 300 → 1200°/s put max
rotation at 3.3 turns per second, which is past the point of usefulness. 420 → 720°/s
means a new player can already turn briskly and a maxed one gets two turns per second.

### The projectile-to-ship speed ratio matters

At 1000 u/s base, the Valrune crosses the screen width in 1.0s, its height in 2.2s.

**Projectile speed must stay well above ship speed** — 2400 against 1000 is 2.4×. If it
narrows, shooting while advancing feels wrong: you chase your own bullets, and forward
shots appear to crawl while backward shots race. Any future change to either number should
preserve roughly this ratio.

At 2400 u/s a projectile crosses `dis4` in 0.67s.

---

## 8. Time

```
TICK         = 0.2s     Fixed (D-048)
TICK_FAST    = 0.1s     Fixed — strict subdivision only
WINDOW       = 120s     Fixed (D-028) — the balance measurement window
COOLDOWN_MAX = 120s     Fixed (D-023)
```

---

## 9. The Expanse

```
EXPANSE_WIDTH  = 6000    Starting point
EXPANSE_HEIGHT = 6000    Starting point
```

Square, wrapping on both axes (D-059). Six screen-widths across, about 2.7 screen-heights.

At the new base speed of 1000 u/s it takes **6 seconds to cross**, down from 13.6 at v0's
440 — the faster ship is what makes an arena this size work.

### What "torus" means

A rectangle that wraps on both edges is topologically a **donut**. Fly off the right edge
and you reappear on the left; off the top and you reappear at the bottom. Practically it
means the arena has **no edges, no corners, and no escape** — every direction eventually
returns you to where you started.

Three consequences worth holding onto:

- **You can never be cornered**, which removes the worst feeling in the genre.
- **You can never truly flee.** Running from a boss brings you back to it.
- **The minimap must be drawn torus-aware.** A baddie near the right edge is also near the
  left edge. A minimap that hides that is worse than no minimap, because it teaches the
  wrong mental model.

---

## 10. Rank line numbers settled so far

| Line | Base | Ranks | Max | Note |
|---|---|---|---|---|
| `crit_chance` | **2%** | 8 × +1% | 10% | Free baseline, see below |
| `crit_damage` | 1.25× | 25 × +0.05 | 2.50× | Unchanged |
| `attack_speed` | 3.0/s | 24 × +0.5 | 15.0/s | Per D-012 |
| `homing` | 2° | 8 × +2° | 18° | See below |
| `bulwark_flat` | 0 | 16 × +1 | 16 | See below |
| `thrusters` | 1000 u/s | 20 × +40 | 1800 u/s | |
| `gyros` | 420 °/s | 20 × +15 | 720 °/s | |
| `velocity` | 2400 u/s | 5 × +160 | 3200 u/s | |

### `crit_chance` — free baseline drops to 2%

v0 gave 5% free so that crit-triggered nodes are not dead on a fresh account. That
reasoning still holds, but 5% was priced against 2 shots/sec. **At 3.0 shots/sec a 2%
baseline still produces a crit every 17 seconds**, which keeps those nodes alive while
making an early crit feel like an event.

The PRD constant table must be regenerated for the 1–10% range (D-012). v0's table covers
5–30% and is now useless.

### `homing` (D-060)

At 18° a target anywhere in a 36° cone gets hit — forgiving without being auto-aim.

**The 2° free baseline mirrors the free crit**, for a different reason with the same shape:
rate-based rotation plus screen-relative movement means a new player is fighting two axes
at once, and zero correction would make the opening hour feel broken rather than hard.
**M0 should test 0° against 2°** — obvious in thirty seconds on a phone, unknowable at a
desk.

### `bulwark_flat` (D-058)

```
DAMAGE_FLOOR = 25% of post-element damage    Fixed (v0 §4.3)
```

Sixteen ranks absorb part of the 3,500-credit sink that vanished with `bulwark_percent`.

**Flat mitigation is strong early and weak late, and that is the point.** Against Sector 1
damage of 3–5 it would zero every hit if the floor did not exist; against Sector 5 damage
near 60 it is a 27% reduction. That natural decay is exactly why flat beats percentage —
it does not compound with evasion, rime, and shields the way D-029 warns about.

**The 25% floor is load-bearing and must never be removed.**

---

## 11. Accessibility — cut Assist Aim, keep the settings

You asked whether any of this is needed. Mostly no, and the v0 list conflates two
different things.

**Keep — these are just settings every shooter ships**, each costing hours not days:
screen shake 0–100%, flash reduction, haptics toggle, colorblind palettes, handedness
swap, stick size, rotation sensitivity.

Flash reduction is worth keeping regardless of accessibility framing: photosensitivity is
the one area where storefronts and some regions actually care, and it is nearly free.

**Cut Assist Aim.** It was the only item with real cost — auto-targeting logic, a
targeting priority rule, and an interaction with every ability that picks targets. And
**`homing` now does its job better**: a player who wants more forgiveness buys homing
ranks, which is a progression decision rather than a menu toggle.

This also dissolves O-14 rather than resolving it. There is no longer any question about
selling a purchasable version of an accessibility feature, because the feature is gone.

---

## 12. Naming

Variables use **full words** in code (D-047): `duration`, `cooldown`, `radius`, `stacks`,
`damage`. The source spreadsheets keep their short forms.

**Band ids stay short** — `r1`, `dis2`, `w1`, `push_1`, `pull_2` — because they are token
values, not variables.

Tooltips say **"short / medium / long" plus the unit count**; data says `r2`.

---

## Open questions

| # | Question | Blocks |
|---|---|---|
| 1 | **`CLUSTERING = 2.0`** is a guess and every AoE valuation rides on it. Not answerable at a desk — it becomes a slider in the Balance Lab and gets measured in M0 | M0 |
| 2 | **Expanse at 6000²** — 6s to cross at base speed. A torus too large stops reading as a torus | M0 |
| 3 | **`HOMING_BASE`** — 0° or 2°? | M0 |
| 4 | **`VALRUNE_SPEED_BASE = 1000`** is 2.3× v0's. Fast for a thumb-driven ship, and it drives the Expanse size and the projectile ratio | M0 |
