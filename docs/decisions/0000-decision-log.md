# 0000 — Decision Log

**Status: authoritative.** Where any document in `docs/` conflicts with an entry here, this file wins until that document is revised in Phase 7.

This is the running index. Load-bearing entries get expanded into full ADRs (`0001-`, `0002-`, …) in Phase 1c. Every entry records what changed and why, so a decision is never silently re-litigated.

Legend: **[R]** reverses a v0 decision · **[N]** new, absent from v0 · **[C]** confirms v0

---

## Scope and content model

### D-001 [R] Triples are bespoke; coverage may be sparse
v0 (`04` §4.8, `08` §8.3, `12`) required triple nodes to be *composed* automatically from their constituent duals, and listed "hand-author a triple" as a CI-enforced NEVER. The ability spreadsheets hand-author 36 triples with unique names and mechanics.

**Decision:** triples are bespoke. Composition is deleted. A combination with no authored ability simply has no node; the coverage report measures gaps rather than demanding completeness.

**Why:** the composed model was scope control, not design. Composed triples would have been recolors of their duals, which is precisely the failure mode `10` §10.8 warns about.

### D-002 [R] Two ability slots per combination
v0 assumed one node per combination (129 total). Each combination has a **passive** and an **active** slot.

- 9 single passives + 36 dual passives + 36 dual actives + 84 triple passives + 84 triple actives = **249 slots**
- This is the denominator behind the tracker's 35.74%

### D-003 [N] Coverage targets
**50% of dual slots, 30% of triple slots.** At 50%, effectively every combination has one ability which is *either* a passive *or* an active. Additionally: **every base element appears in 1–2 triple combinations.**

Gaps are features, not defects. Reference point: Element TD ships 6 elements where each appears in 2–4 duals and 1–2 triples.

**Current state (counted from `data/source/abilities_table.csv`):**

| Scope | Filled | Target | Status |
|---|---|---|---|
| Duals, all 9 | 44 / 72 = 61% | 50% | above |
| Triples, all 9 | 36 / 168 = 21% | 30% | ~15 short |
| Base-5 duals | 12 / 20 = 60% | 50% | above |
| **Base-5 triples** | **2 / 20 = 10%** | 30% | **4 short** |

Base triple participation: Forge 2, Plasma 2, Cryo 1, Volt 1, **Caustic 0**. The only content blocker is four base triple abilities, at least one containing Caustic.

### D-004 [C] Base vs DLC element split holds
Base: PLASMA, CRYO, FORGE, VOLT, CAUSTIC. DLC: CHRONO, GAMMA, VOID, ETHER. The base five must carry a complete, winnable game on their own; gaps get authored before any ability is coded.

### D-005 [R] AETHER is renamed ETHER
"Aether" reads as old fantasy. The setting is modern-and-later.

---

## Combat model

### D-006 [R] All slotted elements apply to all beams
v0 (`04` §4.4) gave each gun its own element with an asymmetric 1.1 / 0.5 / 0.5 split, per-gun matchup, crit cross-propagation, and a level-3 mono-attunement bonus. **All of that is deleted.**

**Decision:** guns buy raw damage and attunement slot count. Every slotted element applies to every beam.

### D-007 [N] Damage packet model
A **damage packet** carries one type set and resolves the type chart **exactly once**. A gun-shot is one packet; a node's flat bonus damage is a *separate* packet with its own type. Simultaneous, not nested.

Multi-type packets blend: `multiplier = mean(multiplier_per_type)`. Three Plasma attunements against a Plasma-weak target get the full 1.30×; a mixed loadout gets a blend. This reproduces v0's matchup hedging without per-gun complexity.

### D-008 [C] Crit applies only to unmodified base damage
From `misc_ideas` line 23. Crit multiplies the gun packet only. Ability bonus damage is a separate, uncritted packet. Falls out of D-007 for free.

### D-009 [R] Statuses apply *after* damage
Reverses `misc_ideas` line 11. Per-status `applies_before_damage` flag for exceptions.

**Why:** an ability that amplifies its own hit makes every stack interaction order-dependent and balance untestable. It also reads wrong — players see the damage number before the debuff icon. Industry norm (WoW, PoE, Diablo, most MOBAs). The launch-time-application concern is moot because everything resolves at impact, not at fire.

### D-010 [R] On-crit belongs to the attunement, not the node
Per `attunement_table.csv`, which gives every element both an on-hit and an on-crit effect. v0 put on-crit on nodes.

### D-011 [R] Surge is deleted
Every reference removed. It existed only to give tier-3 nodes a second trigger layer, a job the I/II/III tier text already does.

### D-012 [R] High fire rate, low crit
Fire rate rises to roughly **3.0 → 15.0 shots/sec** (v0: 2.0 → 8.0). Crit chance falls to roughly **1–10%** (v0: 5–30%). The PRD constant table must be regenerated for the new range.

At 15 shots/sec and 10%, crits land ~1.5×/sec late-game and about once per 7 seconds early, which keeps an early crit a genuine event.

### D-013 [R] Enemy density down
~**30 baddies** in a typical encounter, **75** in a deliberate zerg-rush. v0 said 60 simultaneous and 40–140 per contract.

### D-014 [R] Flat values over percentages
From `misc_ideas` line 24. Percentages survive only on an explicit allowlist: **crit chance, crit damage, Overclock**. The allowlist is closed; additions require an ADR.

---

## Statuses, immunity, avoidance

### D-015 [N] Immunity and override are a framework
Statuses carry `polarity` (buff/debuff) and **tags**: `control` (stagger, paralyze, stasis, gravity), `dot` (burn, corrosion), `vulnerability` (poison, maxhp_loss), `avoidance` (ethereal, evasion), `movement` (push, pull, slow).

- **ImmunitySet** on any entity — `(status_id | tag)` entries, **derived from tags** with per-status overrides, grantable per-placement in wave data
- **OverrideSet** on any effect source — what it pierces, populated by the `Override` statuses
- Resolution: `can_apply(status, target, source) = !target.immune_to(status) || source.overrides(status)`

Replaces 90 hand-maintained booleans with about four rules. **Buffs carry no immunity data** — a boss having evasion is an enemy/wave-data decision, not a status property.

Wave authors gain per-baddie immunity without new archetypes.

### D-016 [N] Threat-class progression rule
Was implicit in the `buffs_debuffs` immune columns; now canon.

- **Control debuffs never land on bosses or minibosses, in any form.** The `+` form only unlocks elites.
- **Damage and vulnerability debuffs pierce everything in their `+` form.**

So investing an element to level 3 makes your damage-over-time bite bosses, while crowd control never will. Boss fights stay authorable; element investment visibly pays off against them.

### D-017 [N] Evasion stays probabilistic, fixed by scope
Evasion applies **only to basic projectile shots**. Never AoE, never a baddie's special ability.

- `evasion` / `evasion+` — dodge minion and elite basic shots
- `evasion++` — extends to miniboss and boss basic shots (mirrors D-016)

**Why not deterministic charges:** a timed avoidance charge is gameable. Against a boss telegraphing every 8–15s, a 5s recharge nullifies *every* attack — 100% mitigation dressed as determinism.

**Why the variance is acceptable:** scoping confines evasion to high-volume chaff fire, where 15–35% converges within seconds, satisfying `02` §2.1's variance law. Boss damage arrives via telegraphed specials evasion cannot touch, so it never decides an outcome.

**Depends on** `basic attack` vs `ability` being crisply defined on the baddie side — a Phase 1a blocker.

### D-018 [N] Death is a policy, not a special case
HP goes negative internally; clamped only for the HUD bar. `death_check_policy` enum:

- `continuous` (default) — dead when HP ≤ 0
- `deferred_to_snapshot` — HP may sit negative; death evaluated only at the snapshot tick (Preservation II)

