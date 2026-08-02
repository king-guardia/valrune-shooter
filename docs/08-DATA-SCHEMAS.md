> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 08 — Data Schemas (the content pipeline)

Everything affecting balance or content lives here as JSON. Code reads these; code contains no numbers. **Cursor authors content by writing data, not logic.** Validate all of it in CI.

---

## 8.1 `upgrades.json`

```json
{
  "schema_version": 2,
  "upgrades": [
    {
      "id": "cadence",
      "display_name": "Cadence",
      "category": "weapons",
      "description": "Increases rate of fire.",
      "max_rank": 24,
      "cost_base": 10,
      "cost_step": 12,
      "effects": [
        { "stat": "fire_rate", "op": "add", "value": 0.25 }
      ]
    }
  ]
}
```

**Cost is arithmetic, never geometric:** `cost(n) = cost_base + cost_step × (n − 1)`. A `cost_growth` field does not exist and must not be added (`04` §4.2).

`op` ∈ `add` · `add_pct` · `mult` · `set` · `set_table`

Some lines use a per-rank value table rather than a single op, e.g. Damage:

```json
{ "id": "damage", "max_rank": 7, "cost_base": 10, "cost_step": 100,
  "effects": [{ "stat": "damage_range", "op": "set_table",
    "table": [[3,5],[5,7],[7,9],[9,11],[11,14],[14,17],[17,21],[19,24]] }] }
```

Hull uses a **tiered cost band** rather than a linear step:

```json
{ "id": "hull", "unit": 10, "cost_bands": [
    { "up_to": 100, "cost_per_unit": 10 },
    { "up_to": 200, "cost_per_unit": 30 },
    { "up_to": 300, "cost_per_unit": 50 },
    { "up_to": 500, "cost_per_unit": 80 } ] }
```

**Canonical stat ids** — the only legal values for `stat`:
```
fire_rate, gun_count, damage_range, projectile_speed,
projectile_size, spread_degrees, pierce_count,
crit_chance_pct, crit_damage_mult,
hull_max, shield_charges, shield_recharge_time,
bulwark_flat, bulwark_pct,
move_speed, rotation_rate,
pickup_magnet_radius
```

**Deliberately absent:** `overcharge_threshold`, `bounce_count`, `xp_gain_pct`, `recall_cooldown`, and anything pilot-level related.

`crit_chance_pct` **starts at 5, not 0** — nodes trigger on crit, so a zero baseline would leave a fresh player with dead nodes. Purchasable to 30; hard cap 60 from all sources.

`surge_interval` is a **fixed constant of 150 shots fired**, not a stat. It is not in the schema.

`credit_gain_pct` and `xp_gain_pct` are **absent by design** — no upgrade or node increases credit gain.

**`crit_chance_pct` resolves through the PRD lookup table**, not a raw roll:
```json
{ "prd_constants": { "1": 0.00014, "5": 0.00380, "10": 0.01475,
                     "15": 0.03222, "20": 0.05570, "25": 0.08474 } }
```
Ship the whole 1–60 table; interpolating at runtime is wrong.

## 8.2 `elements.json`

```json
{
  "schema_version": 2,
  "tier_scaling": [1.0, 2.4, 5.0],
  "max_tier": 3,
  "elements": [
    {
      "id": "PLASMA",
      "display_name": "Plasma",
      "color_primary": "#FF6A1F",
      "color_secondary": "#FFD24A",
      "dlc_pack": null,
      "bias": "Damage over time. Strong against clusters, weak burst."
    }
  ]
}
```

Elements no longer carry tiers or abilities — those belong to nodes. An element is an identity and a colour; the tier is player state.

## 8.3 `nodes.json`

Nodes own the **crit** and **Surge** layers. The on-hit layer belongs to gun attunements (§8.3b), so nodes never define one.

**Every effect must resolve to an entry in `primitives.json` plus parameters.** If it cannot, it is a code task requiring an ADR — never a data edit. Keeping the primitive vocabulary closed is what makes 129 nodes tractable and patchable.

