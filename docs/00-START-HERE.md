> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# VALRUNE — Plan of Plans

**Title:** VALRUNE — also the name of the ship you fly, and the rig at its core.

**Pitch:** A neon top-down space shooter for Android. The full campaign is free. You fly for a mercenary company through a collapsing wormhole network, building your ship from combinable elemental attunements — no gacha, no currency store, no timers.

---

## 0.1 What this is

A plan of plans: each file is a self-contained design or engineering document, written to be handed to Cursor as the project's source of truth. Decision-complete where possible, with open items isolated in `13`.

Companion deliverable: **`valrune-inventory.xlsx`** — the primitive vocabulary, 115 cross-game mechanic references, all 129 nodes awaiting effects, and a 210-row asset inventory.

## 0.2 Reading order

| # | File | Settles |
|---|------|---------|
| 01 | `01-PRODUCT-BRIEF.md` | Vision, pillars, business model, the pledge |
| 02 | `02-DESIGN-PRINCIPLES.md` | **Allowed vs. not allowed.** Binding on everything else |
| 03 | `03-CORE-GAMEPLAY.md` | Controls, modes, edge rules, enemies, combat feel |
| 04 | `04-PROGRESSION-AND-ELEMENTS.md` | Calibration, stat tree, element points, nodes, clauses |
| 05 | `05-WORLD-AND-CAMPAIGN.md` | Story, 5 sectors × 6 contracts, bosses, Encyclopedia |
| 06 | `06-UX-AND-SCREENS.md` | Screen state machine, HUD, accessibility |
| 07 | `07-TECH-ARCHITECTURE.md` | Godot 4.7, project layout, systems, save |
| 08 | `08-DATA-SCHEMAS.md` | JSON contracts — the content pipeline |
| 09 | `09-BACKEND-MONETIZATION-SECURITY.md` | GCP, Play Billing, entitlements, ads, analytics |
| 10 | `10-ART-AND-AUDIO.md` | Art direction, non-designer pipeline, palette, audio |
| 11 | `11-ROADMAP.md` | M0–M6, quality gates, risks |
| 12 | `12-CURSOR-WORKFLOW.md` | Rules files, task granularity, guardrails |
| 13 | `13-OPEN-DECISIONS.md` | What's still open |

---

## 0.3 The settled shape

**Structure.** 5 **sectors** × 6 **contracts** = **30 contracts**. Contract 3 of each sector is a miniboss, contract 6 is a boss. Target ~2:30 per normal contract; realistic first playthrough ~3 hours including retries and menus. A sector is a region, not a place — it can hold a planet's orbit, a meteor field, and a wormhole junction without needing a reason they are adjacent.

**Business.** Campaign 100% free, no paywall, no content locks. Revenue from DLC: element packs, side-contract sectors, skins, and ad removal. Minimal ads (miniboss + boss defeat only, 10 per playthrough).

**Progression.** Two axes: **Credits** (stat tree) and **Element Points** (10 total, one per miniboss and boss, ungrindable). No pilot level, no XP. Respec is 100 credits flat. **No upgrade or node increases credit gain** — greed stats compel players who feel obliged to max them and distort every balance decision downstream. Element points are unusable until Miniboss 1 and capped at 1 until Boss 1, so DLC buyers cannot frontload.


**Power curve.** JRPG scale: **57× DPS** from the stat tree alone, ~68× with element effects. Hull grows 50× against 15× enemy damage for **3.3× base survivability**, reaching ~10× for a defensive build. Enemy hull grows 70× — slightly ahead of the player — so time-to-kill holds near 2.5s and **matchups are the difficulty lever** rather than raw numbers. Chrono Trigger's shape without its treadmill.

**Costs.** Arithmetic, not geometric: `cost(n) = base + step × (n−1)`. Cumulative cost is quadratic against linear benefit, which soft-caps naturally without exponents. No number in the game exceeds four digits.

**Controls.** Left thumb drags a screen-relative movement stick; left tap recalls to the ready line. Right thumb drags a rate-based rotation bar with a soft anchor at north; right tap fires the active ability. Auto-fire always on.

