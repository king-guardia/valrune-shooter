> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 04 — Progression, Calibration & the Element System

## 4.1 Two axes

| Axis | Currency | Source | Job | Reversible |
|---|---|---|---|---|
| **Breadth** | Credits | Every contract | Universal stat tree | Yes, 100 credits flat |
| **Identity** | Element Points | Minibosses + bosses. **10 total** | Element levels → attunements + nodes | Yes, 100 credits flat |

There is no pilot level and no XP. Pacing comes from cost, not from locks. Respec is deliberately cheap so experimenting is never a tax.

**No node, upgrade, or ability increases credit gain.** Greed stats compel players who feel obliged to max them, and they distort every balance decision downstream.

### Element point gating

| Point in campaign | Element points usable |
|---|---|
| Contracts 1–2 (tutorial) | 0 — no element system at all |
| After **Miniboss 1** (contract 3) | 1 |
| After **Boss 1** (contract 6) | All owned, DLC included |

Someone who buys three packs on install still flies Sector 1 as a clean shooter.

---

## 4.2 The power curve

| | Start | End | Multiplier |
|---|---|---|---|
| Player DPS (stat tree only) | 8.1 | 463 | **57×** |
| Player DPS with element effects | 8.1 | ~550+ | **~68×** |
| Player hull | 10 | 500 | 50× |
| Enemy damage | 3–5 | ~60 | 15× |
| **Effective survivability** | 2.5 hits | 8.3 hits | **3.3×** |

50× hull against 15× enemy damage nets 3.3× as the floor. FORGE and CRYO nodes carry a defensive build toward **~10×**; a glass-cannon VOLT build stays near 3× and trades it for damage.

```
start = 4.0 avg dmg × 1 gun × 2.0/sec × 1.0125 (5% crit @1.25×)   = 8.1 DPS
end   = 19.0 avg × 2.1 gun mult × 8.0/sec × 1.45 (30% crit @2.5×) = 463 DPS
```

### Enemy scaling

| | Sector 1 | Sector 5 | Multiplier |
|---|---|---|---|
| Basic enemy hull | 20 | ~1,400 | 70× |
| Enemy damage | 3–5 | ~60 | 15× |
| Enemies per contract | ~40 | ~140 | 3.5× |

Enemies scale slightly ahead of the player, so time-to-kill holds near 2.5–2.8s and **matchups are the difficulty lever** rather than raw numbers. Chrono Trigger's shape without its treadmill: numbers stay legible, and the largest figure in the game is a four-digit boss hull pool.

**Cost scaling is arithmetic:** `cost(n) = base + step × (n − 1)`. Cumulative cost is quadratic against linear benefit, which soft-caps naturally. Never geometric.

---

## 4.3 The stat tree

Fourteen lines. **Total sink: 30,477 credits.**

### Offence

| Line | Ranks | Effect | Cost | Total |
|---|---|---|---|---|
| **Damage** | 7 | Raw damage range (table below) | 10 + 100×(n−1) | 2,170 |
| **Attack Speed** | 24 | +0.25 shots/sec, 2.0 → 8.0 | 10 + 12×(n−1) | 3,552 |
| **Guns** | 2 | 1 → 2 → 3 guns (§4.4) | 800, then 2,000 | 2,800 |
| **Crit Chance** | 25 | +1%, 5% → 30% | 20 + 8×(n−1) | 2,900 |
| **Crit Damage** | 25 | +0.05×, 1.25× → 2.50× | 10 + 6×(n−1) | 2,050 |
| **Velocity** | 5 | +8% projectile speed | 10 + 20×(n−1) | 250 |
| **Spread** | 6 | +2° (§4.4.4) | 10 + 15×(n−1) | 285 |

| Rank | Range | Avg |
|---|---|---|
| 0 | 3–5 | 4.0 |
| 1 | 5–7 | 6.0 |
| 2 | 7–9 | 8.0 |
| 3 | 9–11 | 10.0 |
| 4 | 11–14 | 12.5 |
| 5 | 14–17 | 15.5 |
| 6 | 17–21 | 19.0 |

