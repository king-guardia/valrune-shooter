> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 12 — Working With Cursor

You're building this through an AI coding assistant. That changes what "a good plan" looks like: the plan is the assistant's memory, and the repository structure is the assistant's guardrail. This document is about making that work over 12+ months.

## 12.1 The central problem

An AI assistant is excellent at **bounded tasks with clear contracts** and dangerous at **unbounded tasks with implicit context**. It will confidently invent a numbers system, silently change your damage formula, add a `localStorage` call that can't work, or "helpfully" add a currency purchase flow.

The countermeasures are: (a) put every contract in a file, (b) keep tasks small, (c) make violations fail in CI rather than in your head.

## 12.2 Repo setup

Copy `docs/` into the repo root. Then create `.cursor/rules/` with these files. Cursor reads them automatically as project rules.

### `.cursor/rules/00-project.mdc`
```
---
alwaysApply: true
---
This is VALRUNE, an Android 2D space shooter in Godot 4.7 / GDScript.

Authoritative specifications live in /docs. Before implementing anything,
read the relevant doc. If a request conflicts with /docs, say so and ask;
do not silently deviate.

Key files:
- docs/02-DESIGN-PRINCIPLES.md  — allowed vs not allowed. Binding.
- docs/04-PROGRESSION-AND-ELEMENTS.md — all balance numbers
- docs/07-TECH-ARCHITECTURE.md — structure and conventions
- docs/08-DATA-SCHEMAS.md — the content contract
```

### `.cursor/rules/10-hard-constraints.mdc`
```
---
alwaysApply: true
---
NEVER:
- Introduce variance that decides an outcome: random drops, "pick 1 of 3"
  upgrades, wave composition or enemy order varying between plays. Visual-only
  randomness is fine and must use a separate RNG instance. See docs/02 2.1.
- Generate contract content at runtime. Offline generators that bake to JSON are fine.
- Give enemies critical hits.
- Add any path from a purchase to in-game credits.
- Add energy systems, timers that gate play, loot boxes, daily rewards, or
  rewarded ads. Interstitials exist at exactly ONE state-machine node
  (miniboss/boss defeat) - do not add a second.
- Allow more than 1 equipped active or 3 equipped passives.
- Write an effect that does not resolve to an entry in primitives.json. That is a
  code task requiring an ADR in docs/decisions/, never a data edit.
- Let one status override, replace, or suppress another. All statuses stack.
- Hand-author a triple node. Triples are composed from their constituent duals.
- Add an upgrade or node that increases credit gain.
- Make Surge crit-anchored. It fires every 150 shots fired, so it can be telegraphed.
- Couple the active ability to crit or Surge. It runs on its own cooldown.
- Use geometric cost growth. Costs are arithmetic: base + step * (rank - 1).
- Put balance numbers in code. All numbers live in game/data/*.json.
- Instantiate nodes during combat. Use the object pools.
- Use _process() for anything affecting gameplay outcomes. Use _physics_process().
- Add a new enemy behaviour type without an ADR in docs/decisions/.
- Charge a different credit amount by difficulty level.

ALWAYS:
- Route damage through DamageResolver.apply().
- Emit gameplay events on EventBus rather than calling across entities.
- Use percentage-of-playfield coordinates in wave data, never pixels.
- Reference user-facing strings by key from locale/, never hardcode.
- Respect the caps in docs/02-DESIGN-PRINCIPLES.md §2.7.
```

### `.cursor/rules/20-gdscript-style.mdc`
```
---
globs: ["**/*.gd"]
---
- snake_case for functions/variables, PascalCase for classes, SCREAMING_CASE for constants.
- Static typing on every function signature and member variable.
- class_name on every reusable class.
- One class per file. Files under 300 lines; split if longer.
- No node paths as strings beyond @onready; use exported NodePaths or unique names.
- Document any function whose behaviour isn't obvious from its name in one line.
- Prefer composition (child component nodes) over inheritance beyond 2 levels.
```

### `.cursor/rules/30-data.mdc`
```
---
globs: ["game/data/**/*.json"]
---
Conform exactly to the schemas in docs/08-DATA-SCHEMAS.md.
Only use stat ids from the canonical list in §8.1.
Only use behaviour types from the closed list in §8.4.
Coordinates are percentages (0.0-1.0), never pixels.
After editing, mentally run the validator: unknown ids, missing references,
and out-of-range values are errors.
```

## 12.3 Task granularity — the thing that matters most

