> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 13 — Open Decisions & Identified Gaps

## 13.1 Settled

| Decision | Resolution |
|---|---|
| Title | **VALRUNE** — also the ship's name and the rig at its core |
| Engine | Godot 4.7, GDScript |
| Business | Campaign 100% free; DLC = elements + side sectors + skins + ad removal |
| Ads | Miniboss and boss defeat only, 10/playthrough, skippable, capped at 1 per 10 min, never on death, removed by any purchase |
| Structure | 5 sectors × 6 contracts = 30; contract 3 miniboss, contract 6 boss |
| Naming | **Sectors** (regions) · **Contracts** (levels) · **Clauses** (difficulty addenda) |
| Length | ~2:30 normal contracts, ~88 min flawless path, ~3 hr realistic first playthrough |
| Elements | Base: PLASMA, CRYO, FORGE, VOLT, CAUSTIC. DLC: CHRONO, GAMMA, VOID, AETHER |
| Type chart | PLASMA→FORGE→VOLT→CRYO→CAUSTIC→PLASMA · AETHER→VOID→GAMMA→CHRONO→AETHER. **1.30× / 0.75×**, flat **0.90× across circles**, clamped **0.55–1.85** |
| Tier model | Element TD: single = level, dual = `min(both)`, triple = `min(all) − 1` cap 2. A tier-3 dual and a tier-1 triple both cost 6 points |
| Tier scaling | 1.0 / 1.5 / 2.1 singles and duals; 1.0 / 1.7 triples. **Flat, because breadth and depth cost the same** — the 4-slot equip cap does the limiting |
| Nodes | 129 total (9 singles, 36 duals, 84 triples). **36 bespoke duals; singles and triples composed** |
| Single paths | Level 1 = on-hit · level 2 = on-crit · level 3 = **mono-attunement bonus** (fires only if all guns carry that element) |
| Guns / attunements | 1 / 2 / 3 guns = 1.0 / 1.6 / 2.1n, split 1.1+0.5+0.5 at three. Each gun is an attunement slot; duplicates and nulls allowed; free re-slot |
| Effect layers | On-hit = attunements · on-crit = nodes · on-Surge = tier-3 singles/duals and tier-2 triples |
| Crit cross-propagation | On a crit, each gun's effect amplifies and cross-applies to every target touched by any other gun's effect |
| Crit | **PRD**. 5% free baseline, purchasable to 30%, 60% hard cap, 1.25–2.50× damage, rolls per volley |
| Surge | **Every 150 shots fired**, telegraphed 3 shots ahead |
| Statuses | **All stack independently, nothing overrides.** Visual indicators cap at 2 per enemy |
| Power curve | **57× DPS** stat tree, ~68× with elements. 50× hull vs 15× enemy damage = 3.3× base survivability, ~10× defensive |
| Enemy scaling | 70× hull, 15× damage, 3.5× count — TTK near 2.5s |
| Enemy design | `archetype × role tags × element affinity`. Tags: fast, tough, repairing, mage, decoy, mob |
| Affinity progression | 100% null in contracts 1–3, declining to ~5% null in Sector 5. Max 4 distinct elements per contract |
| Progression axes | Two: credits + element points. No pilot level, no XP |
| Element point gating | 0 until Miniboss 1, 1 until Boss 1, all thereafter — DLC included |
| Credit gain | **No upgrade or node increases it.** No greed stats |
| Cost scaling | Arithmetic. Total sink 30,477; income 60% coverage on a single clear |
| Respec | **100 credits flat**, no scaling, stats or elements |
| Mobility | Thrusters and Gyros to **4× base**, 20 ranks each, with a 0.5×-to-purchased-max slider |
| Lives | 3 per contract, restart with 3 again; boss phase-1 checkpoint only |
| Boss gating | **No nodes are boss-gated.** Element points are the only requirement |
| Story | Panels after minibosses and bosses only — 10 panels, ~400 words |
| Final boss | Throat only; no mid-fight arena switching |
| Side sectors | 6 contracts, matching base sector length |
| New Game+ | Ships with the first DLC pack |
| Primitives | Closed vocabulary of 50. Every effect must resolve to one, or it is a code task with an ADR |
| Entitlement | **Permanent once verified locally.** Re-verification gates only new DLC downloads and cloud save |
| Play Integrity | Not a store badge and has no listing benefit. If used at all, gates cloud save only |
| Play account | **Organization account under Duffin Holdings LLC** — skips the 12-tester gate. Needs a D-U-N-S number, ~30 days, start at M0 |
| Colour reference | **Radiant Defense** — flat saturated hues on near-black, colour assigned by function |
| Test devices | Old phones on hand, plus an emulator |
| Leaderboards / multiplayer / controller | Deferred |

## 13.1b Resolved from the previous pass

| # | Item | Resolution |
|---|---|---|
| A | Shield hit costs | 500 / 1,500 / 3,000 |
| B | Surge anchor | Every 150 shots fired. Telegraphing requires determinism, which crit-anchoring cannot provide |
| C | Damage tier overlap | Fixed: 7 ranks ending 17–21, every floor equal to the previous ceiling. Stat tree yields 57×; element effects carry the rest |
| D | Bulwark naming | Adopted for both defensive lines |
| E | Mage ability pool | Deferred to M2 by agreement |
| F | Cross-circle relations | Flat 0.90× both directions. Future packs may extend either cycle; a directional relationship needs an ADR |
| G | FORGE name | **Kept.** "Earth" reads as nature or organic in fantasy; FORGE is the industrial bastardization, which is exactly the register |
| H | Combo node names | Weather register dropped. Names should be merc or physics jargon fitted to the specific effect |
| I | Thin single-element nodes | Solved by the three-level single path |
| — | Same-type gun bonuses | Now the level-3 mono-attunement bonus |