Damage is **raw, not percentage** — element effects derive from this number. Bounds are monotonic (each rank's floor equals the previous rank's ceiling), and the widening from rank 4 is deliberate: variance grows with power, so late hits feel less metronomic.

### Defence

| Line | Ranks | Effect | Cost | Total |
|---|---|---|---|---|
| **Hull** | 44 | Raw HP, 10 → 500 | tiered bands | 2,490 |
| **Shield** | 3 | Absorbs one hit entirely | 500 / 1,500 / 3,000 | 5,000 |
| **Shield Recharge** | 3 | 120s / 90s / 60s undamaged | 100 / 500 / 1,000 | 1,600 |
| **Bulwark (flat)** | 8 | −N raw damage per hit | 10 + 40×(n−1) | 1,200 |
| **Bulwark (%)** | 25 | −1% damage, cap 25% | 20 + 10×(n−1) | 3,500 |

Hull bands: 10 cr per 10HP to 100 · 30 to 200 · 50 to 300 · 80 to 500. **Hull damage persists through a contract; shields recharge.**

**Damage resolution order** (fix now, record as an ADR):
```
raw enemy damage → element multiplier (§4.7) → shield absorb (if charged, stop)
→ Bulwark flat → Bulwark % → floor at 25% of post-element damage
```

### Mobility

| Line | Ranks | Effect | Cost | Total |
|---|---|---|---|---|
| **Thrusters** | 20 | +0.15× base speed, to **4.0×** | 10 + 6×(n−1) | 1,340 |
| **Gyros** | 20 | +0.15× base rotation, to **4.0×** | 10 + 6×(n−1) | 1,340 |

Both scale to 4× base, and both have a **settings slider running 0.5× to whatever you have purchased.** Buying raises the ceiling; the slider sets where you fly inside it. A player who over-invests and finds the ship twitchy dials it down without a respec, and a player who wants a slow, deliberate ship can buy the ranks purely to widen the range.

### Income

```
base_credits(contract_index) = 140 + 30 × contract_index
```
Contract 1 pays 170, contract 30 pays 1,040. Campaign total ≈ **18,150 = 60% of the tree** on a single clear. Clauses and challenges supply the rest.

**Single-clear sufficiency** is the gate: one clear of every contract must fund a build that clears the next at 55th-percentile skill. Verified by the sim at every milestone.

**Respec:** 100 credits flat, no scaling, for stats or elements. Attunement and loadout changes are always free.

---

## 4.4 Guns & Attunements

Each gun is an **attunement slot**. Slotting is free and instant.

### 4.4.1 Configurations

| Guns | Damage split | Total | Slots |
|---|---|---|---|
| 1 | 1.0n | 1.0n | 1 |
| 2 | 0.8n + 0.8n | 1.6n | 2 |
| 3 | **1.1n** (centre) + 0.5n + 0.5n | 2.1n | 3 |

The three-gun split is deliberately asymmetric: the centre gun carries **52%** of your damage, each wing **24%**, making the centre slot worth **2.2× a wing slot**.

Any owned element goes in any slot, **duplicates allowed** — 3-0-0, 2-1, or 1-1-1. A slot may be **null**: damage with no element, no on-hit effect, neutral matchup. Guns cost credits while elements cost element points, so null slots are common early and are a visible reason to invest in elements.

### 4.4.2 On-hit effects

Each gun applies **its own** effect independently. Duplicates apply the effect multiple times per volley; spread applies several different effects. There is **no special stacking rule** — the behaviour emerges from the damage split, and adding one would break the elegance.

| Element | On hit | Primitive |
|---|---|---|
| **PLASMA** | **Burn** — damage over time, stacks to 3 | `damage_over_time` |
| **CRYO** | **Splash** — small area damage at impact | `splash` |
| **FORGE** | **Heavy** — flat bonus damage plus a brief stagger | `flat_bonus, knockback` |
| **VOLT** | **Arc** — zaps one nearby second target | `chain` |
| **CAUSTIC** | **Corrode** — stacking vulnerability to all damage | `vulnerability, armor_strip` |
| **CHRONO** | **Drag** — brief movement slow | `slow` |
| **GAMMA** | **Expose** — target takes more from all sources for 2s | `damage_amplify` |
| **VOID** | **Pull** — draws targets toward the impact point | `pull` |
| **AETHER** | **Phase** — a share of the hit bypasses shields and flat reduction | `pierce` |

