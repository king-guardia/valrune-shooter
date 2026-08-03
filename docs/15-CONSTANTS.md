# 15 — Constants

**Status: draft, revision 4 — homing recapped, `piercing` added.** Vocabulary is fixed by
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

Measured, not guessed: a reference screenshot at 400px screen width converts cleanly at
**2.5 units per pixel**, and the ship read well at 40px.

```
VALRUNE_WINGSPAN = 100   Anchor — port to starboard, 10% of screen width
VALRUNE_FUSELAGE = 110   Anchor — bow to stern, so 55 from center either way
BADDIE_WINGSPAN  =  75   Anchor — the standard baddie
BADDIE_MIN       =  50   Fixed — nothing is smaller
PROJECTILE_WIDTH =   8   Starting point
```

**There is no maximum baddie size.** A boss may fill the screen.

| Threat class | Wingspan | Relative to the Valrune |
|---|---|---|
| Debris | 50 | Half |
| Minion | 75 | Three quarters — **the anchor** |
| Elite | 110 | Slightly larger |
| Miniboss | 250 | 2.5× |
| Boss | 500 | 5×, half the screen width |

`wingspan` and `fuselage` are new canon (D-073). They join `bow`/`stern`/`port`/`starboard`
and give the ship real geometry nouns, so "55 units forward of center" has a name instead
of being restated every time.

### Expected baddie counts

```
COUNT_SWARM    = 200   Fixed — the hard ceiling, never more on screen
COUNT_STANDARD =  85   Anchor — a normal wave, range 75–100
COUNT_TANK     =  20   Starting point — fewer, larger, slower
```

**This is a 3× correction to the old assumption and it is the most consequential change in
this revision.** Every AoE valuation in section 3 was written against 30 baddies. At 85
standard and 200 in a swarm, radius abilities are worth several times what the previous
numbers implied.

---

## 3. Radius bands

Regions **around** a point, measured from the center of the ship or effect.

| Band | Units | % of width | Relative to the Valrune | Uses |
|---|---|---|---|---|
| `r_short` | 100 | 10% | Just clear of the hull | 4 |
| `r1` | 150 | 15% | 1.5 wingspans | **51** |
| `r2` | 250 | 25% | 2.5 wingspans | 29 |
| `r3` | 400 | 40% | 4 wingspans | 7 |

**Four rungs is right — the authoring proves it.** Nothing across the ability, attunement,
or status tables reaches for a fifth, and the distribution is heavily bottom-weighted.

**`r1` is the single most consequential number in the game.** Fifty-one abilities read
against it, so the balance engine should report its sensitivity separately from everything
else.

### What these actually hit — O-21 resolved

Measured from your screenshot: **~170 baddies on screen, a semi-targeted `r1` landing on
~15 of them.** That is a real data point and it pins a constant that everything AoE depends
on.

Working backwards, against the action zone (1000 × 2000 = 2,000,000 sq units, where baddies
actually live):

```
r1 covers π × 150² / 2,000,000 = 3.53% of the field
uniform expectation at 170 baddies = 6.0
observed = 15
```

**So the field is roughly 2.5× denser where you aim than it is on average.** That number
bundles two different things, and the balance engine needs them apart:

```
CLUSTERING_BLIND = 1.85    Baddies clump on their own — self and fixed_point deliveries
CLUSTERING_AIMED = 2.50    Plus the player choosing the thickest part — targeted deliveries
```

A nova centered on your own hull cannot pick its spot; a lobbed bomb can. Valuing both at
the same number would overpay every self-centered ability in the game.

**Clustering must decay as the radius grows.** A tiny AoE can always find a dense pocket; an
AoE covering a quarter of the field cannot do better than the field average. So:

```
effective = 1 + (CLUSTERING − 1) × (1 − area_ratio)
hits = min(count × area_ratio × effective, count)
```