```json
{
  "schema_version": 4,
  "tier_scaling": { "single": [1.0, 1.5, 2.1], "dual": [1.0, 1.5, 2.1], "triple": [1.0, 1.7] },
  "max_tier": { "single": 3, "dual": 3, "triple": 2 },
  "nodes": [
    {
      "id": "SCALD",
      "elements": ["PLASMA", "CRYO"],
      "kind": "dual",
      "authoring": "bespoke",
      "slot_type": "passive",
      "display_name": "Scald",
      "flavour": "Coolant flashes to steam the instant plasma touches it. The crew calls it a scald.",
      "triggers": {
        "on_crit": {
          "min_tier": 1,
          "primitives": ["spread_status"],
          "params": { "status": "burn", "radius": [90, 130, 170] }
        },
        "on_surge": {
          "min_tier": 3,
          "primitives": ["field_persistent", "scatter"],
          "params": { "radius": 320, "duration": 4.0, "effect": "break_enemy_targeting" }
        }
      },
      "vfx": { "on_crit": "res://art/vfx/scald_crit.tscn", "on_surge": "res://art/vfx/scald_surge.tscn" },
      "sfx": { "on_crit": "sfx_scald_crit", "on_surge": "sfx_scald_surge" }
    },
    {
      "id": "PLASMA_CRYO_FORGE",
      "elements": ["PLASMA", "CRYO", "FORGE"],
      "kind": "triple",
      "authoring": "composed",
      "composed_from": ["SCALD", "SLAG", "RIME"],
      "amplifier": 1.4,
      "signature": {
        "display_name": "Caldera",
        "flavour": "Molten vapour that clings and keeps burning.",
        "vfx_overlay": "res://art/vfx/caldera_overlay.tscn",
        "sfx": "sfx_caldera"
      }
    }
  ]
}
```

**Field rules**
- `authoring` ∈ `bespoke` (duals) | `composed` (singles and triples).
- **Composed nodes** carry `composed_from` and `amplifier` instead of `triggers`. They fire all three constituent duals' effects simultaneously at the amplifier, plus their signature overlay. No independent effect authoring.
- `min_tier` gates which trigger layers exist. Node tier = `min(constituent element levels)` for singles and duals; `min(all three) − 1` for triples.
- `params` arrays are indexed by node tier — length 3 for singles and duals, length 2 for triples.
- `slot_type` ∈ `active` | `passive`. Actives have no `triggers` and are **never coupled to crit or Surge**.
- **No node may reference the `resource_gain` primitive.** No node grants credits.
- **Unique `vfx` and `sfx` per node per trigger.** CI warns on any duplicate.

**Completeness validation:** CI computes every unordered combination of shipped elements (9 singles + 36 duals + 84 triples = 129) and fails on gaps or duplicates.

**Authoring budget:** 36 bespoke duals; singles and triples composed.

## 8.3b `attunements.json`

Gun slots own the on-hit layer. Nine entries, composed from one framework and math-scaled by element level.

```json
{
  "schema_version": 2,
  "gun_configs": {
    "1": { "damage_split": [1.0],           "spread_mode": "none" },
    "2": { "damage_split": [0.8, 0.8],      "spread_mode": "helix_amplitude" },
    "3": { "damage_split": [1.1, 0.5, 0.5], "spread_mode": "fan_angle" }
  },
  "level_potency": [1.0, 1.5, 2.0],
  "crit_amplifier": 2.0,
  "crit_cross_propagation": true,
  "attunements": [
    { "element": "PLASMA",  "on_hit": "damage_over_time",
      "params": { "status": "burn", "dps": [8, 12, 16], "duration": 3.0, "max_stacks": 3 },
      "projectile_tint": "#FF6A1F" },
    { "element": "CAUSTIC", "on_hit": "vulnerability",
      "params": { "pct_per_stack": [2, 3, 4], "max_stacks": 8, "duration": 5.0 },
      "projectile_tint": "#A8E63A" },
    { "element": "AETHER",  "on_hit": "pierce",
      "params": { "bypass_pct": [15, 25, 35], "bypasses": ["shield", "bulwark_flat"] },
      "projectile_tint": "#D9F2FF" }
  ]
}
```

