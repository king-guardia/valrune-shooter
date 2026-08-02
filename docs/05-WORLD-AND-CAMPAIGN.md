> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 05 — Sector, Story & Campaign

## 5.1 Story brief

The story makes the mechanics feel inevitable and gives the player a reason to reach sector 5. It is not a novel.

- **Static panels with soft parallax animation** — the Diablo 3 approach: 3–4 layers per panel, slow pan, one animated element (drifting particles, a flickering light, a ship easing across). An `AnimationPlayer` on layer offsets. ~1 hour per panel once the template exists.
- Skippable always. Never longer than 20 seconds.
- Every mechanic introduction gets a beat. Mechanics first, fiction second.
- No named characters requiring voice acting or animation beyond a portrait.
- Must survive being ignored entirely.

## 5.2 Setting

**The Lattice** is a network of wormholes threaded through the galaxy by something no longer around. Humanity found an entrance forty years ago and has been mapping it since — cautiously, then greedily.

The Lattice isn't empty. Something lives in the walls between throats: **the Hollow** — a swarm with no ships, no pilots, and no weapons. Just bodies, moving fast, in one direction: at you. Mapping crews called them moths.

Then the Lattice started closing. Throats collapse behind survey ships. Junctions stable for forty years fold in a week. The Hollow are coming out in numbers nobody has seen, and they are getting **better**.

You fly the *Valrune* — a prototype built around a **Resonance Core** salvaged from a Lattice junction. Nobody knows what it is. What it demonstrably does is *attune*: expose it to the raw stuff of a wormhole and it channels something fundamental. Fire. Cold. Mass. Charge.

## 5.3 Mechanics, justified

| Mechanic | Fiction |
|---|---|
| Auto-fire | The cannon is fed directly by the Core. It doesn't stop; it can't |
| Rotation decoupled from movement | Throats have no fixed "down." The gimbal is independent of thrust |
| Horizontal wrap in Throat mode | A throat is a tube. You just flew around it |
| Element points from minibosses and bosses | Salvage rights. The company strips something off the wreck and your rig learns to channel it |
| No element system until Miniboss 1 | The rig is inert until it has been exposed to something. The first two contracts are you flying an ordinary ship |
| Element tiers | The Core deepens an attunement with repeated exposure |
| 1 active + 3 passives | The Core can hold four resonance patterns at once. Which four is your call |
| Critical hits | The Core sheds accumulated potential unevenly. The longer it holds, the likelier the next discharge carries it |
| Element matchups | The Lattice has grain. Push against it and your shots dull; push with it and they bite |
| Recall | An automated retreat vector burned into the flight computer by a previous pilot who did not come back |
| Enemies learn to shoot in Sector 3 | The Hollow adapt. They watched you kill them for two sectors. Now they have your idea |
| Respec | The Core doesn't remember. Neither should you |

## 5.4 Campaign — 5 sectors × 6 contracts = 30 contracts

Each sector: contracts 1–2 normal, **3 miniboss**, 4–5 normal, **6 boss**. Each of the 10 grants one element point. Ads fire on miniboss and boss defeat.

### Sector 1 — "Moths" (contracts 1–6)

Tutorial by level design; no pop-ups. Mote, Lash, Spike. Contract 4 is the first Field arena.

| Contract | Mode | Introduces |
|---|---|---|
| 1 | Throat | Movement, auto-fire, Mote |
| 2 | Throat | Rotation (side entries), Lash |
| 3 | Throat | **MINIBOSS: SHEARHEAD** — an oversized Spike with a wider telegraph. *Teaches reading a lock-on* |
| 4 | Field | Arena, minimap, boundary, floating stick |
| 5 | Throat | Recall as a tool; wrap-around tactics |
| 6 | Throat | **BOSS: THRESHER** — a rotating blade ring filling the throat; the safe gap moves. *Teaches rotation control* |

### Sector 2 — "Adaptation" (7–12)

Husk. Density. Pickup routing.

| Contract | Mode | Introduces |
|---|---|---|
| 7 | Throat | Husk (splitters) |
| 8 | Field | Optional destructible relays |
| 9 | Field | **MINIBOSS: THE SPLIT** — a Husk that splits three times. *Teaches area damage and kill order* |
| 10 | Throat | Density spike; Discharge value |
| 11 | Field | Multi-wave arena, pickup routing |
| 12 | Throat | **BOSS: CANTOR** — mimics your attacks back on a delay. Killing it is the moment the Hollow learn to shoot. *Teaches telegraph reading* |