Any future ability altering death conditions extends the enum rather than special-casing.

---

## Systems

### D-019 [R] No global game-speed manipulation
The Chrono+Cryo "slow time" passive is **cut**. Its thumb-lift trigger collides irreconcilably with `03` §3.2.4 ("recall is unavailable while holding the movement stick" — every recall tap would trigger it).

Tracing every Chrono effect in the spreadsheets, that ability was the **only** consumer of a global clock. What remains is three ordinary mechanisms:

- per-target speed multipliers (`slow` / `slow+`)
- a deferred-damage queue (Delay the Present, Thermal Lag)
- personal cooldown-rate multipliers (Ghost Frame, Overclock, Capacitor, Pocket Universe)

This deletes the highest-risk system in the design at no cost to anything else. The vacated slot returns to the gap report. When re-authored, `paralyze` applied to all non-boss baddies reproduces the time-stop fantasy from existing parts, including the boss immunity.

### D-020 [R] Drones replace "extra ship slots"
Undamageable and untargetable. Behavior:

1. Fly to within `r1` of the nearest baddie and shoot
2. Hold that target until it dies, then retarget
3. Try to stay within `r2` of the player
4. If the player exceeds `r2`, return to the player's `r1` ring, then retarget

May carry a lifetime and a cooldown. **Hologram becomes a drone type** and loses its HP and lethal-hit absorption.

### D-021 [N] Systems confirmed in scope
Persistent field/terrain objects (mines, debris, fissures, zones, portals, trails) · player-side statuses (ethereal, evasion, stasis, untargetable, invisible, damage-immune) · healing and HP regeneration · drones · the immunity matrix.

**Out of scope:** global game-speed manipulation (D-019).

---

## Progression

### D-022 [N] Five loadout slots
**3 passive + 1 active + 1 auto-active.** The active slot is freely toggleable to auto. Auto-active unlocks after **Miniboss 2**.

An auto-fired active uses your **current heading**. Directional abilities (solar beam, comet, spectral lance) lose most of their value on auto; self-centered and auto-targeting ones (mines, neutron stars, taser, lightning storm) lose nothing. The toggle is a real per-ability decision, not a strictly-better button.

Requires an `auto_fire_efficiency` parameter per ability.

### D-023 [R] Maximum cooldown is 2 minutes
Supersedes `misc_ideas` line 36 (1 minute). Matches the balance window, so a 2-minute ability fires exactly once per measured window. Cooldown becomes an explicit power lever.

---

## Geometry and constants

### D-024 [N] Playfield is 1000 units wide
Height derives from device aspect (~2200 on 20:9). Wormhole mode is exactly one screen, so `r1 = 120 units` reads as 12% of screen width and AoE hit count becomes computable:

```
expected_hits = density × (π·r² / playfield_area) × clustering
```

### D-025 [R] Movement splits into push and pull
A single `mr` band conflated two different behaviors (`misc_ideas` line 15 describes the tension).

- **`push_impulse`** — displacement in units over a fixed short window (~0.08s). Does **not** override the target's own movement
- **`pull_speed`** — sustained units/sec toward a source. The `gravity` status is what makes it override movement

### D-026 [C] Banded geometry constants
`r_short` / `r1` / `r2` / `r3` · `dis_short` / `dis1`–`dis4`, where `disN` matches `rN`'s reach · `w1` / `w2` · `TICK_DOT = 0.2s`. Tooltips say "short / medium / long" plus the unit count; data says `r2`. Band values proposed in Phase 1b.

---

## Balance method

### D-027 [N] Three metrics, no single power score
**Offense, defense, utility**, kept separate. Parity is checked per metric at **±15%** (v0 said ±25%).

### D-028 [N] Offense is a 120-second window
```
uses    = ceil(120 / (duration + cooldown))    # 120s cd → 1, 60s → 2, 30s → 4, 10s → 12
offense = actives:  uses × damage_per_use × auto_fire_efficiency (if auto-slotted)
          passives: continuous rate × uptime × 120
```
First use at t=0. `ceil` counts uses *starting* within the window, avoiding the artifact where a 120s ability appears to fire at both t=0 and t=120.

### D-029 [N] Defense resolves to effective HP
```
effective_mitigation = Σ (damage_share_of_category × mitigation_in_category)

EHP = (HP + shield_charges × avg_hit + regen × 120)
      × avg_hit / (avg_hit − bulwark_flat)
      / (1 − effective_mitigation)
```

Avoidance is `1/(1−m)`, **not** `1+m`. 35% dodge is **1.54× EHP**, and mitigation stacks with increasing returns — the scaling risk worth watching.

Because evasion only touches basic shots (D-017), real value depends on composition: against a profile that is 40% basic fire, 35% evasion yields 14% effective mitigation and only **1.16× EHP**.

### D-030 [C] Tier parity targets
From `misc_ideas` line 29:

```
tier III single  =  tier III dual  =  tier II triple
tier I triple    =  tier II dual   =  tier II single
```

Sole exception: a tier I single may be weaker than any other tier I, because it is early-game and cheap.

### D-031 [N] Threat profiles carry damage composition
Boss (1 target) · Miniboss+adds (3) · Normal (~30) · Zerg (75). Each also carries **incoming damage composition** (share arriving as basic shots, AoE, specials, contact) and **average incoming hit size**.

Without these, evasion, rime, bulwark, and shields cannot be priced at all. Both are **guesses until M0 exists** and must remain editable inputs, never constants.

---

## Process

### D-032 [C] Repo is the source of truth
Design data lives in the repo as CSV/JSON — git gives diffs, review, offline access, and CI validation. A sync script allows bulk editing in Google Sheets and pulling changes down. GCP's real work starts at M5; early exceptions are Firebase Hosting for the dashboard and gap report, and optional BigQuery snapshots for trending.

### D-033 [N] The Balance Lab never writes
`balance/balance-lab.html` is a read-only sandbox with "reset to data" and "copy params as JSON". Parameter edits are exploratory. The agent makes the actual data edits.

### D-034 [N] Repo renamed to `valrune-shooter`; local folder deliberately not
GitHub repo and git remote are `valrune-shooter`, matching the GCP project.

**The local working directory stays `C:\Users\bryan\valrune-invaders`.** This mismatch is
intentional, not an oversight — do not "fix" it. Cursor keys chat history and workspace
state to the folder path, and renaming it would strand this project's history for a purely
cosmetic gain. Git is entirely unaffected.

---

---

## Canon review — Phase 1a (Bryan's pass on `docs/14-CANON.md`)

### D-035 [N] American English throughout
defense, offense, behavior, color, armor, center. Project files converted; v0 docs in
Phase 7.

### D-036 [R] `player` is the human, `valrune` is the ship
Reverses `misc_ideas` line 10, which had `player` = ship and `user` = human. Every engine
and code sample uses "player" for the human, so inverting it guarantees slips that are
invisible until they are bugs. **`user` is banned.**

### D-037 [R] Debris is threat class 0, and it acts
Replaces the proposed inert "hazard". Debris drifts on a heading, meteors cross fast, a
derelict gun spins and fires. It takes archetypes and role tags like any baddie; class 0
governs only immunity and scaling. Ladder: **Debris, Minion, Elite, Miniboss, Boss.**

### D-038 [N] Archetype id is descriptive; display name is separate
The two were fused, so renaming the faction would have meant renaming code. Now:
`drifter, weaver, charger, splitter, turret, shielder, miner, spawner`. Data references the
id forever; `locale/` carries whatever the player sees.

**The v0 fiction names are discarded** — Mote, Lash, Spike, Husk, Anchor, Warden, Seeder,
Choir are gone. The id doubles as the display name for now, which is more descriptive
anyway. Writing real fiction names later costs a `locale/` entry and nothing else.