- `damage_split` index 0 is the **centre** gun. The 3-gun split is asymmetric: 52% / 24% / 24%.
- **Each gun applies its own on-hit effect independently.** Duplicates apply it multiple times per volley. There is no special stacking rule.
- A slot may be `null`: damage with no element, no effect, neutral matchup.
- Each gun's damage carries **its own** element for type-chart purposes, so hedged configs earn partial matchup credit.
- **`crit_cross_propagation`**: on a crit, each gun's effect fires at `crit_amplifier` magnitude and cross-applies to every target touched by any other gun's effect.

Player state: `attunement_slots: ["PLASMA", "PLASMA", "CRYO"]`, index 0 centre. Re-slotting is free and instant.

## 8.3c `primitives.json`

The closed mechanic vocabulary. 50 entries across Damage, Delivery, Control, Defence, Modifier, and Utility categories. Every node and attunement effect must resolve to entries here.

```json
{ "id": "spread_status", "category": "Delivery",
  "description": "Copies existing statuses to nearby targets",
  "params": ["radius", "statuses", "max_targets"] }
```

**CI check:** every `primitives` array in `nodes.json` and `attunements.json` references only ids present here. An unknown primitive fails the build.

Adding a primitive requires an ADR, and is usually the right move when several planned abilities all need the same missing mechanic — one well-designed primitive unlocks a dozen abilities more cheaply than forty special cases.

## 8.3d `statuses.json`

```json
{
  "schema_version": 1,
  "visual_stack_cap": 2,
  "statuses": [
    { "id": "burn", "source": "PLASMA", "max_stacks": 3, "reapply": "refresh",
      "rim_light": "#FF6A1F", "particle": "res://art/vfx/status_burn.tscn" },
    { "id": "corrode", "source": "CAUSTIC", "max_stacks": 8, "reapply": "stack",
      "rim_light": "#A8E63A", "particle": "res://art/vfx/status_corrode.tscn" }
  ]
}
```

**All statuses stack independently. No status overrides, replaces, or suppresses another.** `reapply` ∈ `refresh` | `stack` governs same-status behaviour only.

**`visual_stack_cap: 2`** — mechanical stacking is unlimited by this field; only the display is capped. An enemy shows the two most recently applied statuses via blended rim light and particle. Full detail appears on a targeted or player-adjacent enemy only.

## 8.4 `clauses.json`

```json
{
  "schema_version": 1,
  "max_equipped": 4,
  "max_total_bonus_pct": 100,
  "clauses": [
    {
      "id": "dense",
      "display_name": "Dense",
      "description": "Enemies have 25% more hull.",
      "bonus_pct": 15,
      "modifiers": [{ "target": "enemy.hull", "op": "mult", "value": 1.25 }]
    },
    {
      "id": "fragile",
      "display_name": "Fragile",
      "description": "You have one life.",
      "bonus_pct": 25,
      "modifiers": [{ "target": "contract.lives", "op": "set", "value": 1 }]
    }
  ]
}
```

Clauses only ever modify **parameters**, never timeline structure. That's what keeps them cheap and determinism-safe.

Legal `target` values: `enemy.hull` · `enemy.speed` · `enemy.contact_damage` · `enemy.count` · `enemy.elite_waves` · `contract.lives` · `contract.pickups_enabled` · `player.recall_enabled` · `hud.chevrons_enabled`

## 8.5 `waves/stage_XX.json`

