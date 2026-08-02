> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 02 — Design Principles: Allowed vs. Not Allowed

The immune system against scope creep and tonal drift. Run every proposal — from Cursor or from yourself at 2am — through here first.

---

## 2.1 The variance law

The rule is not "no randomness." It is:

> **Variance the player can average out within a single engagement is fine. Variance that decides an outcome is not.**

A critical hit at 8 shots/second converges in about a second — it's deterministic in practice, like Dota crit or Axe's Helix. A random drop that defines your build for 20 minutes never converges at all. The test is **frequency and stakes**, not the presence of a die roll.

| Allowed | Not allowed |
|---|---|
| High-frequency damage variance that averages out in <2s | Random drops with variable contents |
| Particle scatter, explosion shape, debris | "Pick 1 of 3" upgrade offers |
| Muzzle flash and idle animation variation | Random stat rolls on anything |
| Bark and voice selection | Wave composition varying between plays |
| Star field seeding | Enemy attack order varying between plays |

**Enemy waves, timing, paths, and pickups are 100% authored.** A shield pod at wave 3 of contract 7 spawns at the same coordinates every time. Players learn and route them; that's the point.

**Enemies never crit.** Player damage must be predictable enough to budget hull against. Telegraphed heavy attacks give the same spike with none of the frustration.

### On procedural generation

The rule is **no variation between plays of the same contract** — not "no generators."

| Approach | Verdict |
|---|---|
| Runtime generation (different layout each play) | **Banned.** Breaks learnability, reproducibility, and balance testing |
| Offline/seeded generation baked to data at build time | **Fine.** This is just an authoring tool. Generate a thousand candidates, ship the twelve good ones |
| Parametric authoring ("8 motes in an arc") | **Fine.** Already how the wave schema works |

## 2.2 Critical hits, Surge, and effect triggers

Crit uses **pseudo-random distribution** — chance climbs each shot since the last crit, then resets. `P(n) = C × n`. Streaks in both directions are impossible and the long-run rate matches nominal exactly. This satisfies §2.1: at 8 shots/sec it converges inside a second. Full spec and the C lookup table are in `04` §4.5.

- **5% crit is free for every player.** Load-bearing: nodes trigger on crit, so a zero baseline would leave a fresh player with dead nodes. Purchasable to 30%; hard cap from all sources **60%**.
- **Crit rolls per volley, not per projectile.** All guns crit together.
- **Every 150 shots fired is a Surge**, telegraphed by the three shots before it brightening. Anchored to shots rather than crits, because PRD makes crit timing unknowable in advance and Surge must be predictable enough to aim into.
- **Crit damage** 1.25× → 2.50×. **Enemies never crit.**

### The three effect layers

Each system owns one job, with no overlap:

| Layer | Owner | Character |
|---|---|---|
| **On hit** | Gun attunements (`04` §4.4) | Simple, continuous, one effect per element |
| **On crit** | Element nodes | The amplifier — bursts and spreads the on-hit effect |
| **On Surge** | Tier-3 singles and duals, tier-2 triples | The spectacle — rare, large, screen-affecting |

**Crit cross-propagation:** on a crit, every gun's on-hit effect fires at amplified magnitude and cross-applies to every target touched by any other gun's effect. Mixed attunements get breadth on crit; mono attunements get depth. This falls out of each gun resolving independently — no special-case code.

**Every equipped node's matching effect fires simultaneously.** No priority table, no arbitration. Competing effects would create builds that quietly do not work with no way for the player to see why. Balance by keeping each individual effect modest.

**The gun does not fire when no valid target is on screen** (Field mode: camera bounds + 10% margin).

## 2.2b Status effects

**All statuses stack independently. No status ever overrides, replaces, or suppresses another.** A burning enemy that gets chilled is burning and chilled. Same-status reapplication follows that status's own rule — refresh or add a stack.

This creates a readability problem that must be solved rather than ignored: nine status types across 60 simultaneous enemies cannot be shown with nine indicators.

**Rule: visual stacking caps at 2 even though mechanical stacking does not.** An enemy displays at most two status indicators — the two most recently applied — through blended rim-light colour and particle. Full status detail appears only on a targeted or player-adjacent enemy. The mechanics run at full fidelity underneath; only the display is capped.

## 2.3 Mid-run rules

**Not allowed:** any permanent build change during a contract. No level-up-pick-a-perk. No mid-contract shops. No roguelike anything.

**Allowed:** temporary, predictable, positionally-authored pickups. Maximum 4 types, all visually distinct at 32px.

| Pickup | Effect |
|---|---|
| Shield Pod | +1 shield charge |
| Coolant | Next **10 shots** are guaranteed crits |
| Repair Cell | +25% max hull |
| Salvage | +credits |

Coolant is 10 shots rather than one because at 8 shots/second a single-shot buff would be invisible.