### D-039 [N] Affinity splits into attack and defense
`attack_affinity` (types dealt) and `defense_affinity` (types taken). v0's single field
made every baddie symmetrical — a baddie can hit with plasma while armored against cryo.

### D-040 [R] `beam` means a continuous weapon effect
Not a projectile stream. A beam impacts, then damages on ticks. **No hitscan-instant
damage path is needed anywhere**, which removes an architecture branch. `lance` rejected.
**`volley` banned** — synonym for gun-shot.

### D-041 [N] Baddies and the Valrune share weapon vocabulary
No separate baddie terms. Both have a **basic attack**; the Valrune's is a gun-shot, a
baddie's may be a gun-shot, a spit, or a beam. Rules reference `is_basic_attack` and never
ask who fired. Only `fire_rate` stays player-specific.

### D-042 [N] Unbounded parameters are `null`
`lifetime: null` = until the contract ends. `cooldown: null` = not applicable. Never
sentinels like `-1`. The parameter always exists; its value may be unbounded.

### D-043 [N] Drones are non-collidable with everything
Not cover for the player, not an obstacle for baddies. Same-faction projectiles pass
through as if absent. `faction: MERCENARY | UNFORMED` (D-061). Types so far: `hologram`,
`undead`.

### D-044 [R] Everything purchasable is a rank line
**Rank tree / rank line / rank**, replacing stat tree / upgrade line / rank.
**`element_level` becomes `element_rank`.**

**An element is a rank line with 3 ranks priced in element points.** One `currency` field
(`credits | element_points`) collapses two parallel systems. The only asymmetry is
downstream — element ranks feed node tiers — which is a consumer, not a data difference.

**Rank is a purchased step. Tier is a node's I/II/III.** Never interchanged.

### D-045 [N] Empty nodes are never shown to the player
The Hangar shows only nodes with authored abilities. No locked cells, no empty grid
positions, no "coming soon". This is what makes sparse coverage (D-003) read as complete
rather than unfinished.

### D-046 [N] Abilities carry a delivery mode and travel-end conditions
`delivery`: `homing` (retargets if the target dies mid-flight), `skillshot`, `fixed_point`,
`self`, `attached`. `travel_end` is a **list** — `on_hit`, `on_distance`,
`on_target_reached`, `on_lifetime` — whichever fires first triggers the arrival spec.
Absent from the v0 model entirely.

### D-047 [R] Full words for variables in code
`duration`, `cooldown`, `radius`, `stacks`, `damage`. The `misc_ideas` line 12 short forms
stay in the spreadsheets, where a column header supplies context; code has no column
header and `c` sits next to `charges`, `cooldown`, and `combination`.

**Band ids stay short** — `r1`, `dis2`, `w1` — because they are token values, not
variables. `radius: "r2"` reads correctly.

### D-048 [N] One universal tick
```
TICK = 0.2s          # every DoT, zone, beam, periodic anything
TICK_FAST = 0.1s     # strict subdivision only
```
Buys determinism (one batch, one frame, stable order — independent timers drift and make
replays diverge), legibility, and performance. Never a value like 0.15s that aligns with
nothing.

### D-049 [N] All probability is PRD
No bare `randf() < chance` anywhere. Chance climbs on failure, resets on success, so
streaks are impossible in both directions and the long-run rate matches nominal.

**PRD state is per (entity, effect), never shared** — one entity's luck must not drain
another's.

Mean EHP in D-029 is unchanged; only variance tightens. This *strengthens* D-017, whose
concern was slow convergence.

### D-050 [C] A crit is not a source, but what a crit spawns is
Refines `misc_ideas` line 24. "When a source causes X" does not fire because a hit
critted — but VOID's on-crit gravity field is an object, and that object is a source. The
event is not a source; the thing it creates is.

### D-051 [N] Statuses have families, so `+` forms never get lost
`burn` and `burn_plus` share `family: burn`. Every ability condition references the
**family**, never a bare status id — "damages baddies with burn" always means burn+ too.
Referencing a bare status id in a condition is a **validator error**, because it is nearly
always this mistake.

### D-052 [N] Element immunity cascades to that element's statuses
A boss immune to plasma takes no burn and no burn+, with no per-status authoring. Makes
`owner_element` functional rather than documentation. `owner_element: null` is legitimate
for universal statuses (shield, stasis, paralyze, MaxHP_Loss, Override) — **there is no
`Misc` element.** Any entity may also carry bespoke immunities per placement in wave data.

### D-053 [R] Recall moves on the Y axis only
The ready line is a **full horizontal line**, not a point. The Valrune drops straight down
holding its X, rotating smoothly toward north in transit, finishing the rotation on arrival
if the turn rate is too slow. v0's "bottom-center" would have yanked the player sideways
and made recall a repositioning tool rather than a reset.

### D-054 [N] Band ladders
Radius and distance couple only at the short end:
`dis_short = r_short`, `dis1 = r1`, `dis2` independent, **`dis3` = ready line to the top
border** (the one band with physical meaning), `dis4` beyond it for wrapped angled shots
and the open arena.

Widths are Valrune-relative: `w0` projectile width, `w1` half the ship, `w2` full ship,
`w3` two ships plus half a ship each side. Both ladders extend rather than special-case.

### D-055 [R] Gravity disables self-movement; pull becomes the only movement
Replaces D-025's "pull overrides movement", which implied arbitration. Bryan's model has
none: nothing competes, so nothing resolves.

Push and pull are `(distance, time)` **bands** — `push_1`, `pull_2` — keeping "short speedy
Forge knock" and "slow heavy Void drag" as named constants. **`pull_stop_radius`** ends a
pull at a ring while gravity keeps holding the target.

### D-056 [R] `stage` and `throat` are banned
**Contract** is the playable thing; playing it is just playing it. A single attempt is a
**run** (`ContractRun`), for save data and analytics only. **Wormhole**, never "throat".
Also banned: `level` (already means element rank), `mission`.

### D-057 [N] Bounty — a fixed pool divided across the roster
Credits are awarded per kill, but the **total is set by the contract, not by the kills**.

```
pool          = f(contract_index)                        # index-anchored, as v0
baddie_bounty = pool / Σ weights
                × weight(threat_class)                   # elites, minibosses, bosses worth more
                × random(0.85, 1.15)                     # cosmetic spread, like damage
```

Paid only on **reaching a checkpoint or completing the contract**. Clause bonuses apply
to the total at the end.

This resolves O-07 rather than trading against it, which is what the three options I
posed had assumed was necessary:

- **Farming is impossible** — no reward without a checkpoint or a clear.
- **The cap is structural** — you cannot kill more baddies than the contract contains, so
  the pool is the ceiling. No separate cap rule is needed.
- **Clearing fast costs nothing** — the pool does not depend on time.
- **Escapees genuinely cost their share**, giving the wormhole's bottom edge a consequence
  without a second fail state and making `spawner` a soft DPS check.

The `random(0.85, 1.15)` spread is a **range roll, not a probability** — PRD (D-049)
governs binary chances, not continuous ranges. Damage rolls the same way.

Open sub-questions, none blocking: exact threat-class weights (Phase 4, needs the balance
engine), and whether bounty banked before a checkpoint survives a later death.

### D-058 [R] `bulwark_percent` is deleted; flat mitigation only
Resolves O-08. Percentage damage reduction is the worst offender for D-029's
increasing-returns problem — it multiplies with evasion, rime, and shields — and it sat
outside D-014's allowlist.

**The rank tree is now 13 credit lines, not 14.** Defense keeps `hull`, `shield`,
`shield_recharge`, `bulwark_flat`.

Two consequences for Phase 4, neither blocking:

- v0 priced this line at **3,500 credits**, so the tree's total sink drops from 30,477 to
  ~26,977. A campaign clear paying ~18,150 goes from 60% to ~67% of the tree, making the
  economy meaningfully more generous. Retune the pool or extend other lines.
