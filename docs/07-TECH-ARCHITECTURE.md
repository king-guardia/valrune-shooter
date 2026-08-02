> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 07 — Technical Architecture

## 7.1 Engine decision: Godot 4.7

**Recommendation: Godot 4.7 (stable, released June 2026), GDScript primary.**

### Why, specifically for this project

| Factor | Godot 4.7 | Unity 6 |
|---|---|---|
| **Cursor / LLM workflow** | Scripts are plain `.gd`; scenes are human-readable `.tscn` text. An AI assistant can read and edit *everything* | C# is fine, but scenes/prefabs are dense YAML that assistants edit badly. Much of your project state is invisible to the model |
| **Git diffs** | Clean, mergeable | Prefab/scene merges are a known pain |
| **2D** | First-class, purpose-built | Good, but 2D is a layer on a 3D engine |
| **APK size** | ~25–40 MB for a game this size | ~60–90 MB typical |
| **Cost** | MIT licence, $0, no revenue share, no seat fees | Free tier exists; licensing terms have changed before |
| **Play Billing** | First-party `GodotGooglePlayBilling` plugin, maintained, GDScript API | First-party Unity IAP, more mature |
| **Mobile input** | `VirtualJoystick` node shipped production-ready in 4.7 | Input System package |
| **Ecosystem / tutorials** | Smaller | Much larger |
| **Hiring help later** | Harder | Easier |

The decisive factors are the first two. You are building this *through* an AI assistant, and Godot's fully-text project format means the assistant can actually see your game. That advantage compounds over 12 months.

**Choose Unity instead if:** you expect to hire contract help, want a large asset store to substitute for art you can't make, or plan a console port. Both are correct choices; Godot is the better fit for *your stated workflow*.