| Band | Field covered | Aimed, swarm (200) | Aimed, standard (85) | Blind, standard (85) |
|---|---|---|---|---|
| `r_short` | 1.6% | 7.8 | 3.3 | 2.5 |
| `r1` | 3.5% | **17.3** | 7.3 | 5.5 |
| `r2` | 9.8% | 46.2 | 19.6 | 14.7 |
| `r3` | 25.1% | 107 | 45.3 | 34.9 |

The model reproduces your observation — 14.7 at 170 baddies against the 15 you counted —
which is about as much validation as a single data point can give.

**Compare against what this file said last revision: `r1` hit 1.2 baddies.** It hits 7 in a
standard wave and 17 in a swarm. **Every radius ability in the game is worth 6–14× what the
previous numbers implied**, and single-target abilities have not moved at all. Nothing in
the ability set has been balanced against this yet. The Phase 4 engine has to run before
any AoE numbers are trusted.

---

## 4. Distance bands

Reach **away from** a point. **Distances are measured from the bow, radii from the center**
(D-074) — a projectile spawns at the bow, so a distance is literally how far it travels.

The near rungs are offset by the 55-unit half-fuselage so they finish flush with a radius
ring. An ability reaching `dis_short` and an ability covering `r_short` cover the same
forward ground:

| Band | From bow | Reaches, from center | Derivation |
|---|---|---|---|
| `dis_short` | 45 | 100 | Flush with `r_short` **Fixed** |
| `dis1` | 95 | 150 | Flush with `r1` **Fixed** |
| `dis2` | 345 | 400 | Flush with `r3` **Fixed** |
| `dis3` | 800 | 855 | Mid-screen |
| `dis4` | 1600 | 1655 | **Maximum — ready line to the top of the action zone** |
| `dis5` | 2400 | — | Beyond maximum — wrapped angled shots, the Expanse |

The low three are odd numbers on purpose, because ring alignment is what they are for. The
far three are round, because at 800+ the 55-unit bow offset is under 7% and alignment stops
mattering — those rungs are about screen reach.

`dis4` keeps its physical meaning: 2000 minus 360 for the ready line leaves 1640, so **1600
reaches the top of the action zone**, and your solar beam "out to distance_3 (max distance)"
lands exactly where you wrote it.

### What the existing content maps to

Your spreadsheets use three distance rungs, 36 references total:

| You wrote | References | Now means |
|---|---|---|
| `distance_1` | 8 | `dis1` |
| `distance_2` | 19 | `dis2` |
| `distance_3` | 9 | **`dis4`** |

Solar beam annotates `distance_3` as "(max distance)", so the rename matches your intent
rather than reinterpreting it. Phase 2 does the remap mechanically.

**`dis3` at 800 is currently unused**, sitting empty until something needs it. Spectral
Lance is the likely first customer: it fires to max distance and pays bonus damage "beyond
distance_2", and 800 is a better threshold for that than 400.

---

## 5. Width bands

| Band | Units | Definition | Uses |
|---|---|---|---|
| `w1` | 50 | Half a wingspan — a wide beam | 3 |
| `w2` | 100 | A full wingspan — a wider beam | 2 |
| `w3` | 200 | A wingspan plus half either side — **CRYO's wave blast** | 0 |
| `w4` | 400 | A wingspan plus 1.5 either side — held in reserve | 0 |

**`w0` is deleted rather than renumbered.** A projectile's width is not a band anybody
authors against — nobody writes `width: w0`. It is a property of the projectile, so it
lives in entity sizes as `PROJECTILE_WIDTH` and the ladder starts at 1 like every other
ladder.

`w4` exists for abilities that do not exist yet, most likely a boss attack. Leaving a rung
unclaimed is cheaper than discovering the ladder is too short mid-authoring.

---

## 6. Push, pull, and gravity

You are right that a band table would explode: push and pull each want a distance *and* a
time, and pull additionally wants to speak in radii rather than distances. Three dimensions
of banding is a combinatorial mess.

**But fully custom numbers are worse**, because the Balance Calibrator cannot compare two
abilities that each invented their own displacement. Parity checking needs banded values.

