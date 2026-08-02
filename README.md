# VALRUNE

A neon top-down space shooter for Android. You fly the *Valrune* for a mercenary company
through a collapsing wormhole network, building your ship from combinable elemental
attunements. The full campaign is free — no gacha, no currency store, no timers.

**Status: planning.** No game code exists yet and none should be written until the M0
control prototype. The current work is reconciling the design, structuring the ability
data, and building the tools that make balancing several hundred abilities tractable.

## Start here

| If you want to | Read |
|---|---|
| Know what's been decided | [`docs/decisions/0000-decision-log.md`](docs/decisions/0000-decision-log.md) — **authoritative** |
| Understand the game | [`docs/00-START-HERE.md`](docs/00-START-HERE.md) |
| See what's next | [`TASKS.md`](TASKS.md) |

> The numbered documents in `docs/` are **v0**, written before the ability and status
> spreadsheets were reconciled into the plan. Several of their decisions have been
> reversed. Every one carries a banner saying so. **Where a v0 document conflicts with
> the decision log, the decision log wins.**

## Layout

```
.cursor/rules/     Project rules. Read automatically by the assistant.
.cursor/skills/    coverage-auditor, balance-calibrator          (Phase 5)
docs/              Design documents 00–17
docs/decisions/    Decision log and ADRs — the authoritative record
docs/uml/          Model diagrams                                (Phase 3)
docs/prompts/      Reusable authoring prompts
data/source/       The original spreadsheet brain dump. Input, not schema.
data/              Validated JSON. Source of truth for all balance numbers.
data/schema/       JSON schemas, enforced in CI
tools/             Coverage report, balance engine, validators, Sheets sync
balance/           balance-lab.html — interactive tuning sandbox   (Phase 4)
dashboard/         index.html — progress and coverage dashboard    (Phase 6)
game/              Godot 4.7.1 project. Empty until M0.
```

## Ground rules

Two that explain most of the structure:

**Every number that affects balance lives in `data/`, never in code.** That is what makes
the project tractable through an AI assistant — tuning is a JSON edit, and content can be
generated without touching logic.

**Sparse ability coverage is intended.** Targets are 50% of dual slots and 30% of triple
slots, with every base element appearing in 1–2 triples. A combination with no ability is
a valid state. Gaps get authored deliberately, never filled to hit a number.

## Related

| Thing | Where |
|---|---|
| GCP project | `valrune-shooter` — backend work starts at M5 |
| Engine | Godot 4.7.1, GDScript |