**Potency scales with element level: 1.0 / 1.5 / 2.0.** Modest, because nodes carry the depth — but it gives element points a second, immediately visible payoff.

These nine are **composed from one framework, math-scaled**, not authored individually.

### 4.4.3 Matchups and hedging

Each gun's damage carries **its own** element for type-chart purposes, so hedging earns partial credit. Against an enemy weak to PLASMA and resistant to VOLT:

| Config | Effective |
|---|---|
| 3× PLASMA | **1.300×** |
| 2 PLASMA + 1 CRYO | 1.229× |
| PLASMA centre + 2 CRYO | 1.157× |
| CRYO centre + 2 PLASMA | 1.143× |
| 1-1-1 spread | 1.098× |
| All null | 1.000× |
| 3× VOLT | 0.750× |

Committing pays, spreading hedges, centre placement beats wing placement, and nobody is ever hard-countered because the config rebuilds for free between contracts.

### 4.4.4 Visuals and spread

- **1 gun:** single stream.
- **2 guns:** the streams **helix** around a shared axis. Distinctive, cheap (a sine offset perpendicular to travel), and it fixes a readability problem — two spread streams read as noise, a braid reads as one coherent thing.
- **3 guns:** centre fires straight, wings fan outward.

**Spread behaves differently per config:** at 2 guns it controls **helix amplitude** (tight braid at 0°, wide weave at 12°); at 3 guns it controls **fan angle**; at 1 gun it is inert and greys out.

Each gun's projectile is tinted by its attunement, so a 1-1-1 build reads as three colours in flight and a mono build reads as one.

---

## 4.5 Critical hits and Surge

### PRD crit

Crit uses **pseudo-random distribution**. Chance climbs each shot since the last crit, then resets. Streaks in both directions are impossible; the long-run rate matches nominal exactly.

```
P(n) = C × n     n = shots since last crit, reset to 0 on crit
```

| Nominal | C | Guaranteed by shot |
|---|---|---|
| 5% | 0.00380 | 264 |
| 10% | 0.01475 | 68 |
| 15% | 0.03222 | 32 |
| 20% | 0.05570 | 18 |
| 25% | 0.08474 | 12 |
| 30% | 0.11895 | 9 |

Ship the full 1–60% table as data; never solve at runtime.

**Every player has 5% crit for free.** This is load-bearing: nodes trigger on crit, so a zero baseline would leave a fresh player with dead nodes. Purchasable to 30%; hard cap from all sources is **60%**.

**Crit rolls once per volley, not per projectile.** All guns crit together — crits stay visible rather than becoming a statistical smear, and the Guns line doesn't become a secret crit-consistency upgrade. Note the spike: a 3-gun crit at full investment lands 2.1n × 2.5 = **5.25n in one frame**. Balance boss phases against that number, not against a single stream.

**Crit damage:** 1.25× → 2.50×. **Enemies never crit.**

### Crit cross-propagation

This is what makes attunement configuration matter beyond raw matchups.

> **On a crit, every gun's on-hit effect fires at amplified magnitude, and each effect cross-applies to every target touched by any other gun's effect.**

A PLASMA / CRYO / VOLT ship crits: CRYO splashes, and every target caught in that splash also takes PLASMA's Burn and VOLT's arc. A 3× PLASMA ship crits: no cross-propagation, but all three Burn stacks land at once at doubled magnitude.

Mixed builds get **breadth on crit**; mono builds get **depth on crit**. No special-case code — it falls out of each gun resolving independently.

### Surge

**Every 150 shots fired is a Surge.** Anchored to shots, not crits, because PRD makes crit timing unknowable in advance and Surge must be telegraphable.