**The fix is to compose existing ladders rather than build a new one** (D-075). There are no
`push_1`/`pull_2` ids. There is only push and pull, each written as a pair:

```
push: { distance: <any distance or radius band>, time: <time class> }
pull: { distance: <any distance or radius band>, time: <time class> }
pull: { from: <radius band>, to: <radius band>, time: <time class> }
```

Distance draws from the ladders that already exist. Only **time** is new, and it needs four
values, not a matrix:

| Class | Seconds | Feel |
|---|---|---|
| `t_snap` | 0.08 | An impulse — the Forge knock, a stutter rather than a launch |
| `t_quick` | 0.25 | A real shove |
| `t_steady` | 1.00 | Reads as a rate; the natural pairing for a sustained pull |
| `t_slow` | 2.50 | A long drag |

Full expressiveness, no new table, and both dimensions stay banded so parity still works.

### Two forms of pull

Your gravity-well case — "pulling from radius 3 to within radius 2" — is not a displacement,
it is a destination. Both forms exist:

- **`pull` by distance** displaces the target N units toward the origin, and keeps going.
- **`pull` from/to** draws the target inward until it is within the inner radius, then
  stops pulling while the status keeps holding it.

The second is what almost every gravity ability you wrote actually wants.

### What does not change

**Neither push nor pull stops the target's own movement.** Each contributes a velocity that
adds to whatever the target was already doing. **Gravity is the only status that disables
self-movement** (D-055), and only then does a pull become the target's entire motion.
Nothing arbitrates, because nothing competes.

**One thing to watch:** displacement reads completely differently against a minion at
120 u/s and the Valrune at 1000 u/s. A pull that drags a baddie decisively would barely
inconvenience the player. Anything that pulls the player has to pair with gravity to matter
at all.

---

## 7. Speeds

```
VALRUNE_SPEED_BASE   = 1000 u/s    Anchor — 1.0s to cross screen width
VALRUNE_SPEED_RANKS  = 20 × +40    → 1800 u/s at max
BADDIE_SPEED_MINION  =  120 u/s    Anchor — what displacement reads against
PROJECTILE_SPEED     = 2800 u/s    Starting point — see the floor rule below
PROJECTILE_RANKS     = 5 × +160    → 3600 u/s at max
ROTATION_BASE        =  420 °/s    Starting point
ROTATION_RANKS       = 20 × +15    → 720 °/s at max
```

**Speed ranks are now flat per rank, not percentages.** v0 had thrusters and gyros as
`+0.15× base` and velocity as `+8%`, none of which are on D-014's allowlist. Flat u/s and
flat °/s fix that and are easier to reason about.

**Rotation starts faster and tops out lower**, as you asked. v0's 300 → 1200°/s put max
rotation at 3.3 turns per second, which is past the point of usefulness. 420 → 720°/s
means a new player can already turn briskly and a maxed one gets two turns per second.

### Velocity inheritance is cut, and replaced by a floor rule

Projectiles do not inherit ship velocity (D-072 reversed). Instead:

```
PROJECTILE_SPEED must exceed VALRUNE_SPEED at maximum thrusters, with margin.
```

The worst case is a maxed thruster line and an untouched velocity line — 1800 u/s ship
against a base projectile. **At the old 2400 that ratio is 1.33×, which satisfies the rule
on paper and still feels wrong**: charging forward, your bullets pull ahead at only 600 u/s
and appear to hang in front of the ship.

**Base moves to 2800** so the worst case is 1.56× and the normal case is 2.8×:

| Situation | Ship | Projectile | Ratio |
|---|---|---|---|
| Fresh account | 1000 | 2800 | 2.8× |
| **Worst case** — thrusters maxed, velocity untouched | 1800 | 2800 | **1.56×** |
| Both maxed | 1800 | 3600 | 2.0× |

At 2800 u/s a projectile crosses `dis4` in 0.57s and the screen width in 0.36s.

### Velocity buys turn rate too, and the units make it automatic

