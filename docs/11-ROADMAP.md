> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 11 — Roadmap & Milestones

Assumes **~12 hrs/week solo** with meaningful AI assistance on code. Multiply by 0.5 for full-time, 1.5 for first-time-in-an-engine (which you are — plan for 1.5).

**Total: roughly 12–16 months to v1.0.** A normal, honest number for this scope.

---

## M0 — Control Prototype · 3 weeks · **GO/NO-GO GATE**

Nothing else matters if the controls don't feel good.

**Build:** grey boxes. A ship, one enemy, both zones, both modes' movement, rate-based rotation with soft anchor, tap detection with the 6dp suppression window, horizontal wrap, arena boundary, auto-fire, recall.

**Ship it to your phone in week one.** Not the emulator.

**Done when:**
- You can play 10 minutes without frustration.
- Three other people control the ship within 30 seconds, uninstructed.
- Movement and rotation simultaneously feels *good*, not merely possible.
- Taps never fire accidentally during a fumbled re-grip, and never nudge the ship.
- Soft anchor helps rather than fights.

**If it fails:** iterate 3 more weeks. Still failing at 6 weeks total → change the scheme and update `03`. Do not proceed on controls you don't love.

Also: git repo, `.cursor/rules`, **start the D-U-N-S application for Duffin Holdings LLC and create the Play Console account under it** (`09` §9.9 — the clocks start early), buy a cheap reference Android device.

---

## M1 — Vertical Slice · 8 weeks

One complete path through the game in miniature: **Sector 1**.

- Contracts 1–6 authored via the wave schema, including SHEARHEAD (miniboss) and THRESHER (boss)
- 3 enemy types (Mote, Lash, Spike)
- Damage pipeline with the fixed resolution order, PRD crit, raw-HP hull, shields, 3-lives system
- Hangar with 5 stat lines, purchase flow, live preview
- 1 element (PLASMA) with tiers 1–3, its single node, loadout with 1 active + 3 passives (mostly empty)
- Pre-launch screen with 2 clauses
- Save/load with migration scaffolding
- Localization scaffolding (`06` §6.7)
- Placeholder art (Kenney CC0) — **resist making art**

**Done when:** a stranger plays Sector 1, buys upgrades, beats a boss, spends an element point, and wants to keep going. This is the build you show people to learn whether the game is fun.

---

## M2 — Systems Complete · 10 weeks

Every *system* exists; content is thin.

- All 5 base elements, levels 1–3, all 25 base nodes under the Element TD tier model (dual = min, triple = min−1)
- The primitive vocabulary (`08` §8.3c) and the CI check that every effect resolves to it
- Composed-node system: singles and triples derive from one framework and from their constituent duals
- Loadout system, free swapping, element respec
- Full 14-line stat tree with arithmetic costs and tiered hull bands
- PRD crit with the shipped C lookup table; shot-anchored Surge at 150; crit cross-propagation
- Enemy role tags (fast/tough/repairing/mage/decoy/mob) and element affinity
- Type chart with clamping; CI checks for the ≥60%-null and 0.8×-floor rules
- All 8 enemy archetypes + elite variants
- Both contract modes fully featured (minimap, boundary, wrap, chevrons)
- All 9 Clauses (4 slots, +100% cap), performance bonus tracking, index-anchored payouts
- Data schemas locked and CI-validated (including node completeness and VFX uniqueness)
- Balance sim operational
- Encyclopedia with all seven elements shown, DLC ones greyed
- Settings + accessibility

**Done when:** you can build any of the 25 base nodes and play them through the ~8 contracts that exist. **No new systems work after this point** — only content, balance, and polish.

**This is where scope creep kills projects.** Every new idea goes to `13`, not into the code.

---

## M3 — Content · 12 weeks

The grind. Mostly authoring data.

- All 30 contracts across 5 sectors
- 5 minibosses (~3 days each) + 5 bosses (~2 weeks each)
- ~35 log entries, bestiary text, all node descriptions and tier tables
- Campaign map with 5 sector clusters and marked side-sector slots
- Full balance pass via sim + playtesting
- New Game+

**Done when:** the game is completable start to finish. Ugly and quiet, but a *game*.

Verify the length target: ~88 min flawless critical path, ~3 hours realistic first playthrough.

---

## M4 — Art & Audio · 10 weeks

