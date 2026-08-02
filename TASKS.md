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

**1a is a hard gate. Nothing in Phase 2 starts until Bryan has corrected the glossary.**

- [x] `docs/14-CANON.md` drafted — 11 sections, every term with a code identifier
  - [x] People and ships · baddies · spawned objects · weapons · elements
  - [x] Progression · ability anatomy · damage · statuses · geometry · structure · behavior
  - [x] `basic attack` vs `ability` vs `contact damage` — the D-017 blocker
  - [ ] **Bryan reviews and corrects — 9 open questions at the bottom of the file**
    - Only Q3 (is GAMMA's "beam" a real continuous weapon?) has downstream cost;
      it decides whether a hitscan damage path is needed at all. The rest are naming.
- [ ] `docs/15-CONSTANTS.md` — band values, playfield units, variable naming
  - [ ] Resolve O-01: the percentage allowlist
- [ ] `docs/16-STATUS-EFFECTS.md` — status catalog, tags, ImmunitySet/OverrideSet
  - [ ] Resolve O-02: does the five-tag taxonomy carve the space correctly?
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