Your point is correct and worth stating precisely. A projectile's turning circle is
`speed ÷ angular_rate`, so a faster projectile with the same degrees-per-second turns
through a **wider** arc and homes worse. Buying velocity would quietly sell you accuracy.

Holding the turning circle constant means angular rate must scale linearly with speed —
which is exactly what happens if homing is denominated **per unit travelled rather than per
second**:

```
HOMING_RATE = 1° per 100 units travelled    Starting point — see §10
```

**Speed then cancels out of the path entirely.** A 2800 u/s and a 3600 u/s projectile trace
the identical curve, one just gets there sooner. No separate turn-rate rank line, no
coupling constant to tune, and the interaction that would have needed watching in M0 simply
cannot occur.

---

## 8. Time

```
TICK         = 0.2s     Fixed (D-048)
TICK_FAST    = 0.1s     Fixed — strict subdivision only
WINDOW       = 120s     Fixed (D-028) — the balance measurement window
COOLDOWN_MAX = 120s     Fixed (D-023)
```

**Every status duration must be a multiple of `TICK_FAST`** (D-089). This is not cosmetic: an
off-grid duration produces a final partial tick whose damage depends on floating-point drift,
which breaks determinism. `stagger` was authored at 0.05s and `stagger_plus` at 0.09s — both
below tick resolution — and move to **0.1s and 0.2s**.

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
| `crit_chance` | 1.0% | 12 × +0.25% | 4.0% | See below |
| `crit_damage` | 1.25× | 25 × +0.05 | 2.50× | Unchanged |
| `attack_speed` | 3.0/s | 24 × +0.5 | 15.0/s | Per D-012 |
| `homing` | 1.0° | 5 × +0.5° | 3.5° | 98 units lateral at max range, see below |
| `piercing` | 0 | 4 × +1 | 4 | **New line**, see below |
| `bulwark_flat` | 0 | 16 × +1 | 16 | See below |
| `thrusters` | 1000 u/s | 20 × +40 | 1800 u/s | |
| `gyros` | 420 °/s | 20 × +15 | 720 °/s | |
| `velocity` | 2800 u/s | 5 × +160 | 3600 u/s | Carries turn rate with it |

### `crit_chance` — 1.0% base, 12 ranks × +0.25%, max 4.0% (D-070)

At 10% and 15 shots/sec you get 1.5 crits per second, which is not a critical hit, it is a
texture. The ceiling comes down hard. But **crit frequency is `fire_rate × crit_chance`,
and both scale up together**, so frequency scales multiplicatively and the floor cannot be
set independently of the ceiling:

| | Fire rate | Crit chance | Crits/sec | One crit every |
|---|---|---|---|---|
| v0 start | 2.0 | 5% | 0.10 | 10s |
| v0 max | 8.0 | 30% | 2.40 | 0.4s |
| **Locked** start | 3.0 | 1.0% | 0.03 | 33s |
| **Locked** mid | 9.0 | 2.5% | 0.22 | 4.4s |
| **Locked** max | 15.0 | 4.0% | 0.60 | 1.7s |

A 0.05% floor was considered and rejected: it puts **11 minutes between crits** for a new
player, which would make every attunement crit effect — one per element, authored content —
invisible for the entire early game. The 1.0% floor keeps a crit rare enough to feel like
an event at 33 seconds apart while still letting those nodes fire.

The campaign still swings 20× from start to finish. That is accepted as the cost of
keeping the line legible as a percentage.

**This constrains AoE-on-crit and chain-on-crit attunement effects.** At the top end they
fire every 1.7 seconds, so the Balance Calibrator must value them against 0.6 procs/sec,
not against the rare-event framing the early game implies.

The PRD table must be regenerated for the 1–4% range — v0's covers 5–30% and is now
useless. Note that PRD at these low chances needs a long tail, since the naive constant
approaches zero and the shot counter runs into the hundreds between crits.

### `homing` (D-060)

```
homing       = total angular correction budget, 1.0° → 3.5°
HOMING_RATE  = 1° per 100 units travelled
HOMING_RANGE = dis4 (1600)    Beyond this, no correction at all
```