- `bulwark_flat` has only 8 ranks and now carries mitigation alone. It likely wants more
  ranks to absorb the deleted budget. Exact count waits for the balance engine — inventing
  it now would be a guess with no way to check it.

The allowlist is unchanged and now genuinely closed: **crit chance, crit damage,
Overclock.** Those are legitimately probabilistic or multiplicative by nature.

### D-059 [R] The Field is a wrapping rectangle
Resolves O-09. v0's circular arena with elastic pushback is replaced by a rectangular
arena wrapping on both axes.

Deletes the elastic pushback system, the boundary shader, and the HUD boundary arrow.
Wrapping becomes the universal movement rule — the wormhole is simply the case that clamps
Y for design reasons, not a second model to learn.

**A minimap ships with it**, small, with dots for baddies. It must be **drawn
torus-aware**: a baddie near the right edge is also near the left edge, and a minimap that
does not show that is worse than none.

Costs accepted: disorientation without landmarks (parallax and the minimap mitigate), and
fleeing a boss no longer works because it always comes back around.

Arena dimensions in screens are a Phase 1b number.

### D-060 [R] `homing` replaces the `spread` rank line
Each rank grants **N degrees of correction** — a projectile may curve that far off its
launch heading to intersect a baddie. At 15 degrees you no longer have to be dead-on,
which matters given that you rotate and strafe simultaneously.

**Why it is the better line:** spread made your shots less accurate as you bought it, so
it changed playstyle rather than improving anything. Homing makes the control scheme
forgiving, and `03` §3.2.3 notes players value control improvements disproportionately.

Three consequences:

- **The stat is not `delivery: homing`** (D-046). The stat is a one-shot correction toward
  whatever is in the cone at launch. The ability mode retargets mid-flight.
  **Basic-attack projectiles never retarget** — thousands of tracking loops is a real
  frame cost for an imperceptible benefit.
- **Spread also drove visuals** — helix amplitude at 2 guns, fan angle at 3 (v0 `04`
  §4.4.4). Those become fixed values, arguably better: the helix becomes a signature
  rather than a setting.
- **Overlaps Assist Aim** (`03` §3.2.6). See O-14.

Rank count and degrees-per-rank are Phase 1b numbers.

### D-061 [R] The factions are the Mercenaries and the Unformed
`faction: MERCENARY | UNFORMED`, replacing the placeholder `PLAYER | BADDIE`. Grounding the
enum in fiction rather than in role means a third faction later costs nothing.

**Renamed from `HORROR` to `UNFORMED`.** The gain is that it doubles as the in-fiction
collective noun for every baddie, which `Baddie` never was — that stays a code and design
word (D-036). The game can now say "the Unformed are massing in Sector 3" where it
previously had nothing to say, and the name carries the theme: things that never resolved
into a fixed shape.

**"The Hollow" is deleted.** The Unformed is eldritch-plus-technology: organic ones are
collision-focused and may die on impact or ignore it entirely; augmented ones shoot, spit,
or beam. The proposed `chassis: organic | augmented` field drives fiction and behavior
together.

Debris sits in `UNFORMED` [OPEN] — fictionally odd for an asteroid, but faction only decides
what can damage what, and a third `NEUTRAL` faction would earn its keep only if debris
needs to hurt both sides.

---

## Constants — Phase 1b (`docs/15-CONSTANTS.md`)

### D-062 [C] Push and pull are both `(distance, time)`
Upholds D-055 against my proposed split. One is displacement away from a source, the other
toward it, and **neither stops the target's own movement** — each contributes a velocity
that adds to whatever the target was already doing. **Gravity is the status that disables
self-movement**, and only then is a pull the target's entire motion.

Push windows are short (0.08–0.20s) because a push is one impulse. Pull windows are 1.0s
because a pull repeats while the status lasts, so the pair reads as a rate.

Bands are calibrated against a minion at 120 u/s. **They do not transfer to the player at
1000 u/s** — anything pulling the Valrune must pair with gravity to matter.

### D-063 [R] Assist Aim is cut; the settings stay
Dissolves O-14 rather than resolving it. The v0 accessibility list conflated two things:

**Kept** — ordinary settings every shooter ships, hours of work each: screen shake 0–100%,
flash reduction, haptics toggle, colorblind palettes, handedness swap, stick size, rotation
sensitivity. Flash reduction earns its place regardless of framing, since photosensitivity
is the one area storefronts and some regions actually care about.

**Cut — Assist Aim.** The only item with real cost (auto-targeting logic, a priority rule,
and an interaction with every ability that picks targets), and **`homing` now does its job
better**: a player wanting more forgiveness buys homing ranks, making it a progression
decision rather than a menu toggle. There is no longer any question about selling a
purchasable version of an accessibility feature, because the feature is gone.

### D-064 [N] Vertical geometry is sized against the minimum device, not the reference
Width is fixed at 1000 and height varies with aspect — the correct way round, since fixing
height would rescale the ship and every radius between phones and balance would not
transfer.

All vertical constants are sized against **2000** (18:9, minimum supported), not 2200
(20:9, reference). A taller phone gets bonus sky, a mild advantage not worth correcting.
Sizing against the reference would push content off-screen on an 18:9, which is a real
failure.

**The ready line is 360 units from the bottom**, not a percentage — a percentage drifts
with aspect and puts the ship under the player's thumb on short screens.

### D-065 [N] `dis4` anchors the distance ladder; rungs added beneath it
Resolves O-19. Ladder: `dis_short` 60, `dis1` 120, `dis2` 400, `dis3` 800, **`dis4` 1600**,
`dis5` 2400.

`dis4` keeps the physical meaning — 2000 minus 360 for the ready line leaves 1640, so 1600
reaches the top of the action zone.

**Existing content remaps mechanically in Phase 2**, counted across the spreadsheets:
`distance_1` (8 refs) → `dis1`, `distance_2` (19) → `dis2`, `distance_3` (9) → **`dis4`**.
Solar beam annotates `distance_3` as "(max distance)", so the rename follows the authoring
rather than reinterpreting it.

**`dis3` at 800 is deliberately unused** — the rung requested for future need. Spectral
Lance is the likely first customer, since it fires to max and pays bonus "beyond
distance_2", for which 800 is a better threshold than 400.

### D-065a [N] Four radius rungs is correct, and `r1` is the game's most sensitive number
Counted across the ability, attunement, and status tables: `r_short` 4 references,
**`r1` 51**, `r2` 29, `r3` 7. Nothing reaches for a fifth rung, so the ladder stays at four.

**Fifty-one abilities read against `r1`.** Moving it from 120 to 150 silently buffs a fifth
of the content. It changes only deliberately, and the balance engine should report its
sensitivity.

### D-065b [N] `w0` is deleted; the width ladder starts at 1
A projectile's width is not a band anyone authors against — nobody writes `width: w0`. It
is a property of the projectile and lives in entity sizes as `PROJECTILE_WIDTH`.

Ladder: `w1` 40 (half Valrune), `w2` 80 (full), `w3` 240 (three), **`w4` 400** — the CRYO
answer, since its wave blast trades reach for width and needs a rung rather than a special
case.

### D-066 [N] `CLUSTERING` is a balance input, not a constant
```
expected_hits = density × (π r² / playfield_area) × CLUSTERING
```
Uniform distribution understates every AoE, because baddies arrive in waves and converge
on the player. A Wormhole wave front clusters far harder than an Expanse scatter, so this
lives in the balance engine as a per-profile tunable. `2.0` is a placeholder and **every
AoE valuation rides on it** until M0 measures it.

### D-067 [N] The action zone — baddies spawn off-screen but act in view
> Baddies spawn above the visible area, but **every entry and first action happens inside
> the bottom 2000 units.** Nothing sits on a border. Traversing outside is fine.

