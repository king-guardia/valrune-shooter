# 15 — Constants

**Status: draft, awaiting Bryan's review.** Vocabulary is fixed by
[`14-CANON.md`](14-CANON.md); this file puts numbers on it.

Every value here is one of three kinds, and the difference matters:

| Mark | Meaning |
|---|---|
| **Fixed** | Derived from geometry or a decision. Changing it changes the design. |
| **Anchor** | A judgment call that everything else is measured against. Changing it rescales the ladder. |
| **Starting point** | A guess. Expected to move once M0 or the balance engine exists. |

Nothing here is a balance number in the sense D-014 means — these are the *units* balance
numbers are expressed in. `data/constants.json` is generated from this file in Phase 2a.

---

## 1. The playfield

```
PLAYFIELD_WIDTH        = 1000    Fixed (D-024)
PLAYFIELD_HEIGHT_REF   = 2200    Fixed — 20:9, the reference device
PLAYFIELD_HEIGHT_MIN   = 2000    Fixed — 18:9, the minimum supported
READY_LINE_FROM_BOTTOM =  360    Anchor
```

Width is fixed and height varies with the device. That is the correct way round: fixing
height instead would make the ship's size and every radius change between phones, and
balance would not transfer.

**Common aspects at 1000 wide:**

| Aspect | Height | Notes |
|---|---|---|
| 16:9 | 1778 | Old and budget devices. Below minimum — see below. |
| 18:9 | 2000 | **Minimum supported.** |
| 19.5:9 | 2167 | Very common. |
| 20:9 | 2200 | **Reference.** |
| 21:9 | 2333 | Extra sky, no downside. |

**All vertical geometry is sized against 2000, not 2200.** A taller phone gets bonus space
at the top — slightly more warning time, which is a mild advantage and not worth
correcting. Sizing against the reference instead would push content off-screen on an 18:9,
which is a real failure.

16:9 at 1778 is 222 units short of the minimum [OPEN]. Options are to letterbox, to accept
a compressed playfield, or to drop support. Android 16:9 is now mostly budget devices and
tablets; worth a market-share check before spending effort.

**The ready line is measured from the bottom, not as a percentage.** At 360 units up, the
Valrune sits clear of resting thumbs on every device. A percentage would drift with aspect
and put the ship under your hand on short screens.

The input zones (`03` §3.2.1) are a **transparent overlay** on the lower 38%, not a region
carved out of the playfield. Projectiles and baddies occupy that space normally.

---

## 2. Entity sizes

```
VALRUNE_WIDTH   = 80    Anchor
BADDIE_MINION   = 64    Starting point
BADDIE_ELITE    = 90    Starting point
BADDIE_MINIBOSS = 200   Starting point
BADDIE_BOSS     = 400   Starting point
PROJECTILE_W    = 8     Starting point
```

The Valrune at 80 units is **8% of screen width**, which delivers the "smaller ship,
slightly zoomed out" you asked for. Sanity check against your density targets: 30 minions
at 64 units wide is 1,920 units of total width, so under three screen-widths — comfortably
spread over a 2200-tall screen. A 75-baddie zerg is about seven rows' worth, dense but
navigable. **The density targets and the ship size are consistent**, which is worth
knowing before either gets tuned.

---

## 3. Radius bands

Regions **around** a point. Anchored on `r1 = 120` (D-024).

| Band | Units | % of width | Typical use |
|---|---|---|---|
| `r_short` | 60 | 6% | Forge impact, tight detonations |
| `r1` | 120 | 12% | The workhorse — chain jumps, static touch, most auras |
| `r2` | 240 | 24% | Upgraded versions of the above |
| `r3` | 400 | 40% | Vacuum reach, the largest routine AoE |

### What these actually hit

This is the whole reason the playfield got a unit count. With
`area = π r² / (1000 × 2200)`:

| Band | Playfield covered | Baddies hit at 30, uniform | At clustering 2.0 |
|---|---|---|---|
| `r_short` | 0.5% | 0.15 | 0.3 |
| `r1` | 2.1% | 0.6 | 1.2 |
| `r2` | 8.2% | 2.5 | 4.9 |
| `r3` | 22.8% | 6.9 | 13.7 |

**Uniform distribution understates every AoE**, because baddies arrive in waves and
converge on you. `CLUSTERING` is therefore a first-class tunable in the balance engine,
not a constant here — a Wormhole wave front clusters far harder than an Expanse scatter.