**Bad task:** "Build the upgrade system."
**Good task:** "Implement `UpgradeResolver.resolve(save_data) -> ShipStats` per docs/07 §7.3.2. Read upgrades.json. Apply operations in the documented order. Add GUT tests covering: empty save, single rank, capped rank, and operation ordering with mixed add/add_pct/mult."

Rules of thumb:
- A task should touch **1–3 files**.
- A task should be verifiable by a test or by looking at one screen.
- If you can't describe the acceptance criteria in two sentences, it's a milestone, not a task.
- **Give it the doc section reference.** `per docs/04 §4.3` is worth a paragraph of explanation.

Keep a `TASKS.md` at the repo root with the current milestone's tasks as checkboxes. Point Cursor at it. Cross things off — for your own morale as much as anything.

## 12.4 The content loop (your highest-leverage workflow)

For all 30 contracts:

```
You:    "Write game/data/waves/stage_14.json. Sector 3, Field mode, ~2:30.
         Teaches crossfire: two Anchor nests at opposite arena edges plus
         continuous rammer pressure so the player can't sit still.
         Roster: anchor, spike, lash. kind: normal. Per docs/08 §8.5.
         Slightly above contract 13."
Cursor: [writes the file]
CI:     [validates schema, runs balance sim, reports estimated clear time]
You:    [play it, adjust two numbers]
```

Write this prompt template into `docs/prompts/contract-authoring.md` so it's identical every time. Consistency across 30 contracts is what makes the campaign feel authored rather than assembled.

Same pattern for node tier values, enemy definitions, clauses, and log entries.

## 12.5 Guardrails against the specific failure modes

| Failure mode | Guardrail |
|---|---|
| Invents balance numbers | All numbers in JSON + CI validation with range checks |
| Silently changes damage formula | GUT tests on `DamageResolver` and `UpgradeResolver`; they fail loudly |
| Adds a monetization dark pattern | The three static-analysis tests in `09` §9.4, run in CI |
| Makes DLC elements stronger than base | The automated pay-to-win gate (`01` §1.7) |
| Invents an effect outside the primitive vocabulary | CI validates every `primitives` reference against `primitives.json` |
| Hand-authors triples instead of composing them | `authoring` field is required and CI checks triples are `composed` |
| Reintroduces geometric cost growth | Schema has no `cost_growth` field; validator rejects it |
| Drifts from the design | `docs/` in context + the rules files; re-read the relevant doc at the start of each session |
| Writes 800-line files | Style rule + review; split aggressively |
| Loses context across sessions | ADRs in `docs/decisions/` — one file per decision, ~10 lines each. This is the assistant's long-term memory |
| Over-engineers | State the scope explicitly in the task: "no abstraction beyond what this needs" |

## 12.6 Architecture Decision Records

Every non-obvious decision gets a file in `docs/decisions/NNNN-title.md`:

```markdown
# 0007 — Stat operation order is add → add_pct → mult → cap

Date: 2026-08-14
Status: Accepted

## Context
Upgrades, elements, and combos all modify the same stats. Without a fixed
order, the same purchases produce different results depending on evaluation
sequence, and balance becomes untestable.

## Decision
UpgradeResolver applies, per stat: flat additions, then additive percentages
summed and applied once, then multiplicative category modifiers, then caps.

## Consequences
Multiplicative sources are strictly stronger, so they are reserved for
triple-combo actives. Any new modifier type requires updating this ADR.
```

Ten of these will save you more time than any other documentation you write, because the assistant re-reads them and stays consistent, and because in month nine you will not remember why.

## 12.7 Session hygiene

- **Start each session by telling Cursor what milestone you're in and which doc is relevant.** Context resets; the docs don't.
- **Commit before every AI-driven change of any size.** `git diff` is your review tool and your undo.
- **Review generated GDScript for**: unpooled instantiation, `_process` used for gameplay, hardcoded numbers, missing static types, direct cross-entity references bypassing EventBus.
- **Run the game after every task.** A build that compiles is not a feature that works.
- **When it's confidently wrong twice, stop and write a doc section instead.** The problem is missing context, not a bad model. Fixing the doc fixes the class of error permanently.

## 12.8 What Cursor should NOT be doing

- Making design decisions. It will happily invent a crit formula, add a currency, or "improve" the element system back into sockets. That's your job, and `02`/`04` already decided.
- Art direction.
- Balance judgement (it can run the sim; you interpret it).
- Deciding what to cut. Scope discipline can't be delegated.
- Writing the Play Store listing or privacy policy without your review — these have legal consequences.
