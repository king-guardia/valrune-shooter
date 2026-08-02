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
through as if absent. `faction: MERCENARY | HORROR` (D-061). Types so far: `hologram`,
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

### D-061 [N] The factions are the Mercenaries and the Horror
`faction: MERCENARY | HORROR`, replacing the placeholder `PLAYER | BADDIE`. Grounding the
enum in fiction rather than in role means a third faction later costs nothing.

**"The Hollow" is deleted.** The Horror is eldritch-plus-technology: organic ones are
collision-focused and may die on impact or ignore it entirely; augmented ones shoot, spit,
or beam. The proposed `chassis: organic | augmented` field drives fiction and behavior
together.

Debris sits in `HORROR` [OPEN] — fictionally odd for an asteroid, but faction only decides
what can damage what, and a third `NEUTRAL` faction would earn its keep only if debris
needs to hurt both sides.

---

## Open — not yet decided

| # | Item | Blocks |
|---|---|---|
| O-01 | Percentage allowlist beyond crit chance, crit damage, Overclock | Phase 1b |
| O-02 | Status tag taxonomy — do the five tags carve the space correctly? | Phase 1d |
| O-03 | Chrono+Cryo passive replacement (slot vacated by D-019) | Content, not blocking |
| O-04 | Element point economy — 10 points, max level 3, 5 base elements needs 15 to max. Sparse coverage means some spreads unlock very little; the Hangar must show that before you spend | Phase 7 |
| O-05 | Cross-axis parity — how a pure-defensive ability compares to a pure-offensive one. Starting heuristic: abilities declare a role, parity checked within role, cross-role weights set once | Phase 4, deferred until real data |
| O-06 | Threat profile calibration — composition and hit size are guesses until M0 | M0 |
| O-11 | **Screen names.** Proposed: Star Chart (sector select, which also frees `map`), Drydock (upgrades and loadout), Requisitions (purchases), Briefing, Settlement (payout), Expanse (open arena type). Also needed: Settings, Bestiary, Pause, Attributions | Phase 7 |
| O-12 | **Dogfighting boss.** Achievable — utility-scored behavior tree, ~8 candidate actions, 1–2 weeks mostly tuning. Must use the gameplay RNG and fixed timestep or determinism breaks. Conflicts with `03` §3.7 (every boss attack telegraphed ≥0.6s). Proposed resolution: **positioning** is reactive and untelegraphed, **attacks** stay telegraphed — it out-flies you rather than out-drawing you. Scope to exactly one boss. Reading `defense_affinity` as a counter to the player's attunement is cheap | Post-M0, content |
| O-13 | Which element owns `invisible` — Bryan leans GAMMA, since ETHER already carries several | Phase 1d |
| O-14 | **`homing` vs the Assist Aim accessibility toggle.** Selling a purchasable version of an accessibility feature needs care. Cleanest split: `homing` bends the *projectile*, Assist Aim bends the *ship's heading* — different mechanisms, no awkwardness about paying for accessibility | Phase 1b |
| O-15 | **Does debris need a `NEUTRAL` faction?** Only if it should damage both sides. Otherwise it stays `HORROR` and the fiction oddity costs nothing | Phase 2 |