```
expected_hits = density × (π r² / playfield_area) × CLUSTERING
CLUSTERING = 2.0    Starting point — must be measured in M0
```

That r1 catches roughly one extra baddie is the correct read, and it is why chain
lightning specifies "the nearest baddie within r1" rather than "all baddies."

---

## 4. Distance bands

Reach **away from** a point. Per D-054, coupled to radius only at the short end.

| Band | Units | Derivation |
|---|---|---|
| `dis_short` | 60 | = `r_short` **Fixed** |
| `dis1` | 120 | = `r1` **Fixed** |
| `dis2` | 440 | Geometric mean of `dis1` and `dis3` |
| `dis3` | 1600 | **Ready line to the top border** on the minimum device |
| `dis4` | 2400 | 1.5 × `dis3` — wrapped angled shots, and the Expanse |

`dis3` is the anchor with real meaning: at 2000 height minus 360 for the ready line, the
usable column is 1640, so **1600 reaches the top of the screen on the smallest supported
device** and covers most of it on taller ones. Exactly your intent, made device-safe.

### One finding worth flagging

**The ladder is bottom-heavy, and it is structural.** `dis1` is pinned to `r1` (120, small
by design) while `dis3` is pinned to screen height (1600, necessarily large). The gap
between them spans more than a factor of thirteen, and 440 is only the geometric mean —
there is no natural band in the 150–400 range, which is exactly where a lot of mid-range
projectile abilities will want to sit.

I have not invented one, because the honest way to size it is to convert the abilities
first and see where they actually cluster. **Expect to add `dis1_5` or to re-space the
ladder in Phase 2**, once the 89 abilities are in JSON and the histogram is visible.

---

## 5. Width bands

Valrune-relative, per D-054, which makes them self-documenting.

| Band | Units | Definition |
|---|---|---|
| `w0` | 8 | A projectile's own width |
| `w1` | 40 | Half the Valrune |
| `w2` | 80 | The full Valrune |
| `w3` | 240 | Two Valrunes plus half a Valrune each side |

CRYO's wave blast will likely want `w4` given how short its reach is. Extend the ladder;
do not special-case.

---

## 6. Push, pull, and gravity

Per D-055, gravity disables self-movement and pull becomes the only movement, so nothing
arbitrates.

**Push is a one-shot displacement** — a `(distance, time)` pair:

| Band | Distance | Time | Feel |
|---|---|---|---|
| `push_1` | 40 | 0.08s | The Forge tick — a stutter, not a launch |
| `push_2` | 100 | 0.12s | A real knock |
| `push_3` | 220 | 0.20s | A detonation |

**Pull is a sustained speed**, not a displacement:

| Band | Speed | vs a minion at 120 u/s |
|---|---|---|
| `pull_1` | 80 u/s | Slowed but still advancing — your line-15 case exactly |
| `pull_2` | 200 u/s | Overpowered; it comes to you |
| `pull_3` | 450 u/s | Near player top speed. Inescapable under gravity |

**This deviates slightly from D-055**, which described both as `(distance, time)`. Pull is
sustained and open-ended — its distance depends on how long the status lasts and where the
target started — so a speed is the honest parameterization. Push genuinely is a fixed
displacement over a fixed window. Recorded as D-062.

`pull_stop_radius` defaults to `null` (pull all the way to the point); a value stops the
pull at that ring while gravity keeps holding the target.

---

## 7. Speeds

```
VALRUNE_SPEED_BASE   = 440 u/s    Starting point (v0: 440 Field, 520 Wormhole)
VALRUNE_SPEED_MAX    = 4.0×       Fixed (D — thrusters, 20 ranks)
BADDIE_SPEED_MINION  = 120 u/s    Anchor — the reference all pulls are read against
PROJECTILE_SPEED     = 2000 u/s   Starting point
ROTATION_BASE        = 300°/s     Starting point (v0 §3.2.3)
ROTATION_MAX         = 4.0×       Fixed (gyros, 20 ranks)
```

At 2000 u/s a projectile crosses `dis3` in **0.8s**, which is the right order for a
shooter — fast enough not to feel floaty, slow enough that leading a target is a skill.

The Valrune at 440 u/s crosses the screen width in 2.3s and the full height in 5s.

