> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 01 — Product Brief

## 1.1 Vision

A premium-feeling Android space shooter that respects the player's time, wallet, and intelligence. It borrows the build-crafting joy of Element TD — where the fun is discovering what happens when you combine things — and puts it inside a fast, tactile, neon shooter with controls designed for thumbs rather than ported from a gamepad.

**The full campaign is free.** Revenue comes from expansions that add elements, side-contract sectors, and skins. The player fantasy: *"I built this ship, and it does something nobody else's does."*

## 1.2 Design pillars

Every feature serves at least one. Features serving none get cut.

**P1 — Build identity over run luck.** Power comes from informed choices in a menu, never from what dropped. Two players at the same point should have visibly, audibly, mechanically different ships.

**P2 — Thumbs, not ports.** Two thumbs, portrait, bottom corners. No control needs a precise screen location. Nothing important sits under a thumb.

**P3 — Readable chaos.** At 200 objects on screen the player still knows where they are, where they're facing, and what will kill them next. A hard constraint on art direction, not a nice-to-have.

**P4 — Honest money.** The entire cost is visible before spending anything. No store selling numbers.

**P5 — Deterministic and fair.** Same contract, same inputs, same result. Failure is attributable.

## 1.3 Audience

**Primary:** 25–45, played WC3 custom maps / tower defense / arcade shooters, has money, has ~20 minutes at a time, actively resents modern mobile monetization.

**Secondary:** Bullet-hell and Vampire-Survivors-adjacent players who want a permanent progression spine instead of per-run randomness.

**Not the audience:** the whale-hunting F2P market. Do not design for retention metrics.

## 1.4 Session and length targets

| Unit | Target |
|---|---|
| Normal contract | ~2:30 |
| Miniboss contract | ~3:00 |
| Boss contract | ~5:00 |
| Sector (6 contracts) | ~18 min critical path |
| Flawless full run | ~90 min |
| **Realistic first playthrough** | **~3 hours** (retries + Hangar + menus) |
| With clauses and challenges | 8–15 hours |

Every contract completable in under 5 minutes. Nothing ever requires keeping the app open while not playing.

## 1.5 Scope guardrails (v1.0)

| Dimension | v1.0 | Deferred |
|---|---|---|
| Platform | Android phone, portrait | iOS, tablet, landscape, PC |
| Elements | 5 (PLASMA, CRYO, FORGE, VOLT, CAUSTIC) | CHRONO, GAMMA, VOID, AETHER |
| Element points | 10 | +1 per DLC pack |
| Content | 5 sectors × 6 contracts = 30 | Side-contract sectors (DLC) |
| Bosses | 5 bosses + 5 minibosses | |
| Modes | Throat (vertical), Field (arena) | Endless, timed, escort |
| Enemies | 8 archetypes × 6 role tags × element affinity | |
| Online | Cloud save + entitlements | Leaderboards, multiplayer |
| Localization | English (built for expansion) | Everything else |
| Ads | Minimal (see §1.7) | Rewarded ads — never |

Anything not in the left column goes to `13-OPEN-DECISIONS.md`, not into the code.

## 1.6 The pledge

Published on the store listing and in-app. Architecturally enforced (`09` §9.4).

> - The full campaign is free. There is no paywall and no content lock on the main game.
> - There is no currency you can buy. Credits are earned by playing, only.
> - No loot boxes, gacha, or randomized purchases of any kind.
> - No energy meters, stamina, timers, or anything that asks you to stop playing or pay to continue.
> - Ads appear only after defeating a miniboss or boss — about ten times in a full playthrough. Any purchase removes them permanently.
> - Expansions add content at a fixed, visible price and are never required to finish the game.
> - Nothing is time-limited or FOMO-driven.

## 1.7 Business model

| SKU | Price | Contains |
|---|---|---|
| **Supporter Pack** | $2.99 | Removes ads, 3 skins, name in credits |
| Element Pack: CHRONO | $4.99 | 1 element, +1 element point, a 6-contract side sector, removes ads. Node additions compound: the first pack adds 16, the fourth adds 37 |
| Element Pack: VOID | $4.99 | as above |
| Element Pack: GAMMA | $4.99 | as above |
| **Deep Lattice Bundle** | $12.99 | All three packs + 4 exclusive skins |

**Ads:** skippable interstitial on **miniboss defeat** and **boss defeat** only — 10 slots per playthrough, roughly one per 18 minutes. Frequency-capped at one per 10 minutes regardless of triggers. **Never on death, retry, contract start, or menu entry.** Removed permanently by any purchase.

Ad removal is bundled as a bonus, not pitched as the driver. At ~10 impressions per playthrough, ads are not annoying enough to motivate a purchase on their own, and pretending otherwise would mean making them annoying. Content is the pitch.

**No rewarded ads.** Time-for-currency is the same shape as buying currency and breaks §1.6.

### Why full-free rather than a demo paywall

Removes the single biggest drag on paid acquisition, makes the whole install base addressable for DLC, produces a genuinely shareable claim, and eliminates the review damage a paywall generates. The bet is on install volume rather than on conversion pressure.

**The consequence:** purchase intent must form *during* play, not at the end. Only ~20% of installs reach the finale, so waiting until then to surface DLC wastes most of the funnel. Therefore:

- The Encyclopedia's Core tab shows **all seven elements from first launch**, greyed, with full mechanical descriptions.
- The combo web shows locked nodes.
- Side-contract sectors appear on the campaign map from early on, marked as expansions.

This was already required for anti-gambling reasons (`02` §2.4). It is now also load-bearing for revenue.

### Pay-to-win guardrail — automated, permanent

When DLC is the only revenue, there is constant quiet pressure to make new elements *stronger* rather than *different*. Write the test now and run it in CI forever:

> **The strongest base-element build must land within 15% of the strongest DLC build**, by Effective Power at 3, 6, and 10 element points, measured by the balance sim (`04` §4.13).

DLC elements win on novelty and expressiveness, never on raw power.

## 1.8 Success criteria

1. It ships.
2. Median sector-1 completion among installs >55%.
3. Campaign completion among installs >18%.
4. Install→purchase conversion >2.5%.
5. Crash-free sessions >99.5%.
6. You still like the codebase enough to build the first element pack.

Not success criteria: DAU, retention curves, ARPDAU.