A baddie spawns at 2300, drifts down, and cannot telegraph, shoot, or turn until it is at
or below 2000. `SPAWN_MARGIN = 300` is the runway it needs to be moving at speed before it
becomes the player's problem, rather than materializing at the edge.

### D-068 [R] 16:9 letterboxes rather than reshaping the game
Resolves O-16. Designing all vertical geometry against 1778 would cost every modern player
10% of their vertical space for a market slice that is now mostly pre-2017 devices and
tablets. Letterboxing costs thin bars on rare hardware and guarantees **identical gameplay
everywhere** — a shorter action zone would change every timing and every band, and balance
measured on one device would not hold on another.

### D-069 [R] Speed ranks are flat, and the ship is much faster
v0 had thrusters and gyros at `+0.15× base` and velocity at `+8%` — percentages, none on
D-014's allowlist. All three become flat per rank.

```
VALRUNE_SPEED_BASE = 1000 u/s   20 × +40   → 1800    # 1.0s to cross screen width
ROTATION_BASE      =  420 °/s   20 × +15   →  720
PROJECTILE_SPEED   = 2400 u/s    5 × +160  → 3200
```

The ship is **2.3× v0's 440 u/s**, which is what makes a 6000-unit Expanse crossable in 6
seconds instead of 14.

**Rotation starts faster and tops out lower**, as requested: v0's 300 → 1200°/s put maximum
rotation at 3.3 turns per second, past the point of usefulness.

**The projectile-to-ship ratio is a constraint, not a coincidence.** At 2.4× the player
outruns nothing. If it narrows, shooting while advancing feels wrong — you chase your own
bullets, forward shots crawl and backward shots race. Preserve the ratio if either number
moves.

### D-070 [R] `crit_chance` runs 1.0% → 4.0%, 12 ranks × +0.25%
At 10% and 15 shots/sec you get 1.5 crits per second, which is texture rather than a
critical hit. The ceiling drops to **4%** — one crit every 1.7s at max fire rate.

**Crit frequency is `fire_rate × crit_chance`, and both scale up together**, so frequency
scales multiplicatively and the floor cannot be set independently of the ceiling. v0 swung
24× across the campaign. A 0.05% floor would swing 400× and leave a new player **11
minutes between crits**, making every attunement crit effect invisible content for the
early game — the exact failure the free baseline exists to prevent.

Locked at **1.0% base, 12 ranks × +0.25%, max 4.0%**: one crit per 33s at the start, per
4.4s mid, per 1.7s at max. Rare enough early to feel like an event, frequent enough that
crit-triggered nodes are live from the first contract.

A crits-per-second denomination was considered — deriving per-shot chance as
`target_rate / fire_rate`, which would have flattened the curve entirely — and rejected in
favour of keeping the line legible as a percentage. The 20× campaign swing is accepted.

**Consequence for the Balance Calibrator:** on-crit attunement effects fire 0.6×/sec at the
top end and must be valued against that, not against the rare-event framing implied by the
early game.

PRD table regenerated for 1–4%; v0's covers 5–30% and is useless. At these chances PRD
needs a long tail — the shot counter runs into the hundreds between crits.

### D-072 [R] Velocity inheritance cut; projectile speed gets a floor rule instead
Briefly adopted at 25%, then dropped. Projectiles do not inherit ship velocity. The rule
that replaces it:

```
PROJECTILE_SPEED must exceed VALRUNE_SPEED at maximum thrusters, with margin.
```

The worst case is maxed thrusters against an untouched velocity line. At 2400 that is
1.33× — technically satisfying the rule while still feeling wrong, since bullets pull ahead
at only 600 u/s and appear to hang in front of the ship. **Base moves to 2800** (max 3600),
making the worst case 1.56× and the normal case 2.8×.

### D-073 [N] `wingspan` and `fuselage` enter canon; the Valrune is 100 × 110
Measured rather than guessed: a reference screenshot at 400px width converts at 2.5 units
per pixel, and the ship read well at 40px.

```
VALRUNE_WINGSPAN = 100    port to starboard
VALRUNE_FUSELAGE = 110    bow to stern, 55 either way from center
BADDIE_WINGSPAN  =  75    the standard baddie
BADDIE_MIN       =  50    nothing smaller; no maximum
```

The two new nouns join `bow`/`stern`/`port`/`starboard` and give the ship real geometry, so
"55 units forward of center" has a name instead of being restated.

All bands rescale: radii to 100/150/250/400, widths to 50/100/200/400.

### D-074 [N] Radii measure from center, distances from the bow
A projectile spawns at the bow, so a distance should be literally how far it travels. The
near distance rungs carry a 55-unit offset so they finish flush with the matching radius
ring, letting `dis_short` and `r_short` cover the same forward ground:

| Band | From bow | Reaches, from center |
|---|---|---|
| `dis_short` | 45 | 100 = `r_short` |
| `dis1` | 95 | 150 = `r1` |
| `dis2` | 345 | 400 = `r3` |
| `dis3`/`dis4`/`dis5` | 800 / 1600 / 2400 | — |

Odd numbers at the low end are the point; round numbers at the high end are fine because
55 units is under 7% of 800.

### D-075 [N] No push/pull band ids — compose existing ladders instead
Push and pull each want a distance *and* a time, and pull additionally wants to speak in
radii. Banding three dimensions explodes; letting each ability invent its own numbers makes
parity checking impossible. So neither:

```
push: { distance: <distance or radius band>, time: <time class> }
pull: { distance: <distance or radius band>, time: <time class> }
pull: { from: <radius band>, to: <radius band>, time: <time class> }
```

`push_1`/`pull_2` are deleted. Distance draws from ladders that already exist; only **time**
is new, and it needs four values rather than a matrix: `t_snap` 0.08s, `t_quick` 0.25s,
`t_steady` 1.0s, `t_slow` 2.5s.

The third form is the gravity well — pull inward until within the inner radius, then hold —
which is what most gravity abilities actually want.

### D-076 [N] Homing is a distance-denominated budget, bounded by `dis4`
> **Numbers superseded by D-080** — the ceiling is 3.5° and the rate 1°/100u. The
> distance-denomination principle below still holds and is the load-bearing part.
```
homing       = total angular correction budget, 2° → 18°
HOMING_RATE  = 6° per 100 units travelled
HOMING_RANGE = dis4 (1600); beyond it, nothing homes
```

**Denominating the rate per unit travelled rather than per second makes velocity carry turn
rate automatically.** A projectile's turning circle is `speed ÷ angular_rate`, so a faster
projectile at fixed degrees-per-second turns wider and homes worse — buying velocity would
quietly sell accuracy. Per-unit-travelled holds the circle constant exactly, so a 2800 and a
3600 u/s projectile trace an identical curve. No separate turn-rate line, no coupling
constant, and the M0 watch item from D-072 cannot occur.

The budget only becomes reachable past ~300 units, which is correct: a 75-unit baddie
subtends 20.6° at 100 units but 1.34° at `dis4`, so homing matters where the geometry is
cruel and there is distance there to spend it.

**The 18° ceiling looks too steep** — at `dis4` it reaches 520 units sideways, over half the
screen width, from a badly aimed shot. 12° is more defensible. Left generous so M0 dials
back from "too much" rather than guessing upward.

### D-077 [N] Homing spends its budget differently per delivery
"Bending" only fits things that fly. Same budget, applied at fire time for the rest:

| Delivery | How correction is spent |
|---|---|
| Projectile | Curves in flight at `HOMING_RATE` |
| **Beam** | **The emitter angles** so the beam stays a straight line from bow to target. Beams never bend |
| **Wave / width shape** | The center axis rotates onto the target and the wave spreads from there |