**The ceiling is set by lateral reach, not by the angle.** You want at most 100 units of
sideways correction at maximum range, and `atan(100 / 1600)` is 3.58°, so **3.5° is the
cap** and it buys 98 units.

That is an 80% cut from the 18° this file carried yesterday, and it is the right call — 18°
reached 520 units sideways, over half the screen width, from a shot you aimed badly.

| Rank | Degrees | Lateral reach at `dis4` |
|---|---|---|
| Base | 1.0° | 28 |
| 1 | 1.5° | 42 |
| 2 | 2.0° | 56 |
| 3 | 2.5° | 70 |
| 4 | 3.0° | 84 |
| **5 (max)** | **3.5°** | **98** |

**Degrees and units are interchangeable at this scale.** Below about 5°, `tan` is close
enough to linear that each half-degree is a flat ~14 units of reach at max range. The rank
line reads linearly to the player whichever unit the tooltip uses, so the tooltip should say
units — nobody has intuition for 2.5 degrees.

Against a 75-unit baddie, base homing widens the hit window from ±37.5 to ±65 and maxed
homing takes it to ±135. Roughly triple, without ever feeling like the game is aiming.

**`HOMING_RATE` drops to 1°/100u** to match. At the old 6° the entire 3.5° budget would be
spent within 58 units of leaving the barrel — a visible snap rather than a curve. At 1° the
budget takes 350 units to spend, which reads as a gentle arc.

**Homing is a budget, not a rate.** The rate governs how fast it is spent; the rank line
governs how much there is. The budget only becomes reachable past 350 units of travel, and
that is correct rather than a limitation — **close range is already forgiving in absolute
terms.** A standard baddie subtends 20.6° at 100 units but only 1.34° at `dis4`. Homing
matters exactly where the geometry gets cruel, and there is plenty of flight distance there
to spend the budget.

**Nothing homes beyond `dis4`** (D-076). A shot fired across the Expanse at something 3000
units away flies straight.

**Homing acquires the nearest valid baddie along the line of fire and ignores everything
behind it** (D-080). No re-evaluation for a better target, no reaching past the first thing
in the way. This is what keeps homing and piercing from fighting: homing decides where the
shot goes, piercing decides how far it keeps going after it gets there.

**The 1.0° free baseline mirrors the free crit**, for a different reason with the same
shape: rate-based rotation plus screen-relative movement means a new player is fighting two
axes at once, and zero correction would make the opening hour feel broken rather than hard.
**M0 should test 0° against 1.0°** — obvious in thirty seconds on a phone, unknowable at a
desk.

### `piercing` — the structural answer to the AoE gap (D-081)

```
piercing = 0 base, 4 ranks × +1, max 4
```

A gun-shot passes through `piercing` baddies and continues, so it lands `piercing + 1` hits.
Zero at base: shots stop on first contact, as they do today.

**This is the right fix for the problem D-079 exposed, and it works because it scales with
the same variable AoE does.** Flat damage buffs would have helped single-target shots
everywhere including against bosses, where they were never weak. Piercing pays out only when
there is a crowd — which is exactly where the gap opened.

The reach of a gun-shot through a crowd is a mean-free-path problem. A projectile sweeps a
corridor `PROJECTILE_WIDTH + BADDIE_WINGSPAN` = 83 units wide, so the average gap between
baddies along the line of fire is:

```
gap = field_area / (count × 83 × clustering)
```

| Situation | Count | Mean gap | Hits at P=0 | P=2 | **P=4** | Multiplier |
|---|---|---|---|---|---|---|
| Swarm | 200 | 67 | 1.0 | 3.0 | **5.0** | **5.0×** |
| Standard wave | 85 | 158 | 1.0 | 3.0 | **5.0** | **5.0×** |
| Tank wave | 20 | 672 | 1.0 | 2.0 | **2.4** | **2.4×** |
| Boss, alone | 1 | — | 1.0 | 1.0 | **1.0** | **1.0×** |

