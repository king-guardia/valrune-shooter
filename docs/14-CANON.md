# 14 — Canon

**Status: draft, revision 2 — awaiting Bryan's review.** This is the vocabulary contract.
Once you sign off, these terms are used exactly as written, in documents, in data, and in
code.

Three rules for reading this:

1. **Terms are not interchangeable.** If two words appear here, they mean different
   things. A banned word is banned because it was doing two jobs.
2. **The "Code" column is the literal identifier.** `snake_case` for members and data ids,
   `PascalCase` for classes, `SCREAMING_CASE` for enum values only.
3. **[OPEN]** marks something still needing your call. They are collected at the
   [bottom](#open-questions).

**Spelling is American English** throughout: defense, offense, behavior, color, armor,
center. The v0 documents are full of British spellings and get corrected in Phase 7.

---

## 1. People and ships

| Term | Means | Code |
|---|---|---|
| **Player** | **The human being.** Always. Menus, flight, analytics — same person. | `Player` |
| **Valrune** | **The player's ship.** The entity with hull, position, and heading. | `Valrune` |
| **Pilot** | Fiction only. Never an identifier. | — |

**Banned: `user`.** Confirmed.

The player decides to dodge; the Valrune moves.

---

## 2. Baddies

| Term | Means | Code |
|---|---|---|
| **Baddie** | **Anything the player can damage.** The universal term. | `Baddie` |
| **Threat class** | Rung on the immunity and scaling ladder. | `threat_class` |
| **Debris** | **Class 0.** Asteroids, wreckage, derelict guns. | `DEBRIS` |
| **Minion** | Class 1. Ordinary enemies. | `MINION` |
| **Elite** | Class 2. Upgraded variant. | `ELITE` |
| **Miniboss** | Class 3. Contract 3 of a sector. | `MINIBOSS` |
| **Boss** | Class 4. Contract 6 of a sector. | `BOSS` |

**Debris is a full baddie, not an inert prop.** You were right to push on this. Debris
drifts on a heading, a meteor crosses fast, a derelict gun spins and fires as it tumbles.
It takes archetypes and role tags like anything else — it is simply class 0 for immunity
and scaling. Nothing about it is special-cased.

**Banned: `enemy`, `creep`, `NPC`, `hazard`.** Use **baddie**. (`mob` survives only as a
role-tag name, since it is already in your data.)

### Archetype

An **archetype** is the behavior template — what a baddie *does*, independent of what it
looks like or what it is called. Eight of them carry the whole roster, because
`archetype x role tags x affinity` generates the variety.

Your objection to the v0 names is correct, and the fix is structural: **the archetype id
describes the behavior, and the fiction name is display text.** They were fused before,
which is why renaming the faction meant renaming the code.

| Archetype id | Behavior | v0 fiction name |
|---|---|---|
| `drifter` | Straight-line drift. The chaff. | Mote |
| `weaver` | Sine-wave approach, faster, harder to lead. | Lash |
| `charger` | Locks on, telegraphs, then rams at 3x speed. | Spike |
| `splitter` | Tanky; breaks into two smaller baddies on death. | Husk |
| `turret` | Stationary shooter. | Anchor |
| `shielder` | Slow; grants nearby baddies immunity. | Warden |
| `miner` | Drops field objects on a pattern. | Seeder |
| `spawner` | Carrier; periodically emits minions. | Choir |

The right column is throwaway — v0 fiction, replaced when the faction is named. The left
column never changes. Data references `drifter`; the player sees whatever `locale/` says.

### Affinity

**Two fields, not one.** You are right that these are different questions.

| Term | Means | Code |
|---|---|---|
| **Attack affinity** | The damage types a baddie **deals**. | `attack_affinity` |
| **Defense affinity** | The types it is **typed as** for incoming matchups. | `defense_affinity` |

A baddie can hit you with plasma while being armored against cryo. v0 conflated them,
which quietly made every enemy symmetrical.

---

## 3. Weapons

Your `misc_ideas` lines 19 and 37 asked for this section.

| Term | Means | Code |
|---|---|---|
| **Gun** | **A mount.** You own 1, 2, or 3. Each is one attunement slot. | `gun_count`, `GunMount` |
| **Gun-shot** | **One trigger event across every gun at once.** Three guns firing together is *one* gun-shot. | `GunShot` |
| **Projectile** | **One bullet from one gun.** A 3-gun ship makes 3 per gun-shot. | `Projectile` |
| **Beam** | **A continuous weapon effect.** Deals damage on ticks rather than on impact. | `Beam` |
| **Fire rate** | Gun-shots per second. **Never** projectiles per second. | `fire_rate` |

The gun-shot is the atomic unit: it is what fire rate counts and what crit rolls once per.
Conflating it with projectile inflates DPS 3x on a three-gun ship.

**Banned: `volley`.** Confirmed. **Banned: `lance`** — beam is the word.

**A beam impacts, then ticks.** No hitscan-instant damage path is needed anywhere in the
game, which removes a whole architecture branch. Beams use the same `TICK` cadence as
everything else.

**`fire` is a verb only.** The gun fires. The element is PLASMA, the status is burn, the
field object is a flame trail.

### Basic attack vs ability

The rules-facing distinction. Evasion, ethereal, and poison all scope by it.

| Term | Means | Code |
|---|---|---|
| **Basic attack** | An entity's **default repeating weapon fire**: governed by a *rate*, costs nothing, needs no unlock. | `is_basic_attack` |
| **Ability** | Anything governed by a **cooldown or duration**, or granted by a node or the mage tag. | `Ability` |
| **Contact damage** | Damage from **collision**. **A third category** — neither basic attack nor ability. | `contact_damage` |

The test is **rate vs cooldown**. A basic attack fires on a rate forever; an ability has a
cooldown.

**The Valrune and baddies share this vocabulary completely.** You asked whether they need
separate names — they do not, and unifying them is the better call. Both have a basic
attack. The Valrune's is implemented as a gun-shot; a baddie's may be a gun-shot too, or
a spit, or a beam. The rules reference `is_basic_attack` and never ask who fired.

The only thing that stays player-specific is `fire_rate` living on the rank tree.

**Evadable = basic AND single-target.** Confirmed. So the CRYO wave blast — a basic attack
that became AoE — is not evadable. Contact damage is never evadable.

---

## 3a. Spawned objects

| Term | Means | Code |
|---|---|---|
| **Drone** | An **undamageable, untargetable** companion that flies to `r1` of the nearest baddie and shoots, staying within `r2` of the Valrune (D-020). No HP. | `Drone` |
| **Drone type** | `hologram`, `undead`. Extensible. | `drone_type` |
| **Field object** | A persistent thing occupying space: mine, flame trail, fissure, zone, portal, wire net. | `FieldObject` |
| **Trail** | A field object emitted continuously from the Valrune's stern as it moves. | `TrailEmitter` |
| **Lifetime** | How long a spawned object exists. | `lifetime` |
| **Faction** | `PLAYER` or `BADDIE`. Determines what can hurt what. | `faction` |

**On your lifetime/cooldown question — yes, exactly that.** The parameter always exists;
its value may be unbounded. `lifetime: null` means "until the contract ends",
`cooldown: null` means "never recharges, not applicable". A passive has
`cooldown: null` unless it is auto-triggered with downtime. Never invent a sentinel like
`-1` or `999999`; `null` is the unbounded value and the validator understands it.

**No friendly fire, in either direction** (`misc_ideas` line 26). Player field objects,
abilities, and projectiles never hurt the Valrune or its drones. Baddie effects never hurt
baddies. Projectiles pass through same-faction entities as if they were not there.
**Drones are non-collidable with everything** — they are not cover for you and not an
obstacle for baddies.

A drone is unrelated to the `decoy` role tag, which produces damageable 1-HP baddie
illusions.

---

## 4. Elements and attunement

| Term | Means | Code |
|---|---|---|
| **Element** | One of nine. Base: PLASMA, CRYO, FORGE, VOLT, CAUSTIC. DLC: CHRONO, GAMMA, VOID, ETHER. | `Element` |
| **Attunement** | **An element assigned to a gun** — the assignment itself. | `attunement` |
| **Attunement slot** | One gun's element position. Never bare "slot". | `attunement_slots` |
| **Attunement effect** | The on-hit effect. Scales 1.0 / 1.5 / 2.0 with element rank. | `attunement_effect` |
| **Attunement crit** | The on-crit effect. Belongs to the attunement, not the node (D-010). | `attunement_crit` |
| **Type chart** | The strong/weak/cross-circle table. | `TypeChart` |

**On attunement vs attuned — you are slightly overcomplicating it.** "Attuned" is just the
adjective form of the same word, the way "equipped" is to "equipment." Write whatever
reads naturally in prose ("when cryo and gamma are attuned…"), but keep **one identifier
family**, `attunement_*`. Two identifiers for one concept is the exact failure this
document exists to prevent. So: `attunement_crit`, not `attuned_crit`.

It is **ETHER**, never AETHER (D-005).

Elements are SCREAMING_CASE in prose and as enum values; `snake_case` as data ids.

---

## 5. Progression

Everything purchasable is a **rank line**. Your consolidation is right, and it collapses
two parallel systems into one.

| Term | Means | Code |
|---|---|---|
| **Rank tree** | All rank lines together. | `RankTree` |
| **Rank line** | One purchasable track. | `RankLine` |
| **Rank** | One purchased step within a line. | `rank` |
| **Currency** | What a line costs: `credits` or `element_points`. | `currency` |
| **Credit** | Earned from bounties. | `credits` |
| **Element point** | From minibosses and bosses. **10 total.** | `element_points` |
| **Element rank** | An element's investment, 0–3. | `element_rank` |

**An element is a rank line with 3 ranks priced in element points.** Nothing breaks. Both
currencies buy ranks, the Hangar screen shows one uniform interaction, and respec is one
code path. The only asymmetry is downstream: element ranks feed node tiers, credit ranks
do not. That is a consumer of the data, not a difference in the data.

**Rank is a purchased step. Tier is a node's I/II/III.** Never the same word.

### The thirteen credit rank lines

| Group | Lines |
|---|---|
| **Offense** | `damage`, `attack_speed`, `guns`, `crit_chance`, `crit_damage`, `velocity`, `spread` |
| **Defense** | `hull`, `shield`, `shield_recharge`, `bulwark_flat` |
| **Mobility** | `thrusters`, `gyros` |

Ids are `snake_case`; display names live in `locale/`. Not SCREAMING_CASE — that is
reserved for enum values, and these are data rows.

**Thirteen, not v0's fourteen.** `bulwark_percent` is deleted (D-058): percentage
mitigation multiplies with evasion, rime, and shields, which is the increasing-returns
problem D-029 warns about. **Mitigation is flat only.** The percentage allowlist is now
genuinely closed at crit chance, crit damage, and Overclock — all three legitimately
probabilistic or multiplicative by nature.

### Nodes and abilities

| Term | Means | Code |
|---|---|---|
| **Combination** | A set of 1–3 elements. 9 + 36 + 84 = **129**. | `combination` |
| **Node** | One **(combination x passive-or-active)** position. May be empty. **249 possible.** | `Node` |
| **Ability** | What a filled node grants. Has three tiers. | `Ability` |
| **Tier** | The I / II / III version. Driven by element ranks. | `tier` |
| **Passive** | Always on. | `PASSIVE` |
| **Active** | Player-triggered, on a cooldown. | `ACTIVE` |
| **Loadout** | The five equipped nodes. | `Loadout` |
| **Loadout slot** | 3 passive, 1 active, 1 auto-active (D-022). Never bare "slot". | `loadout_slots` |
| **Auto-active** | An active fired the moment its cooldown ends. | `is_auto_active` |

**Empty nodes are never shown to the player.** Worth stating, because it is not obvious
and it changes the UI: the Hangar shows only nodes with authored abilities. The player
never sees a locked grid position, an empty cell, or a "coming soon" — as far as they
know, the combinations that exist are all the combinations there are. This is what makes
sparse coverage (D-003) invisible rather than looking unfinished.

**Banned: bare `slot`.** Always `attunement slot`, `loadout slot`, or `node`.

---

## 6. Ability anatomy

| Term | Means | Code |
|---|---|---|
| **Trigger** | What fires it: activation, on-hit, on-crit, on-kill, proximity, always. | `trigger` |
| **Delivery** | **How it reaches its target.** See below. | `delivery` |
| **Target spec** | Where it **reaches** — the region it selects from. | `target_spec` |
| **Arrival spec** | Where the effect **lands**, which may differ. | `arrival_spec` |
| **Travel end** | What stops it in flight. See below. | `travel_end` |
| **Shape** | `none, circle, oval, line, 3-point … 7-point`. | `shape` |
| **Duration** | How long the effect persists. | `duration` |
| **Cooldown** | Recharge time. **Begins when duration ends.** | `cooldown` |
| **Charge** | A stored use. Cooldown starts when the *first* is spent. | `charges` |
| **Tick** | One periodic application. | `tick` |
| **Uptime** | `duration / (duration + cooldown)`. | `uptime` |
| **Proc** | One firing of a triggered effect. | `proc` |

### Delivery

You are right that this was missing, and it is not a small omission — it changes what the
projectile code has to do.

| Value | Means |
|---|---|
| `homing` | Tracks a target. **Retargets if the target dies mid-flight.** |
| `skillshot` | Fires along the heading. Hits whatever it meets. |
| `fixed_point` | Travels to a playfield coordinate (screen center, the ready line). |
| `self` | Originates and lands on the caster. |
| `attached` | Rides the caster or a target. Trails and auras. |

### Travel end

Also yours — "goes until it hits a baddie OR reaches N distance, then does arrival_spec."
That is two termination conditions on one projectile, so it is a list, not a value:

`on_hit` · `on_distance` · `on_target_reached` · `on_lifetime`

Whichever fires first triggers the arrival spec.

### Variable naming — full words in code

**You are not being dumb, but the reason for short names does not survive contact with
code.** `dur` and `c` are excellent in a spreadsheet, where you type every character and
the column header supplies the context. In code there is no column header, autocomplete
makes length free, and `c` next to `charges` and `cooldown` and `combination` is a genuine
hazard.

So: **full words for variables** — `duration`, `cooldown`, `radius`, `stacks`, `damage`.

**Short forms survive only as band ids** — `r1`, `dis2`, `w1`. Those are not variables,
they are *token values*, closer to `PLASMA` than to `duration`. `radius: "r2"` reads
correctly.

Your line-12 convention stays exactly as written for the spreadsheets. It just does not
cross into the schema.

### The tick is universal

**Adopting your proposal, and it is a better idea than it may look.** One global cadence:

```
TICK = 0.2s
```

Every damage-over-time, every zone, every beam, every periodic anything lands on the same
0.2s boundary. Three real wins beyond tidiness:

- **Determinism.** All periodic effects resolve in one batch on one frame, in a stable
  order. Independent timers drift and make replays diverge.
- **Legibility.** Damage numbers arrive in rhythm instead of a scatter.
- **Performance.** One timer walking a list, rather than N timers.

If something needs to be faster it must be a **strict subdivision** — `TICK_FAST = 0.1s`,
which still lands on every other global tick. Never 0.15s, which aligns with nothing and
reintroduces exactly the jank you are avoiding.

---

## 7. Damage

| Term | Means | Code |
|---|---|---|
| **Damage packet** | **The unit of resolution.** Carries a type set, resolves the type chart exactly once (D-007). | `DamagePacket` |
| **Damage type** | Named for the element. A packet may carry several. | `damage_types` |
| **Hit** | One packet landing on one baddie. What "on hit" counts. | `on_hit` |
| **Source** | The entity or object credited with an effect. Push is away from source, pull toward it. | `source` |
| **Crit** | A gun-shot rolling critical. Multiplies **only the base gun packet** (D-008). Rolls **once per gun-shot**. | `is_crit` |
| **Bounty** | Credits awarded for a kill. | `bounty` |

### Crits and sources

Your correction is right and my earlier phrasing was too broad. Precisely:

> **A crit is not itself a source. Anything a crit spawns is.**

So "when a source causes X" does not fire merely because a hit critted — but VOID's
on-crit gravity field *is* an object, and that object is a source like any other. The
distinction is between the *event* and the *thing the event creates*.

### Probability is always PRD

**Adopting your rule as universal, with one caveat worth knowing.**

> Any percentage chance in the game uses pseudo-random distribution. There is no bare
> `randf() < chance` anywhere.

Chance climbs with each failed roll and resets on success, so streaks are impossible in
both directions and the long-run rate matches nominal exactly. Crit already worked this
way; evasion, procs, and any future chance join it.

The caveat: **PRD state is per (entity, effect), never shared.** Two baddies each track
their own evasion counter, and a baddie's evasion counter is separate from its crit
counter. Sharing state would let one entity's luck drain another's.

For balance this **changes nothing about mean EHP** in D-029 — the long-run rate is
identical. It tightens variance, which strengthens D-017's case for allowing evasion at
all: the reason scoping was needed was slow convergence, and PRD converges much faster.

**No self-damage, no friendly fire** (line 26), covered in §3a.

---

## 8. Statuses

| Term | Means | Code |
|---|---|---|
| **Status** | Any timed effect on an entity. | `Status` |
| **Buff / Debuff** | Beneficial / detrimental. Buffs carry no immunity data (D-015). | `polarity` |
| **Family** | **The group a status and its `+` forms share.** See below. | `family` |
| **Tag** | Behavior class driving immunity: `control`, `dot`, `vulnerability`, `avoidance`, `movement`. | `tags` |
| **Stack** | One application. A new stack resets the timer for **all** stacks (line 39). | `stacks` |
| **The `+` form** | A stronger variant that pierces further up the threat ladder (D-016). A **separate status**, not a stack level. | `burn_plus` |
| **Owner element** | The element a status belongs to. May be `null`. | `owner_element` |
| **ImmunitySet** | What an entity cannot receive. Derived from tags. | `ImmunitySet` |
| **OverrideSet** | What a source pierces. | `OverrideSet` |

### Families — the `+` forms must never get lost

**This is the most important catch in your review.** You are right that it would have
caused real bugs, and quiet ones.

When an ability says *"deals bonus damage to baddies with burn"*, it means burn **and**
burn+. Every such reference is to the **family**, never to a specific status id.

```
burn, burn_plus            -> family: burn
corrosion, corrosion_plus  -> family: corrosion
evasion, evasion_plus,
evasion_plus_plus          -> family: evasion
```

Ability data references `family: burn`. Referencing a bare status id in a condition is a
schema error the validator rejects, because it is almost always this mistake.

### Element anchoring is functional, not just a design note

You framed this as mostly a design aid, but the lava-boss example you gave is a real
mechanism and worth making explicit:

> **Immunity to an element implies immunity to that element's statuses.**

A boss immune to plasma takes no burn and no burn+, automatically, with no per-status
authoring. That falls out of `owner_element` for free and is why the field earns its
place.

`owner_element: null` is legitimate for genuinely universal statuses — shield, stasis,
paralyze, MaxHP_Loss, Override. There is **no `Misc` element**; that column in
`buffs_debuffs.csv` is a spreadsheet convenience. Invisible is unassigned and probably
wants an owner [OPEN] — you leaned GAMMA, since ETHER already carries several.

Beyond the threat-class ladder, **any entity may carry bespoke immunities or overrides**
per placement in wave data. The lava boss is authored, not a new system.

**Statuses apply after damage resolves** (D-009). **Reapplication never shortens**
duration (line 32).

---

## 9. Geometry

| Term | Means | Code |
|---|---|---|
| **Playfield** | The battle area. **1000 units wide** (D-024). | `PLAYFIELD_WIDTH` |
| **Bow / Stern** | Front / back of a ship. | `bow`, `stern` |
| **Port / Starboard** | Left / right, ship-relative. | `port`, `starboard` |
| **Heading** | The direction a ship faces. Distinct from travel — movement is screen-relative. | `heading` |
| **Ready line** | **A full horizontal line**, not a point. | `READY_LINE_Y` |
| **Radius band** | `r_short`, `r1`, `r2`, `r3`. A region **around** a point. | `r1` |
| **Distance band** | `dis_short`, `dis1`–`dis4`. Reach **away from** a point. | `dis1` |
| **Width band** | `w0`–`w3`. | `w1` |

**Banned: `ground`.** Confirmed — it is space.

**Recall moves on the Y axis only.** Your correction changes the mechanic: the Valrune
drops straight down to the ready line, holding its X, rotating smoothly toward north on
the way and arriving facing forward. If turn rate is too slow to finish in transit, the
rotation completes on arrival. v0 said "bottom-center," which would have yanked you
sideways — worse, and it would have made recall a repositioning tool rather than a reset.

### Band values

**Radius and distance couple only at the short end**, then diverge:

| Band | Value |
|---|---|
| `dis_short` | = `r_short` |
| `dis1` | = `r1` |
| `dis2` | Independent |
| `dis3` | **Ready line to the top playfield border.** The natural full-screen reach. |
| `dis4` | Beyond `dis3` — matters for angled shots that wrap, and for the open arena. |

Both ladders extend if content needs it. `dis3` being screen-derived is useful: it is the
one band with a physical meaning rather than a tuned number.

**Widths are Valrune-relative**, which makes them self-documenting:

| Band | Value |
|---|---|
| `w0` | A projectile's own width. No spread. |
| `w1` | Half the Valrune's width. |
| `w2` | The Valrune's full width. |
| `w3` | Two Valrunes wide, plus half a Valrune beyond each side. |

CRYO likely needs wider, given how short its reach is. Extend the ladder rather than
special-casing.

### Push, pull, and gravity

**Your model is cleaner than mine and I am adopting it.** I had gravity "overriding"
movement, which implies an arbitration rule. Yours has no arbitration at all:

> **Gravity disables self-movement. Pull then becomes the entity's only movement.**

Nothing competes, so nothing needs resolving.

| Term | Means | Code |
|---|---|---|
| **Push** | Displacement **away from** a source. Does not stop self-movement. | `push` |
| **Pull** | Sustained movement **toward** a source. Does not stop self-movement. | `pull` |
| **Gravity** | A **status** that disables self-movement. Pairs with pull. | `gravity` |
| **Push/pull band** | A `(distance, time)` pair, banded like radii: `push_1`, `pull_2`. | `push_1` |
| **Stop radius** | Where a pull stops pulling. | `pull_stop_radius` |

Banding the `(distance, time)` pair keeps the "short speedy Forge knock" and the "slow
heavy Void drag" as two named constants rather than four loose numbers per ability.

**Pull needs a stop condition** — your case of "pull until within a radius, then hold."
`pull_stop_radius: null` pulls all the way to the point; a value stops it at that ring
while gravity keeps holding the target still.

**Banned: `mr` / move rate.** Confirmed.

---

## 10. Structure

| Term | Means | Code |
|---|---|---|
| **Contract** | **One job.** 30 in the campaign. Has an index, a payout, clauses, and waves. The unit of everything. | `Contract` |
| **Wave** | One authored group of baddies inside a contract. | `Wave` |
| **Sector** | Six contracts. Five sectors. | `Sector` |
| **Wormhole** | Vertical contract type. Horizontal wrap, one screen. | `WORMHOLE` |
| **Expanse** | Open arena contract type. **Wraps on both axes.** | `EXPANSE` |
| **Clause** | A voluntary difficulty modifier for higher pay. 4 slots. | `Clause` |
| **Bounty** | One baddie's share of the contract's credit pool. | `bounty` |
| **Bounty pool** | The contract's total credits, set by its index. | `bounty_pool` |

**Banned: `stage`.** Your confusion was correct — the word was doing nothing. A contract
*is* the playable thing; playing it is just playing it. If we ever need to name a single
attempt, that is a **run** (`ContractRun`), and only for save data and analytics.

**Banned: `throat`.** It is a **wormhole**. Claude's word, and not a good one.

**Banned: `level`, `mission`.** `level` especially — it already means element rank.

### Bounty

Credits are awarded per kill, but the **total is set by the contract, not by the kills**.

```
bounty_pool   = f(contract_index)
baddie_bounty = bounty_pool / Σ weights
                × weight(threat_class)
                × random(0.85, 1.15)
```

Paid only on **reaching a checkpoint or completing the contract.** Clause bonuses apply to
the total at the end.

This keeps both properties that looked like a tradeoff. Payout stays index-anchored, so
farming an easy contract is pointless and clearing fast costs nothing — and the ceiling is
structural rather than a rule, because you cannot kill more baddies than the contract
contains. Meanwhile every kill pays visibly, and **an escapee genuinely costs its share**,
which gives the wormhole's bottom edge a consequence without a second fail state and turns
`spawner` into a soft DPS check.

The `random(0.85, 1.15)` spread is a **range roll, not a probability.** PRD (D-049)
governs binary chances; damage and bounty roll ranges.

### The Expanse wraps

The open arena is a **rectangle wrapping on both axes**, not v0's disc with elastic
pushback (D-059). Wrapping is the universal rule; the wormhole is just the case that clamps
Y for design reasons, not a second model to learn.

It ships with a small **minimap** carrying baddie dots, and the minimap must be
**torus-aware** — a baddie near the right edge is also near the left edge, and a minimap
that hides that is worse than no minimap.

---

## 11. Baddie behavior

| Term | Means | Code |
|---|---|---|
| **Action** | One behavior with a duration. **Actions overlap** — a baddie moves and shoots at once. | `BaddieAction` |
| **Idle** | An action. Doing nothing, for a duration. | `IDLE` |
| **Telegraph** | Visual warning. **White** = movement, **red** = attack. | `Telegraph` |
| **Path type** | The shape a move action follows. | `path_type` |
| **Ability pool** | What a `mage`-tagged baddie draws from. Bespoke abilities or existing player nodes. | `ability_pool` |

A baddie is a **set of concurrent actions**, not one state. Phase 3 diagrams it.

---

## Open questions

Two left. Everything else is settled.

### 1. Naming — the faction and the screens

Eldritch-plus-technology gives a clean structural axis that maps to your description, and
I would put it in the data: `chassis: organic | augmented`. Organic ones are
collision-focused and may die on impact or ignore it entirely; augmented ones shoot.
That is one field driving both fiction and behavior.

Names I would put forward, all replaceable:

| Screen | Proposal |
|---|---|
| Sector / contract select | **Star Chart** |
| Ship upgrades and loadout | **Drydock** |
| Purchases | **Requisitions** |
| Pre-contract | **Briefing** |
| Post-contract payout | **Settlement** |

"Star Chart" also frees up `map`, which I would otherwise have banned. **Expanse** is
already adopted for the open arena type.

Screens you did not list but will need: **Settings**, **Bestiary** (v0 references one, and
it is where DLC purchase intent forms), **Pause**, **Attributions**, and a **Store** if DLC
is not folded into Requisitions.

### 2. A dogfighting boss

Very achievable, and not "AI" in any modern sense — a utility-scored behavior tree with
maybe eight candidate actions, picking the best each half-second. Perhaps 1–2 weeks for
one good one, most of it tuning.

Two real constraints. It must use the gameplay RNG and the fixed timestep or it breaks
determinism. And it fights `03` §3.7's rule that every boss attack is telegraphed ≥0.6s —
a reactive opponent that telegraphs everything is not much of a dogfight, but one that
does not is unfair by your own standard.

The resolution I would suggest: the boss's **positioning** is reactive and untelegraphed,
its **attacks** stay telegraphed. It out-flies you rather than out-drawing you. Scope it
to exactly one boss, ideally the final one, and treat it as content rather than a system.

Reading your defense affinity as the counter to the player's current attunement is cheap
and a nice touch.

### Still unnamed

The faction. Not "the Hollow."