---

## 8. Time

```
TICK       = 0.2s     Fixed (D-048)
TICK_FAST  = 0.1s     Fixed — strict subdivision only
WINDOW     = 120s     Fixed (D-028) — the balance measurement window
COOLDOWN_MAX = 120s   Fixed (D-023)
```

Every periodic effect lands on a `TICK` boundary. Never an unaligned value.

---

## 9. The Expanse

```
EXPANSE_WIDTH  = 3000    3 screens    Starting point
EXPANSE_HEIGHT = 4400    2 screens    Starting point
```

Wraps on both axes (D-059). Total area is **6× a Wormhole screen**, in the same range as
v0's disc (~9×) but rectangular so it can actually wrap.

Crossing it takes about 7s horizontally and 10s vertically at base speed — long enough to
feel open, short enough that you meet the wrap often and learn it. **This is a feel number
and M0 should challenge it.** A torus that is too large stops reading as a torus and just
feels like a big empty box.

The minimap must be drawn **torus-aware** (D-059): a baddie near the right edge is also
near the left edge.

---

## 10. Rank lines needing numbers

### `homing` (D-060)

```
HOMING_BASE      = 2°     Starting point
HOMING_RANKS     = 8
HOMING_PER_RANK  = 2°
HOMING_MAX       = 18°
```

**The 2° free baseline is deliberate and mirrors the free 5% crit** (v0 §4.5), which
exists so that crit-triggered nodes are not dead on a fresh account. Here the reasoning is
different but the shape is the same: rate-based rotation plus screen-relative movement
means a brand-new player is fighting two axes at once, and zero correction would make the
opening hour feel broken rather than difficult.

**M0 should test 0° against 2°** before this is settled — it is exactly the kind of thing
that is obvious in thirty seconds on a phone and unknowable at a desk.

At 18°, a target anywhere in a 36° cone gets hit. That is forgiving without being
auto-aim, which stays the accessibility toggle's job.

### O-14 resolved — `homing` vs Assist Aim

They are different mechanisms and never overlap:

| | Bends | Player sees |
|---|---|---|
| **`homing`** | The **projectile**, after it leaves the gun | Shots curve |
| **Assist Aim** | The **ship's heading** | The ship turns |

Nothing purchasable is gated behind the accessibility toggle, and nothing about the
toggle is for sale. Recorded as D-063.

### `bulwark_flat` (D-058)

```
BULWARK_RANKS    = 16     Starting point (v0: 8, before the % line was deleted)
BULWARK_PER_RANK = 1      damage reduced per hit
BULWARK_MAX      = 16
DAMAGE_FLOOR     = 25%    Fixed (v0 §4.3) — of post-element damage
```

Doubling the ranks absorbs part of the 3,500-credit sink that vanished with
`bulwark_percent`.

**Flat mitigation is strong early and weak late, and that is intended.** Against Sector 1
damage of 3–5 it would zero every hit if the 25% floor did not exist; against Sector 5
damage near 60 it is a 27% reduction. That curve is the *reason* to prefer flat over
percentage — it decays naturally instead of compounding with evasion, rime, and shields
the way D-029 warns about.

The 25% floor is load-bearing and must never be removed.

---

## 11. Naming

Variables use **full words** in code (D-047): `duration`, `cooldown`, `radius`, `stacks`,
`damage`. The source spreadsheets keep their short forms.

**Band ids stay short** — `r1`, `dis2`, `w1`, `push_1`, `pull_2` — because they are token
values, not variables. `radius: "r2"` reads correctly.

Tooltips say **"short / medium / long" plus the unit count**; data says `r2`.

---

## Open questions

| # | Question | Blocks |
|---|---|---|
| 1 | **16:9 support.** 1778 height is 222 short of the minimum. Letterbox, compress, or drop? Worth a market-share check first | Phase 6 |
| 2 | **The `dis1`–`dis2` gap.** Nothing sits in 150–400. Best resolved by converting the abilities and looking at the histogram, not by guessing now | Phase 2 |
| 3 | **`CLUSTERING = 2.0`** is a pure guess and every AoE valuation rides on it | M0 |
| 4 | **Expanse size.** A torus that is too large stops reading as a torus | M0 |
| 5 | **`HOMING_BASE`** — 0° or 2°? Thirty seconds on a phone answers it | M0 |