CRYO I–III are the wave case: short and wide, so the player needs the wide part centered on
something, not the edge curling. Keeps one rank line meaningful across every ability instead
of dead value on half of them.

### D-078 [N] O-21 resolved — clustering split into blind and aimed, decaying by coverage
Measured: **~170 baddies on screen, a semi-targeted `r1` hitting ~15.** Against the action
zone, `r1` covers 3.53%, so uniform expectation is 6.0 and the observed density multiple is
**2.5×**.

That number bundles two things the engine needs apart:

```
CLUSTERING_BLIND = 1.85    baddies clump on their own — self, fixed_point
CLUSTERING_AIMED = 2.50    plus the player choosing the thickest part — targeted
```

A nova centered on your own hull cannot pick its spot; a lobbed bomb can. One number would
overpay every self-centered ability in the game.

**Clustering decays as radius grows**, since a large AoE cannot beat the field average:

```
effective = 1 + (CLUSTERING − 1) × (1 − area_ratio)
hits = min(count × area_ratio × effective, count)
```

### D-079 [N] Expected baddie counts triple, revaluing every AoE in the game
```
COUNT_SWARM    = 200   hard ceiling, never more on screen
COUNT_STANDARD =  85   a normal wave, range 75–100
COUNT_TANK     =  20   fewer, larger, slower
```

**The previous file assumed 30 and computed that `r1` hits 1.2 baddies. It hits 7 in a
standard wave and 17 in a swarm.** Every radius ability is worth 6–14× what the earlier
numbers implied, while single-target abilities have not moved at all.

No authored damage number has been checked against this. **The Phase 4 engine has to run
before any AoE value is trusted**, and the AoE-versus-single-target parity rule (O-05)
becomes the most load-bearing open question in the balance work.

Also a performance claim: 200 baddies at 75 units needs proving on a real mid-range phone in
M0.

### D-080 [R] Homing capped at 100 units of lateral reach; targets the nearest baddie in line
The ceiling is set by its observable effect rather than by an angle. **At most 100 units of
sideways correction at maximum range**, and `atan(100 / 1600)` = 3.58°, so the cap is
**3.5°** and buys 98 units.

An 80% cut from the 18° carried the day before, and correct: 18° reached 520 units sideways
at `dis4`, over half the screen width, from a badly aimed shot.

```
homing       = 1.0° base, 5 ranks × +0.5°, max 3.5°
HOMING_RATE  = 1° per 100 units travelled
HOMING_RANGE = dis4
```

**Below ~5°, degrees and lateral units are interchangeable** — `tan` is near-linear, so each
half-degree is a flat ~14 units at max range. The line reads linearly either way, so
tooltips quote units.

`HOMING_RATE` drops from 6° to 1° per 100 units: at 6° the whole 3.5° budget would be spent
within 58 units of the barrel, reading as a snap rather than a curve. At 1° it takes 350
units.

**Homing acquires the nearest valid baddie along the line of fire and ignores everything
behind it.** No re-evaluation for a better target. This is what stops homing and piercing
from fighting — homing decides where the shot goes, piercing decides how far it continues.

### D-081 [N] New rank line: `piercing`, 0 → 4
A gun-shot passes through `piercing` baddies and continues, landing `piercing + 1` hits.
Zero at base.

**This is the structural answer to the gap D-079 exposed, and it works because it scales
with the same variable AoE does.** A flat damage buff would have helped single-target shots
everywhere, including against bosses where they were never weak. Piercing pays out only in
crowds — exactly where the gap opened.

Modelled as mean free path: a projectile sweeps a corridor `PROJECTILE_WIDTH +
BADDIE_WINGSPAN` = 83 units wide, so `gap = field_area / (count × 83 × clustering)`.

| Situation | Count | Mean gap | Hits at P=4 | Multiplier |
|---|---|---|---|---|
| Swarm | 200 | 67 | 5.0 | 5.0× |
| Standard wave | 85 | 158 | 5.0 | 5.0× |
| Tank wave | 20 | 672 | 2.4 | 2.4× |
| Boss, alone | 1 | — | 1.0 | 1.0× |

Full value in crowds, partial against tanks, **nothing against a lone boss** — where
single-target was already competitive, since an AoE hitting one target is just a worse
single-target hit.

Incidental finding worth keeping: **at 85 baddies a gun-shot currently dies 158 units out of
the barrel**, under a tenth of its range. That is most of why single-target felt weak.

No damage falloff per pierce — flat is simpler, consistent with D-014, and rank cost is a
cleaner lever than a decay curve nobody can feel. Piercing is a **basic-attack stat**;
abilities declare their own in data, and authored abilities granting piercing get reworked
against this line rather than stacking with it.

### D-082 [N] Crit rolls once per gun-shot, not once per pierced hit
Forced by D-081. Per hit, a maxed loadout reaches 15 shots/sec × 5 hits × 4% = **3.0
crits/sec**, five times the 0.6/sec locked in D-070. Rolling once per gun-shot preserves the
cadence exactly and reads better: the whole pierced line crits at once.

Attunement on-crit effects proc once per gun-shot for the same reason. **On-hit effects
still apply per hit** — spreading burn down a line of five is the point.

**`guns` × `piercing` × `attack_speed` is one compound axis, not three lines.** Three guns
each piercing four is fifteen hits per gun-shot; compounded with fire rate the offense
multiplier from base to maxed reaches roughly 75×, steeper than anything else in the tree.
This is the D-029 increasing-returns pattern and the Calibrator must treat it as a single
dimension.

**Piercing must be priced against the crowd case**, since at four ranks it is a 5× multiplier
for most of the game. Priced cheap, single-target guns become the answer to every encounter
and the AoE abilities are weak again — the same problem inverted.

---

## Statuses — Phase 1c

Full catalog and reasoning in [`16-STATUS-EFFECTS.md`](../16-STATUS-EFFECTS.md).

### D-083 [N] Stacks cap per family, not per status id
Only corrosion, poison, and maxhp_loss stack, all at 5. Per-id caps would make mixing a base
and `+` form a way to double the cap — nobody would author that deliberately and everybody
would find it. Each stack keeps its own magnitude, so 3 base + 2 plus deals `3n + 2y` rather
than averaging. A new stack resets the timer for all stacks in the family.

### D-084 [R] The tag taxonomy is six, not five — O-02 resolved
D-015 proposed `control`, `dot`, `vulnerability`, `avoidance`, `movement`. Tested against all
20 families, two failed and one had no members:

| Tag | Means |
|---|---|
| `control` | Restricts movement or action |
| `dot` | Damages the bearer on its own timer |
| `reactive` | Adds damage only when something else hits the bearer |
| `vulnerability` | Bearer takes more damage, or has less HP |
| `weakness` | Bearer **deals** less damage |
| `avoidance` | Bearer cannot be hit |

**`movement` deleted — it had no members.** Push and pull stopped being statuses at D-075;
slow is `control`. An empty tag means immunity authors track two names for one concept and
eventually forget one on a boss.

**`weakness` split from `vulnerability`.** Poison makes the bearer deal less; corrosion makes
it take more. Opposite directions, and one immunity should not silently cover both.

**`reactive` split from `dot`.** Burn ticks by itself; corrosion and shock fire only when
something else lands a hit. **Burn kills a fleeing baddie, corrosion does not** — tagging them
alike would make the engine value conditional damage as guaranteed.

**Tags on debuffs feed ImmunitySet; tags on buffs feed OverrideSet.** Same field, two
consumers, no overlap — which is how `avoidance` covers ethereal and evasion without
contradicting D-015's rule that buffs carry no immunity data.