- Modular ship kit — hulls, wings, engines, accents
- 8 enemies + 5 minibosses + 5 bosses
- **36 dual-node VFX + 9 attunement signature layers + 84 triple overlays** — non-negotiable, this is where build identity lives
- ~130 audio assets total (`10` §10.7.1); see the ASSET_INVENTORY tab of the workbook for the itemised 210-row list; most are 200ms, and Sonniss + one pack covers ~70% with light editing
- Three rendering paths (Flat / halos / bloom) with the shared energy-float mapping
- Backgrounds, wormhole tunnel shader, boundary shader
- UI theme, 3 colour-blind palettes
- Story panels with soft parallax
- Full audio pass: element sonic palettes, voice limiting, ducking, 10 music tracks
- `CREDITS.md` complete; CI check that no scene references `assets/proto/`
- **Juice pass** (`03` §3.8) — budget 2 full weeks
- Store assets, icon, wordmark, screenshots (**lead with the Hangar**)

**Start recruiting 12 closed testers now.**

---

## M5 — Backend, Ads, Store & Hardening · 6 weeks

- Firebase Auth + Firestore cloud save
- Play Billing + Cloud Function verification + RTDN + signed entitlement JWTs
- GCS signed-URL DLC delivery (scaffolded even though no DLC ships yet)
- AdMob at the single state-machine node, with frequency cap and entitlement suppression
- GA4 event taxonomy including the conversion proxies, BigQuery export verified
- Crashlytics
- Account deletion, privacy policy, Data Safety, GDPR consent
- Performance pass on the cheap reference device
- Firebase Test Lab device matrix
- **Closed testing: 12 testers, 14 continuous days** — a hard calendar dependency

---

## M6 — Launch · 4 weeks

- Bug triage from closed testing
- Final balance pass on real data
- Soft launch in 1–2 small markets (Canada, NZ, Ireland) for 2 weeks — also your first read on CPI and conversion
- Global launch
- **8 weeks of active bugfixing before starting any DLC.** The difference between a 3.8 and a 4.5 rating.

---

## Post-launch sequence

| Order | Item | Why |
|---|---|---|
| 1 | Bugfix + balance (8 weeks) | Ratings are set in month one |
| 2 | Element Pack: CHRONO | Proves the DLC pipeline with the mechanically simplest of the three |
| 3 | Element Pack: VOID | The wreckage-wingman feature is the biggest draw — save it until the pipeline works |
| 4 | Element Pack: GAMMA + Deep Lattice Bundle | Completes the set, enables CHORUS OF WRECKS |
| 5 | iOS port | Only if Android worked |

Each pack: 1 element, +1 element point, a 6-contract side sector, and 16–37 new nodes (mostly composed triples). ~6 weeks.

---

## Quality gates (end of every milestone)

| Gate | Threshold |
|---|---|
| Frame rate on reference device | ≥45 fps sustained at peak density, **on the Flat rendering path** |
| Unit tests | All passing |
| Data validation | All JSON valid; node set complete (129); every effect resolves to a known primitive; no duplicate node VFX/SFX |
| Balance sim | Every build within ±25% of the reference EP curve |
| **Matchup floor** | No contract is unclearable at 0.75× effective damage |
| **Affinity progression** | Every contract matches the sector density table (`04` §4.6); no contract exposes more than 4 elements |
| **PRD convergence** | 100k-trial test lands within 0.5% of nominal at every crit rank |
| **Single-clear sufficiency** | One clear of every contract funds the next contract at 55th-percentile skill (`04` §4.2 step 6) |
| **Pay-to-win** | Strongest base build within 15% of strongest DLC build |
| **Pledge tests** | Three static-analysis checks passing (`09` §9.4) |
| Crash-free sessions | ≥99.5% (from M5) |
| Accessibility | Playable end-to-end with Assist Aim + colour-blind palette + flash reduction + on-screen buttons |
| Length | Critical path within 15% of the 88-minute target |

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| **Controls don't feel good** | Medium | M0 exists to find out cheaply. Don't skip or shorten it |
| **Element scope explodes** | High | 5 base elements / 25 base nodes is the M3 cap. Composed singles and triples are what keep 129 total tractable — if you find yourself hand-authoring triples, stop |
| **Motivation collapses in M3** | **High** | Twelve weeks of authoring with no new toys. Mitigate: one polish task per week, monthly playtests with real people, and keep the prose→JSON Cursor loop fast so authoring stays cheap |
| **Bosses eat the schedule** | Medium | 10 bosses would have been 5 months. 5 bosses + 5 cheap minibosses is the compromise already baked in — don't erode it |
| **DLC pressure distorts balance** | Medium | The automated pay-to-win gate, running from M2 onward |

The third one is the actual killer of first games. Plan for it now, while you're excited.
