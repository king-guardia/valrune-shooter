# TASKS

Current work only. Decisions go in [`docs/decisions/0000-decision-log.md`](docs/decisions/0000-decision-log.md);
ideas that aren't happening yet go in `docs/13-OPEN-DECISIONS.md` §13.5.

---

## Phase 0 — Repo restructure ✅

- [x] Create the directory tree
- [x] Import 14 v0 plan documents with status banners
- [x] Import the 6 source spreadsheets to `data/source/`
- [x] Remove the stale marketing-game stub
- [x] Write `.cursor/rules` against current decisions
- [x] Write the decision log (34 decisions, 6 open items)
- [x] Root files: README, TASKS, CREDITS, .gitignore
- [x] Rename GitHub repo and git remote to `valrune-shooter`
  - Local folder stays `valrune-invaders` on purpose — see D-034

## Phase 1 — Glossary, constants, ADRs

### 1a — Canon ✅ **ACCEPTED**

- [x] `docs/14-CANON.md` — 11 sections, every term with a code identifier
- [x] `basic attack` vs `ability` vs `contact damage` — the D-017 blocker, resolved
      on **rate vs cooldown**
- [x] Bryan's review pass — 27 decisions recorded (D-035 to D-061)

The gate is open. Phase 2 can proceed on vocabulary.

### 1b — Constants ✅ **ACCEPTED**

- [x] `docs/15-CONSTANTS.md` — playfield, entity sizes, all band ladders, push/pull,
      speeds, time, the Expanse, and every rank line needing numbers
- [x] Ladders respaced against a **count of real usage** in the spreadsheets:
      `r1` appears 51 times, `distance_3` is authored as "max distance"
- [x] Resolved O-01, O-14 (Assist Aim cut), O-16 (letterbox), O-19 (ladder respaced)
- [x] Bryan's review pass — `crit_chance` 1.0% → 4.0% (D-062 to D-072)
- [x] **Revision 3: geometry rescaled against a measured reference screenshot.** Valrune
      100 × 110, all ladders respaced, `wingspan`/`fuselage` into canon, push/pull bands
      deleted in favour of composed pairs, velocity inheritance cut, homing denominated per
      unit travelled and bounded by `dis4` (D-073 to D-079)
- [x] Resolved O-21 (clustering, split blind/aimed and measured) and O-23 (dissolved)
- [x] **Revision 4:** faction `HORROR` → `UNFORMED`, homing capped on lateral reach at 3.5°,
      new `piercing` rank line closing the AoE gap (D-080 to D-082). Resolved O-22

Everything still open is an **M0 measurement, not a desk decision** — Expanse size,
`HOMING_BASE` and ceiling, `VALRUNE_SPEED_BASE`, and the 200-baddie performance claim.

> **Carried into Phase 4:** baddie counts tripled, so every AoE ability is worth 6–14× what
> the previous numbers implied. No authored damage value has been checked against this.

### 1c — Statuses (drafted)

- [x] `docs/16-STATUS-EFFECTS.md` — 35 statuses, 20 families, tags, ImmunitySet/OverrideSet
- [x] Resolved O-02 (six tags, not five) and O-13 (`invisible` → GAMMA)
- [x] Audited all 17 debuffs against D-016 — 15 matched, so the 51 immune booleans become
      **generated** from tag plus form rather than authored
- [x] 10 decisions recorded (D-083 to D-092)
- [x] **Revision 2** — duration policy (riders fixed, payloads from source), stack-ramp model
      for the Balance Lab, lockout family collapsed. `stasis` deleted, `paralyze_plus` added
      (D-093 to D-098). Resolved O-29, O-31, O-32
- [x] **Revision 3** — stacking debuffs reach elites, `radiate` becomes flat bonuses, gravity
      fields bend projectiles (D-099, D-100). Resolved O-28, O-33, O-34
- [ ] **Bryan reviews conflicts §7.1, 7.2, 7.4, 7.5, 7.7, 7.9, 7.11** — the ones not yet
      confirmed. The two that are bugs rather than inconsistencies: base `override` is a
      no-op, and the `Shield` status collides with the `shield` rank line

Three questions remain, all deferrable: is `rime`'s recoil a basic attack, does anything strip
a ward, and how many gravity fields may coexist.

### 1d — ADRs ← **NEXT**

- [ ] Expand load-bearing decisions into full ADRs (D-007, D-009, D-015, D-019, D-029)

## Phase 2 — Data model and gap report

- [ ] JSON schemas: ability, status, attunement, drone, field object, constants
- [ ] Convert 8 pilot abilities covering every distinct shape → **Bryan reviews the model**
- [ ] Bulk-convert the remaining 81 abilities, 30 statuses, 9 attunements
- [ ] `tools/coverage` — slot coverage vs targets, per-element triple participation, orphan and neglect detection
- [ ] **Bryan authors 4+ base triple abilities, at least one containing Caustic**

## Phase 3 — UML

- [ ] Entity model · ability model · status model with immunity resolution
- [ ] Damage-flow sequence diagram for one gun-shot
- [ ] Baddie action state machine (overlapping move/idle/shoot, telegraph windows)
- [ ] Drone behavior state machine · deferred-damage queue
- [ ] **Bryan reviews and critiques**

## Phase 4 — Balance Lab

- [ ] `tools/balance/engine.js` — shared math, imported by lab and agents
- [ ] `balance/balance-lab.html` — read-only sandbox
- [ ] Resolve O-05: cross-axis parity rule
- [ ] Resolve O-06: threat profile calibration (blocked on M0)

## Phase 5 — Agents

- [ ] `.cursor/skills/coverage-auditor`
- [ ] `.cursor/skills/balance-calibrator` — asks filtered / standard / full, flags beyond ±15%, never auto-applies

## Phase 6 — Dashboard

- [ ] `dashboard/index.html` + hosted gap report
- [ ] `gcloud auth login bryan@valrune.com`
- [ ] Firebase Hosting on a valrune.com subdomain

## Phase 7 — Doc revision

- [ ] Revise `docs/00`–`13` against the decision log; remove the v0 banners as each lands
- [ ] Delete every Surge reference · rename Aether → Ether
- [ ] Resolve O-03 (Chrono+Cryo replacement) and O-04 (element point economy)
- [ ] Fold per-baddie immunity grants into `04`'s role-tag section

## Then

- [ ] **M0 — control prototype.** Three weeks, grey boxes, on a real phone. Go/no-go gate.