| Point in game | Cadence |
|---|---|
| Early (2 shots/sec) | ~1 per 75s — roughly once a contract, a genuine event |
| Late (8 shots/sec) | ~1 per 19s |

The three shots before a Surge visibly and audibly brighten with rising pitch, giving ~0.4s of warning to point the ship at something worth hitting. Tier-3 nodes and tier-2 triples hook into it.

### All effects fire together

When a crit or Surge lands, **every equipped node's matching effect fires simultaneously.** There is no priority table and no arbitration. Competing effects would create builds that quietly don't work with no way for the player to see why. Balance by keeping each individual effect modest.

**The gun does not fire when no valid target is on screen** (Field mode: camera bounds + 10% margin). Saves battery, cuts audio fatigue, reads as intentional.

---

## 4.6 Enemy design — two dimensions

Every enemy is `archetype × role tags × element affinity`. Both extra dimensions default to null.

### Role tags (stackable)

| Tag | Effect | Sector scaling |
|---|---|---|
| *(null)* | Baseline | — |
| **fast** | ×1.6 speed, ×0.6 hull | →×1.9 speed by S5 |
| **tough** | ×2.5 hull, ×0.7 speed | →×3.5 hull by S5 |
| **repairing** | 3%/s regen after 2s undamaged | →6%/s by S5 |
| **mage** | One ability per placement from a pool | pool widens by sector |
| **decoy** | N 1-HP illusions mimicking its movement | N = 1 (S2), 2 (S3), 3 (S4–5) |
| **mob** | Group sharing one hull pool | 2:1 (S2) → 5:1 (S5) |

Tags stack. `fast + mob` is a five-strong swarm of quick chaff; `tough + repairing` is a kill-it-now problem. **Mage is deliberately open-ended** — its ability is chosen per placement in wave data, giving large authored variety from one tag.

### Element affinity progression

Affinity density rises across the campaign so matchup decisions grow in importance:

| Contracts | Null | 1 element | 2 elements | 3 elements |
|---|---|---|---|---|
| S1 c1–3 (tutorial) | **100%** | — | — | — |
| S1 c4–6 | 85% | 15% | — | — |
| Sector 2 | 60% | 35% | 5% | — |
| Sector 3 | 40% | 40% | 18% | 2% |
| Sector 4 | 20% | 35% | 35% | 10% |
| Sector 5 | 5% | 25% | 45% | 25% |

**A contract exposes at most 4 distinct elements**, and the fourth appears only rarely. That bound is what makes pre-contract attunement selection a real, solvable decision rather than a guess.

Affinity does not gate progress. The worst case a player can face is the clamp floor at 0.55×, and a non-owner facing DLC-affinity enemies loses only 0.10× per element (0.729× against a full triple), which is survivable and symmetrical — they take less damage too.

---

## 4.7 The type chart

Two closed cycles:

```
PLASMA → FORGE → VOLT → CRYO → CAUSTIC → PLASMA
AETHER → VOID → GAMMA → CHRONO → AETHER
```

| Relationship | Rationale |
|---|---|
| **PLASMA > FORGE** | Superheated plasma cuts armour |
| **FORGE > VOLT** | Mass grounds the arc |
| **VOLT > CRYO** | Charge conducts freely through ice and coolant |
| **CRYO > CAUSTIC** | Cold slows the reaction |
| **CAUSTIC > PLASMA** | Corrosion eats the containment |
| **AETHER > VOID** | No mass, so gravity has nothing to grip |
| **VOID > GAMMA** | A black hole swallows radiation |
| **GAMMA > CHRONO** | Lightspeed is the constant that bounds time |
| **CHRONO > AETHER** | A ghost is a recording, and you control the tape |

### Multipliers

| Relationship | Multiplier |
|---|---|
| Strong against | **1.30×** |
| Weak against | **0.75×** |
| **Across circles** (base ↔ DLC, either direction) | **0.90×** |
| Neutral, or either side null | 1.00× |

Multi-element defenders multiply pairwise results, then clamp:

```
final = clamp(Π multipliers, 0.55, 1.85)
```

Without the clamp a four-element enemy weak to your attunement reaches 2.86×. Bounds sit just past two-element perfection (1.30² = 1.69, 0.75² = 0.56).

