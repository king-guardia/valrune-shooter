# 0000 — Decision Log

**Status: authoritative.** Where any document in `docs/` conflicts with an entry here, this file wins until that document is revised in Phase 7.

This is the running index. Load-bearing entries get expanded into full ADRs (`0001-`, `0002-`, …) in Phase 1c. Every entry records what changed and why, so a decision is never silently re-litigated.

Legend: **[R]** reverses a v0 decision · **[N]** new, absent from v0 · **[C]** confirms v0

---

## Scope and content model

### D-001 [R] Triples are bespoke; coverage may be sparse
v0 (`04` §4.8, `08` §8.3, `12`) required triple nodes to be *composed* automatically from their constituent duals, and listed "hand-author a triple" as a CI-enforced NEVER. The ability spreadsheets hand-author 36 triples with unique names and mechanics.

**Decision:** triples are bespoke. Composition is deleted. A combination with no authored ability simply has no node; the coverage report measures gaps rather than demanding completeness.

**Why:** the composed model was scope control, not design. Composed triples would have been recolours of their duals, which is precisely the failure mode `10` §10.8 warns about.

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
Undamageable and untargetable. Behaviour:

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

An auto-fired active uses your **current heading**. Directional abilities (solar beam, comet, spectral lance) lose most of their value on auto; self-centred and auto-targeting ones (mines, neutron stars, taser, lightning storm) lose nothing. The toggle is a real per-ability decision, not a strictly-better button.

Requires an `auto_fire_efficiency` parameter per ability.

### D-023 [R] Maximum cooldown is 2 minutes
Supersedes `misc_ideas` line 36 (1 minute). Matches the balance window, so a 2-minute ability fires exactly once per measured window. Cooldown becomes an explicit power lever.

---

## Geometry and constants

### D-024 [N] Playfield is 1000 units wide
Height derives from device aspect (~2200 on 20:9). Throat mode is exactly one screen, so `r1 = 120 units` reads as 12% of screen width and AoE hit count becomes computable:

```
expected_hits = density × (π·r² / playfield_area) × clustering
```

### D-025 [R] Movement splits into push and pull
A single `mr` band conflated two different behaviours (`misc_ideas` line 15 describes the tension).

- **`push_impulse`** — displacement in units over a fixed short window (~0.08s). Does **not** override the target's own movement
- **`pull_speed`** — sustained units/sec toward a source. The `gravity` status is what makes it override movement

### D-026 [C] Banded geometry constants
`r_short` / `r1` / `r2` / `r3` · `dis_short` / `dis1`–`dis4`, where `disN` matches `rN`'s reach · `w1` / `w2` · `TICK_DOT = 0.2s`. Tooltips say "short / medium / long" plus the unit count; data says `r2`. Band values proposed in Phase 1b.

---

## Balance method

### D-027 [N] Three metrics, no single power score
**Offence, defence, utility**, kept separate. Parity is checked per metric at **±15%** (v0 said ±25%).

### D-028 [N] Offence is a 120-second window
```
uses    = ceil(120 / (duration + cooldown))    # 120s cd → 1, 60s → 2, 30s → 4, 10s → 12
offence = actives:  uses × damage_per_use × auto_fire_efficiency (if auto-slotted)
          passives: continuous rate × uptime × 120
```
First use at t=0. `ceil` counts uses *starting* within the window, avoiding the artifact where a 120s ability appears to fire at both t=0 and t=120.

### D-029 [N] Defence resolves to effective HP
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

## Open — not yet decided

| # | Item | Blocks |
|---|---|---|
| O-01 | Percentage allowlist beyond crit chance, crit damage, Overclock | Phase 1b |
| O-02 | Status tag taxonomy — do the five tags carve the space correctly? | Phase 1d |
| O-03 | Chrono+Cryo passive replacement (slot vacated by D-019) | Content, not blocking |
| O-04 | Element point economy — 10 points, max level 3, 5 base elements needs 15 to max. Sparse coverage means some spreads unlock very little; the Hangar must show that before you spend | Phase 7 |
| O-05 | Cross-axis parity — how a pure-defensive ability compares to a pure-offensive one. Starting heuristic: abilities declare a role, parity checked within role, cross-role weights set once | Phase 4, deferred until real data |
| O-06 | Threat profile calibration — composition and hit size are guesses until M0 | M0 |