### Sector 3 — "The Idea" (13–18)

Enemies shoot. Anchor and Warden.

| Contract | Mode | Introduces |
|---|---|---|
| 13 | Throat | **Anchor** — enemies now shoot |
| 14 | Field | Crossfire, cover debris |
| 15 | Throat | **MINIBOSS: THE BATTERY** — a fixed multi-barrel emplacement. *Teaches cover and angles* |
| 16 | Field | Warden (shield bubbles); kill priority |
| 17 | Throat | Mixed roster, high density |
| 18 | Field | **BOSS: ORRERY** — rotating armature, 6 destructible nodes, wrong order makes it worse. *Teaches sub-part targeting* |

### Sector 4 — "Denial" (19–24)

Seeder and Choir. Terrain and DPS checks.

| Contract | Mode | Introduces |
|---|---|---|
| 19 | Throat | Seeder (mines) |
| 20 | Field | Choir carriers |
| 21 | Field | **MINIBOSS: THE NEST** — a Choir that spawns Spikes. *Teaches source-vs-symptom priority* |
| 22 | Throat | Full roster |
| 23 | Field | Endurance: 6 waves, no pickups after wave 3 |
| 24 | Throat | **BOSS: WEAVER-PRIME** — 3 phases with a hard DPS window only clearable if you save your active. *Teaches ability timing* |

### Sector 5 — "The First Mouth" (25–30)

Collapsing space. Everything at once.

| Contract | Mode | Introduces |
|---|---|---|
| 25 | Throat | Collapsing throat — the playfield shrinks over 2.5 min |
| 26 | Field | Shrinking arena boundary |
| 27 | Field | **MINIBOSS: THE FOLD** — a Warden that shields itself. *Teaches burst windows* |
| 28 | Throat | Gauntlet: every archetype, escalating |
| 29 | Field | Mini-boss rush — Thresher and Cantor variants simultaneously |
| 30 | Both | **BOSS: THE FIRST MOUTH** — 4 phases alternating Throat and Field within one fight. Unlocks New Game+ |

**New Game+** ships with the first DLC pack rather than the base game: replay with your build, enemies +40% hull and speed.

## 5.5 Story delivery

**Story panels appear only after minibosses and bosses** — ten moments across the campaign, at the two points per sector where the player has just earned something and is receptive.

One parallax panel each, 2–4 sentences typed on with a terminal effect, skip on tap, auto-advance at 8s, re-readable in the Encyclopedia.

**Total writing: 10 entries × 3 sentences ≈ 400 words.** Panels between every contract would be noise; this cadence makes each one an event.

## 5.6 The Encyclopedia

Always available, including mid-run. Three tabs. **Load-bearing for revenue** (`01` §1.7) as well as for anti-gambling — purchase intent must form during play, not at the end.

| Tab | Contents |
|---|---|
| **Bestiary** | Each enemy: silhouette, behaviour, threat class, first encountered, kill count. Unlocks on first kill |
| **Lattice** | Log entries in order, re-readable |
| **Core** | **All seven elements and every node — including unowned DLC ones** — with full mechanical descriptions, tier tables, and unlock requirements. Locked entries greyed, never hidden |

Hiding the node list makes players feel gambled-at, which violates `02`. Show everything. Let them plan — and let them want.

## 5.7 Side sectors (DLC)

Each element pack adds a **6-contract side sector** off the main map, matching the base campaign's sector length, themed around its element, with a short log thread. Harder than campaign contracts since they're optional. The natural showcase: a Chrono side sector built around slow-fields teaches you why you bought it.

**Fully clause-compatible**, so they're replayable content rather than one-and-done.

Side sectors appear on the campaign map from early on, marked as expansions.

## 5.8 Naming register

Two registers, kept clean:

- **Human / technical:** Valrune, Resonance Core, Throat, Junction, Ablative, Slipstream. Blunt, functional, faintly military.
- **Hollow / Lattice:** Thresher, Cantor, Orrery, Weaver-Prime, The First Mouth, The Fold. Singular nouns, faintly liturgical, never explained.

Elements are short and hard (PLASMA, CRYO, FORGE, VOLT). Nodes use weather and geology (SCALD, SLAG, RIME, SQUALL, CALDERA, MONSOON, PERMAFROST) except LODESTONE and CRUCIBLE, which are tools — that's the FORGE influence showing through.

**Trademark-search your final title at M1, not M5.**