**Do not choose:** Unreal (wrong tool for 2D mobile), or a custom/web framework (you'd spend six months building an engine).

**Language:** GDScript for everything. Drop to C# or GDExtension **only** if profiling proves a hotspot — realistically only bullet collision at 400+ projectiles, and that's solvable with the approach in §7.5 first.

## 7.2 Repository layout

```
valrune/
├─ .cursor/
│  └─ rules/                    # see 12-CURSOR-WORKFLOW.md
├─ docs/                        # these documents; the source of truth
│  └─ decisions/                # ADRs: one file per significant decision
├─ game/                        # the Godot project
│  ├─ project.godot
│  ├─ autoload/                 # GameState, EventBus, SaveManager, AudioManager, Wallet
│  ├─ core/
│  │  ├─ entities/              # Ship, Enemy, Projectile, Pickup base classes
│  │  ├─ components/            # WrapAround2D, Health, Hitbox, Emitter, StatusStack
│  │  ├─ systems/               # WaveRunner, DamageResolver, UpgradeResolver, SpawnPool
│  │  └─ input/                 # HeadingZone, MovementZone, ProxyPad, FloatingStick
│  ├─ modes/
│  │  ├─ throat/                # ThroatStage scene + wrap logic
│  │  └─ field/                 # FieldStage scene + boundary + minimap
│  ├─ ui/                       # one folder per screen from 06
│  ├─ data/                     # ★ ALL BALANCE AND CONTENT — see 08
│  │  ├─ upgrades.json
│  │  ├─ elements.json
│  │  ├─ nodes.json
│  │  ├─ clauses.json
│  │  ├─ enemies.json
│  │  ├─ waves/                 # one file per contract
│  │  ├─ contracts.json
│  │  └─ sectors.json
│  ├─ assets/
│  │  ├─ proto/                 # placeholder only; CI fails if a scene references this
│  │  └─ ship/                  # art/ audio/ fonts/ shaders/
│  └─ locale/
├─ sim/                         # headless balance simulator (see 04 §4.13)
├─ art-source/                  # .ai and .svg sources, OUTSIDE the Godot project
├─ CREDITS.md                   # every asset: source, author, licence, URL, date
├─ tools/                       # asset pipeline scripts, data validators
└─ tests/                       # GUT unit tests
```

**The single most important architectural rule:** `game/data/` contains every number that affects balance. Code contains zero magic numbers. This is what makes the project tractable in Cursor — you tune by editing JSON, and the assistant can regenerate content without touching logic.

## 7.3 Core systems

### 7.3.1 EventBus (autoload signal hub)
Entities never reference each other directly. `EventBus.enemy_killed.emit(enemy, killer_source)`. Everything else — score, credits, XP, element procs, analytics, achievements — subscribes. This keeps element abilities from becoming a spaghetti of cross-references, which is the failure mode that kills build-crafting games.

### 7.3.2 UpgradeResolver
A single pure function: `resolve(save_data) -> ShipStats`. Takes purchased stat ranks, element tiers, and the equipped loadout (1 active + 3 passives), returns a flat immutable stats struct plus the active node reference. There is no pilot-level input — that axis was removed in r3. Called on Hangar changes and contract start, never per-frame.

Node tier follows the Element TD model: `min(levels)` for singles and duals, `min(levels) − 1` for triples (max tier 2). Values scale 1.0 / 1.5 / 2.1 for singles and duals, 1.0 / 1.7 for triples, and tier also gates **which trigger layers exist** — see `04` §4.8. **Composed nodes** (singles and all triples) derive their effects from a shared framework or from their constituent duals rather than carrying their own. Gun attunements are resolved separately (`04` §4.4): each slot contributes its own damage share, on-hit effect, and element for matchup purposes. **Unequipped nodes and null slots contribute nothing.**

**Order of operations (fix this now, never change it):**
```
base → additive flat bonuses → additive percentage bonuses → multiplicative category modifiers → caps
```
Write it down in an ADR. Every balance bug in games like this traces to inconsistent operation order.

### 7.3.3 StatusStack
Enemies carry a component holding statuses (Burn, Chill, Exposed…). Each status: id, stacks, remaining duration, source. Combos query it (`SCALD` checks for Burn+Chill). One system, extensible without touching enemy code.

### 7.3.4 WaveRunner
Reads a contract's wave JSON and emits spawn commands on a **fixed timeline** (not frame-dependent). Supports: `at_time`, `on_all_dead`, `on_count_below`. Fully deterministic per `02` §2.1.

Equipped clauses are applied as a **modifier layer over the parsed wave data** at load time — HP multipliers, speed multipliers, count multipliers, elite flags, pickup suppression. Clauses never alter timeline structure, only parameters, which is what keeps them cheap to build and impossible to break determinism with.

### 7.3.7 CritResolver
Owned by the ship. Rolls **once per volley fired** (not per projectile), only while a valid target is on screen. Uses PRD: `P(n) = C × n` where `n` is shots since the last crit, `C` from the shipped lookup table. Resets `n` on a crit. Nominal chance caps at 60% from all sources.

A separate counter tracks **shots fired**; every 150 emits a Surge event, with a telegraph fired 3 shots ahead. **Every equipped node subscribes to the same crit and Surge events and fires simultaneously** — no priority arbitration.

On a crit, `CritResolver` also drives **cross-propagation**: each gun's on-hit effect fires at the crit amplifier, and the set of targets touched by any gun's effect becomes the target set for every other gun's effect.

### 7.3.8 ElementMatchup
Pure function: `multiplier(attacker_element, defender_elements) -> float`. Reads `type_chart.json`, multiplies pairwise results, applies the flat 0.90× for any cross-circle pair, clamps to **[0.55, 1.85]**. Called **per gun**, since each gun carries its own attunement — a hedged 3-gun config produces a blended effective multiplier. Null slots and unattuned damage resolve at 1.00×.

### 7.3.5 Object pooling
Projectiles, enemies, particles, damage numbers — all pooled, pre-warmed at level load. Never `instantiate()` during combat. This is the difference between 60fps and 30fps on a mid-range phone.

### 7.3.6 Damage pipeline
One entry point: `DamageResolver.apply(source, target, damage_packet)`. The packet carries amount, element, is_overcharge, can_crit, statuses_to_apply. Every damage source in the game goes through it. Log every call in debug builds — this is your balance telemetry.

## 7.4 Determinism

Because §2.1 requires deterministic gameplay:
- **Fixed timestep for gameplay logic** (`_physics_process` at 60Hz). Never use `_process` delta for anything affecting outcomes.
- Any randomness (purely visual) uses a separate `RandomNumberGenerator` instance seeded per-level and **never** touches gameplay state.
- Enemy AI must be a function of `(time, player_position, own_state)` only.
- Bonus: this gives you replay recording almost free (store the input stream). Worth doing for the killcam and for bug reports.

## 7.5 Performance targets

| Metric | Target | Floor |
|---|---|---|
| Frame rate | 60 fps | 45 fps sustained |
| Reference device | Snapdragon 6-series / 4GB RAM, ~2022 midrange | |
| Frame time budget | 16.6ms: ≤6ms gameplay, ≤7ms render, ≤3ms UI | |
| Draw calls | <120 | 200 |
| APK/AAB size | <60 MB | 100 MB |
| Cold start to title | <2.5s | 4s |
| Battery | <9%/hour | 15%/hour |

**Bullet performance approach, in order:** (1) pool everything; (2) batch bullets into a single `MultiMeshInstance2D` rather than individual nodes; (3) do collision as manual spatial-hash distance checks rather than physics bodies; (4) only then consider C#/GDExtension. Steps 1–3 will get you to 400 bullets comfortably.

**Test on a real cheap device from week one.** The emulator and your desktop will lie to you.

## 7.6 Save data

**Local-first. The game is fully playable offline, forever.**

- Format: JSON, versioned, written to `user://save_v{n}.json`.
- **Migration function per version.** Write `migrate_v1_to_v2()` before you need it; a save-breaking update after launch is a review-score catastrophe.
- Integrity: HMAC-SHA256 of the payload using a key derived from a device-stable identifier + an app secret. This deters casual editing; it does not stop a determined attacker and is not meant to (`09` §9.5).
- Atomic writes: write to temp, fsync, rename. Phones get killed mid-write constantly.
- Autosave on: level complete, purchase, app pause (`NOTIFICATION_APPLICATION_PAUSED`). Never mid-combat.

**Save schema (v1):**
```json
{
  "version": 1,
  "profile": { "created_at": "...", "playtime_s": 31200 },
  "wallet": { "credits": 1340 },
  "progress": {
    "stages_completed": [1,2,3],
    "best_times": {"3": 142.6},
    "challenges_earned": {"3": ["no_life_lost"]},
    "difficulty": "standard"
  },
  "build": {
    "stat_ranks": { "cadence": 5, "hull": 2 },
    "element_tiers": { "PLASMA": 2, "CRYO": 2, "FORGE": 1, "VOLT": 0 },
    "points_unspent": 1,
    "attunement_slots": ["PLASMA", "PLASMA", "CRYO"],
    "loadout": { "active": "SQUALL", "passives": ["PLASMA_NODE", "CRYO_NODE", "SCALD"] },
    "runes_equipped": ["dense", "swift"]
  },
  "settings": { "handedness": "right", "heading_gain": 1.0, "flash_reduction": false },
  "unlocks": { "bestiary": [1,2,3], "logs": [1,2] },
  "entitlements_cache": { "supporter": true, "pack_chrono": false, "verified_at": "...", "signature": "..." },
  "hmac": "..."
}
```

Entitlements are **cached locally with a server signature** and re-verified when online — see `09` §9.3.

## 7.7 Build and CI

- **Git from commit zero.** GitHub. `main` protected, work on branches.
- **Git LFS for binary art** (PNG, audio). Set this up before you add art, not after.
- **GitHub Actions:** on PR → run GUT unit tests + data-schema validation + the balance sim. On tag → export AAB, upload to Play internal testing track via `r0adkll/upload-google-play` or `fastlane supply`.
- **Data validation in CI is high-value:** a script that loads every JSON in `data/`, validates against the schemas in `08`, and fails on unknown enemy ids, missing wave references, or out-of-range values. Cursor will generate content; this catches its mistakes automatically.
- **Semantic versioning**, `versionCode` auto-incremented from the run number.
- Keep the Android keystore in a password manager **and** one offline backup. Losing it means you can never update the app.

## 7.8 Testing

| Layer | Tool | What |
|---|---|---|
| Unit | GUT (Godot Unit Test) | UpgradeResolver math (tier curve, min() rule, damage resolution order), PRD convergence over 100k trials, ElementMatchup clamping, save migration, wave parsing, clause application, economy formulas |
| Data | Custom validator | Every JSON file against schema |
| Simulation | `sim/` headless | Every build × attunement config × every contract (`04` §4.13) |
| Integration | Godot headless + scripted inputs | Complete a level end-to-end |
| Device | Manual + Firebase Test Lab | 4–6 real device profiles |

**Minimum meaningful coverage: the UpgradeResolver and the economy.** Those two are where silent balance bugs hide.

Two additional CI assertions that are cheap and permanent:
- **Monetization pledge tests** (`09` §9.4) — three static-analysis checks.
- **Pay-to-win test** (`01` §1.7) — strongest base-element build within 15% of strongest DLC build, via the sim.