### D-085 [N] `invisible` belongs to GAMMA — O-13 resolved
Your lean, with a mechanical argument behind it. Gamma owns `radiate`, which manipulates how
far effects reach; invisibility is the same idea inverted, manipulating whether you register
at all. And ETHER already owns both avoidance families — a third would make ETHER the answer
to every defensive question and leave GAMMA with no survivability at all.

### D-086 [R] `override` reworked — the base form did nothing
As authored, base override let **non-basic attacks** pierce `ethereal` and `evasion`. Both only
ever block basic attacks (ethereal by definition, evasion by D-017), so a non-basic attack
already landed against both. **It granted permission that was never withheld** — a purchasable
no-op that would survive playtesting, because nobody can perceive the absence of an effect
that never existed.

Moved one rung down rather than up:

| Status | Pierces |
|---|---|
| `override` | Your **basic attacks** pierce `ethereal` and `evasion` |
| `override_plus` | **Everything you do** pierces both families entire, including `evasion_plus_plus` |

The base now does the obvious job, and the `+` gains a unique capability, since
`evasion_plus_plus` is otherwise unpierceable by anything in the game.

### D-087 [N] The `Shield` status is renamed `ward`
It collided with the `shield` rank line, and the two are not variations on a theme: the rank
line is a **regenerating pool**, the status is **N charges each negating one hit entirely**.
They behave differently against one big hit versus many small ones, which is exactly what a
player needs to reason about.

**The collision produces wrong behavior rather than an error** — `entity.shield` is ambiguous
and both readings compile. The status is the cheaper side to rename, being referenced by a
handful of abilities rather than the whole progression tree.

### D-088 [R] `stasis` is a debuff
Marked a buff, but it is `paralyze` plus rotation lock plus passive suspension plus recall
denial — strictly worse than a status the same file calls a debuff.

**Polarity describes the effect on the bearer, not the intent of whoever applied it.**

This is load-bearing, not bookkeeping: D-015 says buffs carry no immunity data, so left as a
buff **nothing could ever be immune to stasis**, including the bosses the `control` ladder
exists to protect. The pause-all-durations clause stays as a property of the status.

### D-089 [N] Every status duration is a multiple of `TICK_FAST`
`stagger` at 0.05s and `stagger_plus` at 0.09s were both below tick resolution and would have
rounded 2–4× off intent. They become **0.1s and 0.2s** — still a stutter rather than a stun,
now deterministic.

The general rule matters more than the fix: an off-grid duration produces a final partial tick
whose damage depends on floating-point drift.

### D-090 [R] D-014's percentage rule restated as a principle
`slow` (0.85× / 0.65× speed) and `evasion` (15/25/35%) both violated the closed allowlist, and
**flat values are genuinely wrong for both** — a flat −18 u/s slow nearly stops a 120 u/s
minion and is imperceptible on a 1000 u/s Valrune.

Replacing the maintained list with a rule:

> Percentages are allowed where the quantity is **inherently a chance or a multiplier**. Every
> additive stat bonus is flat.

Covers crit chance, crit damage, Overclock, evasion, and slow without a list to maintain, and
still bans what D-014 was aimed at: `+8% velocity` rank lines that compound into increasing
returns.

### D-091 [N] Three catalog corrections
- **`maxhp_loss` base drops to `minion` reach.** It was immune on nothing, meaning the base
  form landed on bosses when by D-016 only `+` forms should. Almost certainly an oversight,
  since `maxhp_loss_plus` would otherwise be identical.
- **`damage_immune` decomposes.** It granted damage immunity *and* debuff immunity, the second
  being exactly what ImmunitySet does. It now grants damage immunity and adds every debuff tag
  to the bearer's ImmunitySet — same behavior, no special case, and "immune to damage but still
  debuffable" becomes authorable.
- **`invisible` grants `untargetable`** rather than restating the targeting rules, including
  the clause about launched abilities flying straight instead of homing.

### D-092 [N] The immune columns are generated, not authored
Auditing all 17 debuffs against D-016, **15 matched exactly** and the two exceptions are
addressed in D-091 and O-29. So the three immune columns are a **consequence of tag plus
form**, not data.

Phase 2 generates them. That removes 51 hand-maintained booleans and, more usefully, makes it
structurally impossible to author a boss that is accidentally stunnable.

### D-093 [N] Rider statuses have fixed durations; payload statuses take them from the source
Most games put duration on the source — WoW, MOBAs, tower defense all do. **But in all of them
each source applies its own private debuff**, so durations can differ freely. Valrune has *one*
`burn` applied by fifteen abilities, which is the case where source-defined durations break:

- **The refresh rule becomes incoherent.** Reapplication never shortens (canon §8), so
  reapplying a 3s burn over an 8s burn takes the max — the longest source wins permanently and
  every other ability's duration silently stops mattering.
- Fifteen numbers to balance instead of one.
- Nothing is learnable. "Burn is five seconds" is a fact a player can hold.

The discriminator is **payload versus rider**:

| | Duration | Examples |
|---|---|---|
| **Rider** — the ability does something else and the status tags along | Fixed on the status | burn, shock, corrosion, poison, slow, stagger, radiate |
| **Payload** — the status *is* the ability | `from_source` | ethereal, evasion, rime, gravity, ward, invisible, damage_immune, untargetable |

"Ethereal for 2s" and "ethereal for 5s" are two abilities at two prices and the duration is the
whole difference. "Deals 40 plasma and applies burn" is a rider, and the burn should be the same
burn every time or the phrase means nothing.

Abilities wanting a longer rider declare `duration_scale: 1.0 | 1.5 | 2.0` — banded so the
Calibrator can compare, and constrained to values that stay on the `TICK_FAST` grid. The `+`
form remains the preferred home for longer durations.

14 fixed, 19 from source, 2 endless.

### D-094 [N] The Balance Lab models stack ramp against target lifetime
Full stacks is a fine approximation against durable targets and badly wrong against chaff, and
the exact formula is two lines:

```
ramp = C / R
if ramp >= T:   average = R × T / 2
else:           average = C − C² / (2 R T)
```

**`T` is target lifetime, not the 120s window.** At 5 stacks and 3 applications/sec: 4.99
against a 60s boss, 4.90 against an 8s elite, 3.75 against a 2s minion, and **0.50** against
that minion from a 0.5/sec source — a tenfold overstatement if full stacks were assumed.

**This exposes a conflict between stacking and D-016** (O-33). Stacking needs long-lived
targets; the ladder locks base forms to minions and debris, which die before stacks accumulate.
So base `corrosion` and base `poison` are near-worthless stubs while all value sits in their `+`
forms. Recommended fix is exempting stacking debuffs from the base restriction so they reach
**elites**, with `+` forms still gating bosses and minibosses — D-016 exists to keep boss fights
authorable, and corrosion stacking on an elite does not threaten that.

### D-095 [R] `stasis` deleted; `paralyze` gains a `+` form; `gravity_plus` stops denying actions
`paralyze`, `stasis`, and `gravity_plus` were three statuses competing to be one mechanic,
escalating by accretion: `gravity_plus` was `paralyze` with a pull bolted on, `stasis` was
`paralyze` with two extra clauses.

```
paralyze       cannot move or act. Minions.
paralyze_plus  also cannot rotate; passives and status durations pause. Elites.
```