```json
{
  "schema_version": 2,
  "contract_id": 7,
  "sector": 2,
  "mode": "throat",
  "kind": "normal",
  "duration_estimate_s": 150,
  "music": "world2_combat_a",
  "waves": [
    {
      "id": "w1",
      "trigger": { "type": "at_time", "value": 0.0 },
      "spawns": [
        { "enemy": "mote", "count": 8, "formation": "line",
          "origin": { "x_pct": 0.5, "y_pct": -0.05 },
          "spacing": 54, "stagger_s": 0.12, "heading_deg": 180 }
      ]
    },
    {
      "id": "w2",
      "trigger": { "type": "on_count_below", "value": 4 },
      "spawns": [
        { "enemy": "husk", "count": 2, "formation": "point",
          "origin": { "x_pct": 0.2, "y_pct": -0.05 } }
      ]
    }
  ],
  "pickups": [
    { "type": "shield_pod", "at_time": 42.0, "position": { "x_pct": 0.5, "y_pct": 0.35 } }
  ],
  "objectives": { "primary": { "type": "clear_all_waves" } }
}
```

- `kind` ∈ `normal` | `miniboss` | `boss`. Miniboss and boss award an element point and are the only ad triggers.
- **Trigger types:** `at_time` · `on_all_dead` · `on_count_below` · `on_previous_wave_complete` · `on_boss_phase`
- Spawn entries may carry `role_tags`, `elements`, and `mage_ability` overrides, layered on top of the enemy definition
- **Formations:** `line` · `arc` · `column` · `wedge` · `ring` · `point` · `grid`
- Coordinates are **percentages of the playfield, never pixels** — this is what makes the game resolution-independent across the Android device zoo.

Challenges are **not** authored per contract; the same three apply everywhere (`04` §4.12) and live in `contracts.json` defaults.

**Element point gating** lives in `contracts.json`: `element_points_usable` is `0` for contracts 1–2, `1` from contract 3 (Miniboss 1), and `"all"` from contract 6 (Boss 1). DLC-granted points obey the same gate.

## 8.6 `enemies.json`

```json
{
  "schema_version": 2,
  "enemies": [
    {
      "id": "spike",
      "display_name": "Spike",
      "threat_class": "rammer",
      "sprite_parts": { "hull": "hull_04", "wings": "wings_02", "engine": "engine_01" },
      "tint": "#FF2D78",
      "hitbox_radius": 18,
      "hull": 40,
      "contact_damage": 22,
      "xp_value": 4,
      "credit_value": 3,
      "behaviour": {
        "type": "charge",
        "params": { "approach_speed": 90, "lock_time": 0.8, "charge_speed": 420,
                    "telegraph_color": "#FF3B6B", "post_charge_recovery": 1.2 }
      },
      "weapon": null,
      "role_tags": [],
      "elements": [],
      "elite_variant": { "hull_mult": 2.5, "speed_mult": 1.2, "credit_mult": 3.0 },
      "death": { "vfx": "burst_small", "sfx": "kill_light", "spawns": [] },
      "first_seen_stage": 3
    }
  ]
}
```

**Behaviour types — the complete legal set.** Adding one requires code and an ADR:
`drift` · `sine` · `charge` · `orbit` · `stationary` · `spawner` · `aura` · `mine_layer` · `mirror` · `boss_script`

Keeping this list closed is scope control. `sprite_parts` references the modular kit (`10` §10.4) so enemy visuals are composed, not drawn per enemy.

**Enemies never define crit** (`02` §2.1) — the schema has no field for it.

### Role tags and element affinity

Both dimensions default to empty, and **empty is the common case** (`04` §4.6).

`role_tags` legal values: `fast` · `tough` · `repairing` · `mage` · `decoy` · `mob`. Stackable. Defined once in `role_tags.json` with per-sector scaling:

```json
{
  "id": "mob",
  "display_name": "Mob",
  "description": "Spawns as a group sharing one hull pool.",
  "world_scaling": { "2": { "count": 2 }, "3": { "count": 3 },
                     "4": { "count": 4 }, "5": { "count": 5 } },
  "modifiers": [{ "target": "hull", "op": "divide_across_group" }]
}
```

`mage` takes an ability id per **placement**, chosen in wave data rather than fixed on the enemy — one tag, enormous authored variety:

```json
{ "enemy": "warden", "role_tags": ["mage"],
  "mage_ability": "ally_haste", "count": 1, "formation": "point" }
```