## 2.4 Time and money rules

**Never, for any reason:** energy or stamina systems · timers that gate play · daily login rewards · streak mechanics · limited-time offers · purchasable in-game currency · randomized purchases · rewarded ads · notifications that sell something.

**Ads** are permitted only as specified in `01` §1.7: miniboss and boss defeat, skippable, frequency-capped, never on death or retry, removed by any purchase.

**Architectural enforcement:** there is no code path from a purchase to a credit balance. `Wallet.add_credits()` is callable only from `ContractCompleteController`. Enforced by a CI test (`09` §9.4).

## 2.5 Difficulty and failure

- **3 lives per contract.** Out of lives restarts that contract with 3 again. It can never reach a blocked state, so it is not an energy system.
- **No mid-contract checkpoints** in normal contracts — at ~2:30 the maximum loss doesn't warrant one, and it removes a whole category of state-serialization bugs. Boss contracts get one checkpoint at the phase-1 transition.
- **Credits banked at a boss checkpoint are kept.** Credits from a failed attempt are forfeit. Nothing already banked is ever taken.
- **Retry in ≤2 taps and ≤1.5 seconds.**
- **Failure must be attributable.** On death, 1.5s slow-motion of the killing blow with the source highlighted. This converts "unfair" into "I see it."
- **Difficulty options** (Standard / Veteran) change enemy HP and speed only, never composition. Freely switchable, **identical credit payouts** — paying Easy less punishes exactly the people who chose it. Hard-mode reward is cosmetic: records, titles, a map marker.
- **No difficulty paywalls**, ever.

## 2.6 Control rules

- **Two thumbs maximum**, never lifting either during normal play. Taps are brief interruptions the game must be survivable through.
- **Zone-anchored, not position-anchored.** Touch anywhere in your half.
- **Nothing important renders in the bottom 22%.**
- **Auto-fire always on.** No fire button, ever.
- **Tap and drag only.** No double-tap, no gestures, no long-press for anything non-optional.
- **The active ability is never coupled to crit.** It runs on its own cooldown. Coupling creates constant "am I wasting this" anxiety.

## 2.7 Complexity budget

| Thing | Cap |
|---|---|
| Equipped active abilities | **1** |
| Equipped passives | **3** |
| Passive procs firing per second | 4 |
| HUD elements | 7 |
| Stat upgrade lines | 14 |
| Enemy archetypes per contract | 5 |
| Simultaneous enemies | 60 |
| Total crit chance from all sources | 60% |
| Attunement slots (= gun count) | 3 |
| Visible status indicators per enemy | 2 |
| Attunement slots (= gun count) | 3 |
| Element multiplier, clamped | 0.55× – 1.85× |
| Distinct elements exposed per contract | 4 |
| Clause slots | 4 |
| Total Clause bonus | +100% |
| Bullets on screen | 400 |
| Colours carrying gameplay meaning | 6 |
| Total clause bonus | +150% |

The 1-active/3-passive cap is what makes 7 elements survivable later: DLC adds *options*, never on-screen complexity. When a cap is hit, something is cut before something is added. Mirror these into `.cursor/rules`.

## 2.8 The "does this belong" test

1. Which pillar does it serve? (None → cut.)
2. Does it introduce outcome-deciding variance? (Yes → redesign or cut.)
3. Can it be expressed in the `08` schemas? (No → it's a milestone, not a feature.)
4. Does it fit §2.7? (No → what gets cut to make room?)
5. Can a new player understand it within 3 seconds of seeing it? (No → needs a telegraph, sound, and colour, or needs cutting.)

## 2.9 Tempting things to resist

- **Runtime procedural contracts.** §2.1.
- **A third currency.** Two is correct: credits and element points.
- **Reintroducing a deterministic crit counter for base crits.** PRD is the decision; Surge runs on a shot counter. §2.2.
- **An effect that cannot be expressed as primitives plus parameters.** That is a code task needing an ADR, not a data edit.
- **Letting a status override another.** All statuses stack. §2.2b.
- **Adding stacking rules for duplicate attunements.** Each gun applies its effect independently. That emergence is the whole elegance.
- **Cross-circle type relationships beyond the flat 0.90×.** Any exception is a pay-to-win risk and needs an ADR.
- **Ship hulls with different base stats.** Doubles the balance surface for a cosmetic gain. If it ever happens, hulls change loadout *shape*, never raw stats.
- **Multiplayer / leaderboards.** Huge backend, anti-cheat, and moderation cost for a single-player build-crafting loop.
- **A second control scheme.** Two schemes means both are half-tuned.
- **Animated cutscenes.** Static panels with soft parallax (`05` §5.5).
- **Making DLC elements stronger.** Kills the pledge and the model (`01` §1.7).
- **Consumable clauses.** Creates hoarding anxiety and an inventory UI you don't need.