**The shape is exactly right.** Full value in crowds, partial against a handful of tanks,
nothing at all against a lone boss — where single-target output was already competitive,
since an AoE hitting one target is just a worse single-target hit.

Note the incidental finding in that table: **at 85 baddies your shots currently die 158
units out of the barrel.** A gun-shot at `piercing: 0` in a standard wave never travels a
tenth of its range. That is most of why single-target felt weak.

### Three things piercing collides with

**1. Crit must roll once per gun-shot, not once per hit.** Per hit, a maxed loadout gets
15 shots/sec × 5 hits × 4% = **3.0 crits/sec**, blowing straight through the 0.6/sec we
just set in D-070. Rolling per gun-shot preserves the cadence exactly, and it makes crits
read better anyway: the whole line lights up at once. Attunement on-crit effects proc once
per gun-shot for the same reason.

**On-hit effects still apply per hit.** Spreading burn down a line of five is the point.

**2. `guns` and `piercing` multiply, and that is the D-029 pattern.** Three guns each
piercing four is fifteen hits per gun-shot. Stacked with attack speed, the offense
multiplier from base to maxed reaches roughly 75×, which is a steeper curve than anything
else in the tree. **The Balance Calibrator has to treat guns × piercing × attack_speed as
one compound axis**, not three independent lines.

**3. Piercing must be priced against the crowd case.** At four ranks it is a 5× damage
multiplier for most of the game. If it is cheap, single-target guns become the correct
answer to every encounter and the AoE abilities go back to being worse — the same problem
inverted.

**No damage falloff per pierce.** Flat is simpler, consistent with D-014, and the rank cost
is a cleaner lever than a decay curve nobody can feel.

Piercing is a **basic-attack stat**. Abilities declare their own piercing in data, and the
ones you wrote that grant piercing get reworked against this line rather than stacking with
it.

### Homing means something different per delivery (D-077)

"Bending" is only right for things that fly. The other deliveries get the same budget
applied at the moment of firing:

| Delivery | How correction is spent |
|---|---|
| **Projectile** | Curves in flight at `HOMING_RATE`, up to the budget |
| **Beam** | **The emitter angles.** The gun rotates up to the budget so the beam is a straight line from bow to target. Beams never bend |
| **Wave / width shape** | The wave's center axis rotates onto the target, up to the budget, and the wave spreads from that aligned axis |

CRYO I–III are the wave case: the blast is short and wide, so what the player needs is for
the wide part to be centered on something, not for the edge to curl.

This keeps one rank line meaningful across every ability in the game instead of being dead
value on half of them.

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

**Band ids stay short** — `r1`, `dis2`, `w1`, `t_quick` — because they are token values,
not variables.

Tooltips say **"short / medium / long" plus the unit count**; data says `r2`.

---

## Open questions

| # | Question | Blocks |
|---|---|---|
| 1 | **Every AoE ability is now worth 6–14× what the last revision assumed**, because baddie counts tripled. None of the authored damage numbers have been checked against this | Phase 4 |
| 2 | **Expanse at 6000²** — 6s to cross at base speed. A torus too large stops reading as a torus | M0 |
| 3 | **`HOMING_BASE`** — 0° or 1.0°? The ceiling is settled at 3.5°; only the free baseline is open | M0 |
| 4 | **`VALRUNE_SPEED_BASE = 1000`** is 2.3× v0's. Fast for a thumb-driven ship, and it drives the Expanse size and the projectile floor | M0 |
| 5 | **200 baddies at 75 units each** is a rendering and collision load worth proving early on a real mid-range phone | M0 |
| 6 | **Pricing `piercing`** — a 5× crowd multiplier at 4 ranks, and `guns × piercing × attack_speed` compounds to roughly 75× | Phase 4 |

O-21 is resolved: `CLUSTERING_BLIND = 1.85`, `CLUSTERING_AIMED = 2.50`, with decay by
coverage. Still a Balance Lab slider, but it is now anchored to a measurement instead of a
guess.