`elements` is 0–4 entries from the element list. Enemies with affinity **must render their element as a rim light and particle trail** — matchups are read at a glance, never from a menu.

### `type_chart.json`

```json
{
  "schema_version": 3,
  "strong": 1.30, "weak": 0.75, "cross_circle": 0.90,
  "clamp": { "min": 0.55, "max": 1.85 },
  "circles": {
    "base": ["PLASMA", "CRYO", "FORGE", "VOLT", "CAUSTIC"],
    "deep": ["CHRONO", "GAMMA", "VOID", "AETHER"]
  },
  "beats": {
    "PLASMA": ["FORGE"], "FORGE": ["VOLT"], "VOLT": ["CRYO"],
    "CRYO": ["CAUSTIC"], "CAUSTIC": ["PLASMA"],
    "AETHER": ["VOID"], "VOID": ["GAMMA"], "GAMMA": ["CHRONO"], "CHRONO": ["AETHER"]
  }
}
```

Any attacker/defender pair spanning the two circles resolves to a flat **0.90×**, symmetrically. Adding a directional cross-circle relationship is a pay-to-win risk and requires an ADR.

**Every gun's damage carries its own attunement element.** Null slots and unattuned damage resolve at 1.00×, which is the floor that guarantees no build is ever locked out.

Multi-element defenders multiply pairwise results, then clamp to [0.55, 1.85]. Without the clamp, a four-element enemy weak to your attunement reaches 2.86×.

## 8.7 `contracts.json` and `sectors.json`

```json
{
  "schema_version": 1,
  "default_challenges": [
    { "id": "no_life_lost",  "type": "complete_without_life_loss", "bonus_pct": 15 },
    { "id": "elite_run",     "type": "complete_with_rune", "clause": "elite", "bonus_pct": 15 },
    { "id": "no_pickups",    "type": "collect_no_pickups", "bonus_pct": 15 }
  ],
  "credit_formula": { "base": 150, "per_index": 22 },
  "contracts": [
    {
      "id": 7, "sector": 2, "index_in_world": 1,
      "display_name": "Shear Line",
      "mode": "throat", "kind": "normal",
      "wave_file": "res://data/waves/stage_07.json",
      "unlocks_after": [6],
      "requires_entitlement": null,
      "log_entry": "log_007",
      "introduces": ["husk"]
    }
  ]
}
```

`requires_entitlement` is **null for all 30 campaign contracts** — the campaign is free (`01` §1.7). It is non-null only for DLC side-sector contracts. One place in the codebase reads it (`09` §9.4).

## 8.8 Content-authoring loop with Cursor

1. You describe intent in prose: *"contract 14, Field mode, ~2:30, teaches crossfire — two Anchor nests at opposite edges plus continuous rammer pressure. Roster: anchor, spike, lash. Slightly above contract 13."*
2. Cursor writes `waves/stage_14.json` against this schema.
3. CI validates and runs the balance sim, reporting estimated clear time.
4. You play it and adjust two numbers.

Write the prompt into `docs/prompts/contract-authoring.md` so it's identical every time. Consistency across 30 contracts is what makes the campaign feel authored rather than assembled.

## 8.9 Tooling

| Tool | Effort | Value |
|---|---|---|
| JSON schema validator (CI) | 1 day | Catches most AI-generated content errors |
| Node completeness + VFX-uniqueness check | ½ day | Stops combos becoming recolours |
| In-game data hot-reload (dev builds) | 1 day | Tune balance without rebuilding |
| Wave visualizer (timeline → image) | 2 days | Review a contract without playing it |
| Balance sim (`sim/`) | 4 days | The only way to balance 14 nodes solo |
| Clause-stack simulator | ½ day | Verify the +150% cap holds against the index anchor |
| PRD constant table generator | 2 hrs | Solve C for 1–60% once, ship as data |
| Matchup matrix report | 1 day | Every attunement config × every contract's affinity mix; surfaces 0.8×-floor and DLC-cap violations |
| Attunement config previewer | ½ day | Shows effective multiplier per config against a chosen enemy set — doubles as the Hangar's live readout |