Time-to-kill in Sector 5 against a 2.8s neutral baseline:

| Matchup | TTK |
|---|---|
| Perfect 2-element | 1.66s |
| Single strong | 2.15s |
| Neutral | 2.80s |
| Cross-circle | 3.11s |
| Single resist | 3.73s |
| Double resist | 4.98s |

### The cross-circle rule

A player without DLC facing DLC-affinity enemies deals 0.90× **and takes 0.90×** — net-neutral difficulty, fights roughly 11% longer. This is the structural anti-pay-to-win mechanism: DLC elements are never *better* against base content, only different.

DLC-affinity enemies appearing in the base campaign are therefore fine, and they double as the best advertisement in the game: a CHRONO-affinity enemy in Sector 4, with the Bestiary noting it resists conventional attunements, forms purchase intent *during* play, which is what `01` §1.7 requires.

Adding a directional cross-circle relationship is a pay-to-win risk and requires an ADR. Future element packs may extend either cycle.

---

## 4.8 Element levels, tiers and nodes

10 element points. Each raises one element's level, maximum 3. DLC packs grant +1 each.

An element's level does three things:
1. **Unlocks it for attunement slots** (level ≥1).
2. **Scales its on-hit effect** (1.0 / 1.5 / 2.0).
3. **Feeds node tiers.**

### The tier model

Following Element TD:

| Node kind | Tier formula | Max tier | Cost to max |
|---|---|---|---|
| **Single** | element level | 3 | 3 points |
| **Dual** | `min(both levels)` | 3 | 6 points (3+3) |
| **Triple** | `min(all three) − 1` | 2 | 9 points (3+3+3) |

A triple exists at tier 1 from 2-2-2, which also costs 6 points. **Breadth and depth cost the same**, so the tier curve does not punish spread — the 4-slot equip cap already does that.

**Tier scaling is 1.0 / 1.5 / 2.1** for singles and duals. Triples start higher and run 1.0 / 1.7.

What 10 points buys:

| Build | Tier-3 duals | Tier-2 triples | Tier-1 triples | Nodes available |
|---|---|---|---|---|
| 3-3-3-1 | 3 | 1 | 0 | 11 |
| 3-3-2-2 | 1 | 0 | 4 | 14 |
| 2-2-2-2-2 | 0 | 0 | 10 | 25 |

The wide build gets 25 options to choose 4 from plus full matchup coverage across five attunements. The deep build gets three maxed duals and a tier-2 triple. Both are legitimate.

### Node counts

| Kind | Base (5 elements) | Full (9 elements) | Authoring |
|---|---|---|---|
| Singles | 5 | 9 | **Composed** — one framework, math-scaled |
| Duals | 10 | 36 | **Bespoke** — the heart of the game |
| Triples | 10 | 84 | **Composed** from constituent duals |
| **Total** | **25** | **129** | |

**Composed triples** fire their three constituent duals' effects simultaneously with a shared amplifier, plus one signature line and one VFX overlay each. This is both cheaper and better design — a triple should feel like your three duals resonating, not an unrelated fourth thing.

### Single-element paths

Every single follows one framework, math-scaled per element:

| Element level | Grants |
|---|---|
| 1 | The on-hit attunement effect becomes available |
| 2 | An on-crit effect |
| 3 | A **mono-attunement bonus** — fires only when **all** guns carry this element |

That third tier is the counterweight to spreading: going all-in on one element earns something no mixed build can access.

### Loadout

**1 active + 3 passives**, drawn from unlocked nodes, **swappable free and instantly**. No node is boss-gated; element points are the only requirement.

---

## 4.9 The nine elements