`stasis` is deleted, its two distinctive clauses becoming what makes `paralyze_plus` a `+` form.
Resolves the polarity error (D-088), O-29 (paralyze had a `+` form's reach with no base), and
O-32 (nothing applied stasis).

**`gravity_plus` escalates by strength of pull, not by borrowed lockout.** That leaves gravity as
trajectory control rather than paralysis — being dragged somewhere while still able to shoot is a
more interesting problem than being switched off.

### D-096 [N] VOID gravity fields bend projectiles
Nothing else in the game manipulates projectiles in flight, and it gives VOID an identity no
other element can copy: bend incoming fire away defensively, curve your own shots into a cluster
offensively. **It interacts with `homing` without duplicating it** — homing corrects toward a
target, gravity displaces regardless of intent, and the two forces sum cleanly.

**It belongs to the field object, not the status.** The `gravity` status is "this entity is being
pulled"; projectile bending is a property of the field doing the pulling.

Cheap and deterministic — projectiles already carry position and velocity. The real limit is how
many gravity fields exist at once, which is wave authoring rather than engine work. Logged as
O-34 since this is new scope rather than a correction.

### D-097 [R] The defensive statuses are four flags, not five statuses
`ethereal`, `damage_immune`, `untargetable`, and `invisible` overlapped because each was defined
whole rather than composed. They separate on four axes — `targetable`, `collidable`, `damageable`,
`visible` — and each occupies a genuinely distinct cell, so none needs deleting.

**The distinction that saves `damage_immune`: ethereal *misses*, damage immunity *lands for
zero*.** A miss applies no riders; a zero-damage hit still applies everything else. That is the
boss phase worth having — *you cannot hurt it yet, but you can stack corrosion for when the
shield drops.*

So **`damage_immune` stops blocking debuffs.** Dropping that clause is what makes it distinct
from `ethereal_plus` rather than a duplicate, and it simplifies D-091 to `damageable: false` with
no ImmunitySet manipulation.

**`untargetable` is the anti-homing status.** Now that `homing` is a rank line every player buys,
an untargetable baddie is real counterplay and a natural GAMMA signature. **`invisible` grants
`untargetable`** rather than restating the targeting rules — so invisibility breaks homing, and
breaks *yours* without any baddie needing homing of its own.

### D-098 [R] `ward` charges get a 1.0s internal cooldown
A charge blocks a hit regardless of size, so its value is set by the largest thing it can be
spent on — 9 damage against chaff, 600 against a boss special. **And piercing makes it worse**:
D-081 turns one gun-shot into five hits, so ward evaporates in a swarm. Simultaneously useless
where you are chipped to death and dominant where you are hit once every five seconds.

```
ward: N charges. Consuming one puts ward on a 1.0s internal cooldown.
```

One number fixes both ends. In a swarm it blocks one hit per second instead of vanishing;
against a boss it still absorbs the big telegraphed hits, which is the job no other defensive
status can do — evasion cannot touch specials (D-017) and ethereal+ only helps if timed.

**Resolves O-31**: a five-hit pierced gun-shot consumes one charge, because the cooldown starts
at the first hit.

An HP-pool ward was rejected as duplicating the `shield` rank line, and basic-attack scoping as
a third answer to a question `evasion` and `ethereal` already answer twice. Ward-stripping stays
available as an elite mechanic but is deferred — additional scope, not a fix.

### D-071 [N] The Expanse is 6000 × 6000
Square, wrapping both axes. Six screen-widths across, ~2.7 screen-heights. Six seconds to
cross at base speed.

---

## Open — not yet decided

| # | Item | Blocks |
|---|---|---|
| O-02 | ~~Status tag taxonomy~~ — **resolved by D-084.** Six tags, not five | ✅ |
| O-03 | Chrono+Cryo passive replacement (slot vacated by D-019) | Content, not blocking |
| O-04 | Element point economy — 10 points, max level 3, 5 base elements needs 15 to max. Sparse coverage means some spreads unlock very little; the Hangar must show that before you spend | Phase 7 |
| O-05 | Cross-axis parity — how a pure-defensive ability compares to a pure-offensive one. Starting heuristic: abilities declare a role, parity checked within role, cross-role weights set once | Phase 4, deferred until real data |
| O-06 | Threat profile calibration — composition and hit size are guesses until M0 | M0 |
| O-11 | **Screen names.** Proposed: Star Chart (sector select, which also frees `map`), Drydock (upgrades and loadout), Requisitions (purchases), Briefing, Settlement (payout), Expanse (open arena type). Also needed: Settings, Bestiary, Pause, Attributions | Phase 7 |
| O-12 | **Dogfighting boss.** Achievable — utility-scored behavior tree, ~8 candidate actions, 1–2 weeks mostly tuning. Must use the gameplay RNG and fixed timestep or determinism breaks. Conflicts with `03` §3.7 (every boss attack telegraphed ≥0.6s). Proposed resolution: **positioning** is reactive and untelegraphed, **attacks** stay telegraphed — it out-flies you rather than out-drawing you. Scope to exactly one boss. Reading `defense_affinity` as a counter to the player's attunement is cheap | Post-M0, content |
| O-13 | ~~Which element owns `invisible`~~ — **resolved by D-085.** GAMMA | ✅ |
| O-15 | **Does debris need a `NEUTRAL` faction?** Only if it should damage both sides. Otherwise it stays `UNFORMED` and the fiction oddity costs nothing | Phase 2 |
| O-17 | **Expanse at 6000².** Six seconds to cross at base speed. A torus too large stops reading as a torus and becomes an empty box | M0 |
| O-18 | **`HOMING_BASE` — 0° or 2°?** The 2° baseline mirrors the free crit: rate-based rotation plus screen-relative movement means a new player fights two axes at once. Thirty seconds on a phone answers it | M0 |
| O-20 | **`VALRUNE_SPEED_BASE = 1000`** is 2.3× v0's. Fast for a thumb-driven ship, and it sets both the Expanse size and the projectile ratio. The most load-bearing feel number in the game | M0 |
| O-21 | ~~`CLUSTERING = 2.0`~~ — **resolved by D-078** against a measured screenshot | ✅ |
| O-22 | ~~Is the 18° homing ceiling too steep?~~ — **resolved by D-080** at 3.5°, capped on lateral reach | ✅ |
| O-26 | **Pricing `piercing`.** At 4 ranks it is a 5× damage multiplier in crowds. Too cheap and single-target guns answer every encounter; too dear and the AoE gap stays open | Phase 4 |
| O-27 | **Abilities that grant piercing need reworking** against the new rank line rather than stacking with it | Phase 2 |
| O-28 | **`radiate`'s band shift is non-uniform** on the D-073 ladders — the same buff is worth +50 units on one ability and +800 on another. Cap, percentage, or flat bonus? Leaning flat | Phase 2 |
| O-29 | ~~`paralyze` has a `+` form's reach with no base form~~ — **resolved by D-095.** It has both now | ✅ |
| O-30 | **Is `rime`'s recoil a basic attack?** If so it is evadable, and two rimed entities shooting each other need a recursion guard | Phase 2 |
| O-31 | ~~Does `ward` block a gun-shot or a hit?~~ — **resolved by D-098.** The internal cooldown makes it one charge per gun-shot | ✅ |
| O-32 | ~~`stasis` — who applies it, and to whom?~~ — **dissolved by D-095.** Deleted | ✅ |
| O-33 | **Stacking versus the threat ladder.** Base `corrosion` and `poison` can only land on targets that die before stacks accumulate, making them stubs. Let base stacking debuffs reach elites? | Phase 2 |
| O-34 | **Gravity bending projectiles** (D-096) is new scope rather than a correction — worth confirming before it enters the schema | Phase 2 |
| O-23 | ~~Homing plus velocity inheritance~~ — **dissolved by D-076.** Distance-denominated correction makes the path speed-invariant, so the interaction cannot occur | ✅ |
| O-24 | **Every AoE ability is now worth 6–14× what the last revision assumed** (D-079). No authored damage number has been checked against the new counts | Phase 4 |
| O-25 | **200 baddies at 75 units** is a rendering and collision claim, not a design one. Prove it on a real mid-range phone | M0 |

**Resolved since last revision:** O-01 (D-058), O-07 (D-057), O-08 (D-058), O-09 (D-059),
O-10 (D-061), O-14 (dissolved by D-063 — Assist Aim cut), O-16 (D-068), O-19 (D-065).