**Crit.** Pseudo-random distribution, the Dota technique — chance climbs each shot since the last crit, then resets. **5% free baseline** (nodes trigger on crit, so a zero baseline would leave dead nodes), purchasable to 30%, hard cap 60%. Rolls per volley, not per projectile. **Every 150 shots fired is a Surge**, telegraphed three shots ahead — anchored to shots because PRD makes crit timing unknowable in advance.

**Guns are attunement slots.** 1 / 2 / 3 guns deal 1.0 / 1.6 / 2.1n, with the three-gun split asymmetric at 1.1 + 0.5 + 0.5 — so the centre slot is worth 2.2× a wing. Each gun holds any owned element (duplicates and nulls allowed), applies that element's on-hit effect independently, and carries its own element for matchups, so hedged configs earn partial credit. Two guns **helix**. Re-slotting is free.

**Three effect layers.** On-hit belongs to gun attunements. On-crit belongs to nodes. On-Surge belongs to tier-3 singles and duals and tier-2 triples. **On a crit, each gun's effect cross-applies to every target touched by any other gun's effect** — mixed attunements get breadth, mono attunements get depth.

**Elements.** Base: PLASMA, CRYO, FORGE, VOLT, CAUSTIC. DLC: CHRONO, GAMMA, VOID, AETHER. Real physics vocabulary, slightly misapplied — the working slang of a mercenary company that does not quite understand the tech it flies. VOLT is a unit used as a substance; AETHER is a discarded theory pressed into service.

**Element system.** Element TD's model: single tier = element level, dual tier = `min(both)`, triple tier = `min(all three) − 1`, capped at 2. A tier-3 dual and a tier-1 triple both cost 6 points, so **breadth and depth cost the same** — the 4-slot equip cap is what makes specialization a real choice, not the tier curve. 129 nodes total: 36 bespoke duals, with singles and triples composed.

**Statuses all stack.** Nothing overrides anything. Visual indicators cap at 2 per enemy; the mechanics run at full fidelity underneath.

**Enemies** are `archetype × role tags × element affinity`. Both extra dimensions default to null and null is the common case (≥60% of every contract). Tags (fast, tough, repairing, mage, decoy, mob) stack. Affinity feeds a two-cycle type chart at 1.30× / 0.75×, with a flat 0.90× across circles so DLC elements are never better against base content — including VOID beating GAMMA, since a black hole swallows radiation.


**Lives.** 3 per contract. Out of lives restarts the contract with 3 again. No mid-contract checkpoints except boss phase transitions.

**Replay.** **Clauses** are contract addenda you accept for higher pay — 4 slots, +100% cap — plus three repeatable performance bonuses at +15%. Payout is anchored to contract index, so farming early contracts is never optimal and no diminishing-returns penalty is needed.

## 0.4 Scope honesty

Roughly **12–16 months at ~12 hrs/week.** The largest risks are, in order: the control scheme not feeling good (M0 exists to find out cheaply), element combinatorics expanding (capped at 4 base elements = 14 nodes), and motivation collapsing during M3's content grind.

Build M0 before committing to any of this.

## 0.5 Glossary

| Term | Meaning |
|---|---|
| **The Lattice** | The wormhole network; the setting |
| **The Hollow** | The enemy faction |
| **Throat** | Wormhole interior — the vertical contract type |
| **Field** | Open arena — the free-range contract type |
| **Attunement / Element** | PLASMA, CRYO, FORGE, VOLT (+3 DLC) |
| **Element Point** | Raises an element's level; from minibosses and bosses only |
| **Role tag** | An enemy modifier: fast, tough, repairing, mage, decoy, mob |
| **Affinity** | The elements an enemy carries, feeding the type chart |
| **PRD** | Pseudo-random distribution — the bounded crit system |
| **Primitive** | A mechanic the engine supports. Every node effect must resolve to one, or it is a code task |
| **Node** | An ability unlocked by holding a specific element, pair, or triple |
| **Credits** | Currency for the universal stat tree. Earned only |