| Element | Identity | Bias |
|---|---|---|
| **PLASMA** | Superheated plasma, ignition, burn | Damage over time. Strong vs clusters, weak burst |
| **CRYO** | Cryogenics, pressure, coolant | Control and sustain. The "I want to not die" element |
| **FORGE** | Mass, plating, inertia | Tanky and slow. Strong first pick for a new player |
| **VOLT** | Lightning, arc, static discharge | Highest ceiling, highest damage, fragile |
| **CAUSTIC** | Corrosion, solvent, decay | Ramping vulnerability. Rewards sustained focus on one target |
| **CHRONO** *(DLC)* | Time | Slow fields, position and hull rewind, cooldown reduction |
| **GAMMA** *(DLC)* | Radiation | Beams, armour-stripping Expose, blinds that break rammer lock |
| **VOID** *(DLC)* | Gravity | Micro black holes, curving projectiles, wreckage reanimation. Grants Blink |
| **AETHER** *(DLC)* | The luminiferous aether — a substance that turned out not to exist | Phase and incorporeality. Damage bypassing armour and shields, brief intangibility, effects persisting after a target dies |

The names are real physics vocabulary, slightly misapplied — the working slang of a mercenary company that does not quite understand the tech it flies. VOLT is a unit used as a substance; AETHER is a discarded theory pressed into service; GAMMA means "light" to a pilot who has never read the paper.

## 4.10 Node design rules

Combos are **named events, not stat modifiers**. If a node's description reads like a spreadsheet entry, redesign it.

**Every node's effect must express as primitives plus parameters** (see the PRIMITIVES tab of the inventory workbook). If it can, it is a JSON row and ships the same day. If it cannot, it is a code task requiring an ADR. Keeping the primitive vocabulary closed is what makes 129 nodes tractable and patchable.

**Every node needs unique VFX and SFX.** CI enforces it. Triples inherit their constituents' effects but need their own signature overlay.

Target ratio roughly **1 active : 3 passives**, matching the equip slots.

## 4.11 DLC packs

Each pack adds one element, +1 element point, and a **6-contract side sector**. Node additions compound: the first pack adds 16 nodes, the fourth adds 37.

**CHORUS OF WRECKS** (GAMMA + VOID + any): destroyed enemies reassemble as echo wingmen in fixed formation slots, mirroring your heading. Deterministic. The headline DLC feature.

**New Game+ ships with the first DLC pack**, not the base game.

## 4.12 Clauses and replay

Payout is anchored to contract index and never diminishes. Contract 3 pays contract-3 money forever, so farming early contracts is never optimal and no diminishing-returns penalty is needed.

**Clauses** are contract addenda you voluntarily accept for higher pay. **4 slots, +100% total cap.** Permanent, never consumable. All are deterministic parameter modifiers on existing wave data.

| Clause | Effect | Bonus |
|---|---|---|
| Dense | Enemy hull +25% | +15% |
| Swift | Enemy speed +20% | +15% |
| Elite | 2 waves become elite variants | +20% |
| Fragile | You have 1 life | +25% |
| Austere | No pickups spawn | +15% |
| Anchored | Recall disabled | +15% |
| Heavy | Enemy contact damage +50% | +15% |
| Silent | No off-screen threat chevrons | +10% |
| Swarm | Enemy count +40% | +20% |

Plus three **repeatable** performance bonuses at +15% each: no life lost, cleared with the Elite clause, no pickups collected.

**Clause and bonus multipliers apply to credits only.** Element points are never awarded by clauses, bonuses, or replays.

## 4.13 Balance framework

**Effective Power** = `DPS_sustained × (1 + survivability_index)`, every build within ±25% of the reference curve.

The sim covers the **matchup surface**: every attunement configuration and node loadout against every contract's affinity mix, verifying the affinity progression table in §4.6 and that no contract is unclearable at 0.75× effective damage.

Plus the permanent **pay-to-win gate** (`01` §1.7): the strongest base-element build lands within 15% of the strongest DLC build.

## 4.14 Anti-patterns

- **Don't let nodes multiply each other.** Additive within a category, multiplicative only across capped categories.
- **Don't add a node or upgrade that grants credits.**
- **Don't add stacking rules for duplicate attunements.** Independent per-gun application is the whole elegance.
- **Don't couple actives to crit or Surge.** They run on their own cooldowns.
- **Don't let an effect fall outside the primitive vocabulary** without an ADR.
- **Don't let a node be a number.**