## 13.2 Still needs your call

| # | Decision | Blocks | Recommendation | By |
|---|---|---|---|---|
| 1 | **Trademark-search VALRUNE** | Store account, art, wordmark, domain | Play Store, Steam, USPTO TESS, domain availability | M1 |
| 2 | **D-U-N-S number for Duffin Holdings LLC** | Play Console org account, which skips the 12-tester gate | Free from Dun & Bradstreet, ~30 days. Start at M0 | M0 |
| 3 | **Final SKU prices** | Store setup | $2.99 / $4.99 × 3 / $12.99 as hypothesis; revisit with soft-launch data | M5 |
| 4 | **Cloud save in v1.0?** | The whole M5 backend block | Yes, but it's the safest thing to cut if M5 overruns | M4 |
| 5 | **Mage ability pool** | Enemy authoring | Five to start: ally buff, player debuff, ranged attack, ally shield, minion summon | M2 |
| 6 | **Art: DIY vs commissioned style bible** | M4 budget | DIY with placeholders through M2, decide at M3 based on how your ships look | M3 |
| 7 | **Node naming pass** | 129 names + flavour | Merc and physics jargon fitted per effect; you take a pass after the first draft | M2 |

## 13.3 Calls I made that you should sanity-check

1. **Crit rolls per volley, not per projectile.** Makes crits visible and stops the Guns line being a secret crit-consistency upgrade — but a 3-gun crit lands 2.1n at once, so tune crit damage against that spike, not against a single stream.
2. **Horizontal wrap in Throat mode.** A real departure from Space Invaders. Removes cornering pressure. Play both in M0.
3. **Leaked enemies wrap back around** rather than damaging you — removes a fail state you might want.
4. **Blink targets along heading, not by tap.** Avoids making tap/drag disambiguation load-bearing under pressure.
5. **Element multipliers apply only to node damage, never the base gun.** Guarantees a damage floor for every build; the cost is that matchups matter less for stat-heavy builds than element-heavy ones.
6. **Tier curve 1.0 / 2.4 / 5.0.** Aggressive enough that spreading thin is clearly worse. If specialization feels *mandatory* rather than *rewarded*, flatten toward 1.0/2.2/4.0.
7. **Node tier = min(), with no unlock gating.** Four tier-1 elements unlock all 14 nodes. Intentional — the 4-slot loadout is the real constraint.
8. **Only 4 actives among 14 nodes.** Active-heavy players may feel short of options.
9. **No mid-contract checkpoints** in normal contracts.
10. **Same credits on every difficulty.**
11. **Skipping Play Integrity.** More tolerant of piracy in exchange for never false-positiving a custom-ROM user.

## 13.4 Gaps worth flagging

| Gap | Why it matters |
|---|---|
| **Photosensitivity** | A neon shooter is a genuine seizure risk. The Flat rendering path is also the accessible path, and the flash toggle belongs on first run |
| **The 12-tester gate** | 3+ weeks of calendar; catches most first-time Android devs by surprise |
| **Keystore backup** | Lose the Android signing key and you can never update the app. Two backups, one offline. Distinct from entitlement tokens — the keystore signs the app itself |
| **Localization scaffolding** | 4 hours at M1, 3 weeks retrofitted |
| **Account deletion flow** | Play policy requirement |
| **Data Safety accuracy** | AdMob means declaring an advertising ID. Mismatches get apps pulled |
| **A cheap real test device** | Old phones on hand cover this. Confirm at least one is a low-end 2021-or-earlier device — the emulator will lie about performance |
| **Save migration function** | Write it before you need it, or the first content patch wipes saves |
| **RTDN refund handling** | Without Pub/Sub, refunded purchases stay unlocked permanently |
| **Battery and thermal** | Test a 45-minute session. A phone that gets hot gets uninstalled |
| **Interruption handling** | Calls, notifications, app switching mid-boss. Instant pause, clean resume |
| **What happens after contract 30** | New Game+ ships with DLC 1. Extra NG+ contracts are a later question |
| **Server-side purchase records** | Client purchase events are lossy and spoofable; you'll need the server number to reconcile against ad spend |
| **Sector 5 audio density** | Nine statuses plus dozens of node signatures can stack a dozen sounds per second. Priority tiers in `10` §10.7.4 |
| **Status visual capping** | Mechanically all statuses stack; visually only 2 show. Verify this reads correctly at 60 enemies during M4 |

## 13.5 Parking lot

Adding here is how you say no without losing the idea.

- Endless / survival mode with a leaderboard
- Weekly authored challenge contracts (deterministic, so no new systems needed)
- Consumable Clauses
- Nodes granting both an active *and* a passive
- Ship hulls that change loadout shape
- Multiple saved loadout presets
- Photo mode / replay export
- Controller support (trivial in Godot; nice for Android TV or a Backbone)
- iOS
- Tablet / landscape
- A player-facing contract editor
- Achievements via Play Games Services
- Regional ad strategy for low-ARPU markets (revisit with real data, not estimates)

## 13.6 Next steps

1. Read `02-DESIGN-PRINCIPLES.md` and flag disagreements — it constrains everything downstream, so objections are cheapest now.
2. Answer decisions 1 and 2 in §13.2.
3. Install Godot 4.7. Create the repo. Drop `docs/` in it. Build `.cursor/rules/` from `12`.
4. Create the Play Console account and buy the cheap test device.
5. **Build M0. Nothing else.** Three weeks, grey boxes, your actual phone.

Then revise these documents against what M0 taught you — because it will invalidate part of this plan, and that's the point of building it first.
