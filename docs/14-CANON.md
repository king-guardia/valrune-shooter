# 14 — Canon

**Status: draft, awaiting Bryan's review.** This is the vocabulary contract. Once you
correct it, these terms are used exactly as written — in documents, in data, and in code.

Three rules for reading this:

1. **Terms are not interchangeable.** If two words appear here, they mean different
   things. Where a word is listed as banned, it has been banned because it was being used
   for two different concepts.
2. **The "Code" column is the literal identifier**, in `snake_case` for members and
   `PascalCase` for classes, per `.cursor/rules/20-gdscript-style.mdc`.
3. **[CONFIRM]** marks a call I made that you should check. They are collected at the
   bottom under [Open questions](#open-questions) — nine of them.

---

## 1. People and ships

Your `misc_ideas` line 10 asks whether there is a better designation here. There is, and
it is the reverse of what the spreadsheet proposes.

| Term | Means | Code |
|---|---|---|
| **Player** | **The human being.** Always. Menus, ship control, analytics — all the same person. | `Player` |
| **Valrune** | **The player's ship.** The entity in the world with hull, position, and heading. | `Valrune` |
| **Pilot** | Fiction only. Never a code identifier. | — |

**Banned: `user`.** [CONFIRM]

The spreadsheet has `player` = the ship and `user` = the human. I am recommending the
opposite, for one reason: every engine, tutorial, and code sample in existence uses
"player" for the human or their avatar. Inverting that guarantees both of us slip, and a
slip is invisible until it produces a bug. Meanwhile the ship has an actual proper name —
*Valrune* — which is a better identifier than "player" ever was.

So: the **player** decides to dodge; the **Valrune** moves.

---

## 2. Enemies

| Term | Means | Code |
|---|---|---|
| **Baddie** | **Anything the player can damage.** Minions, elites, minibosses, bosses, asteroids, debris. The universal term. | `Baddie` |
| **Threat class** | A baddie's rung on the immunity and scaling ladder. | `threat_class` |
| **Minion** | Threat class 1. Ordinary enemies. | `MINION` |
| **Elite** | Threat class 2. Upgraded variant. | `ELITE` |
| **Miniboss** | Threat class 3. Contract 3 of a sector. | `MINIBOSS` |
| **Boss** | Threat class 4. Contract 6 of a sector. | `BOSS` |
| **Hazard** | A damageable object that never acts: asteroid, debris. A baddie with no behaviour. | `HAZARD` |
| **Archetype** | The eight behaviour templates (Mote, Lash, Spike, Husk, Anchor, Warden, Seeder, Choir). | `archetype` |
| **Role tag** | Stackable modifier: fast, tough, repairing, mage, decoy, mob. | `role_tags` |
| **Affinity** | The elements a baddie's *defence* is typed as, for the type chart. | `element_affinity` |

**Banned: `enemy`, `mob`, `creep`, `NPC`.** Use **baddie**. (`mob` survives only as the
role-tag name, which is unfortunate but it is your word and it is already in the data.)

Threat class drives immunity (D-016), so it is a first-class field, not a label.

---

## 3. Weapons — the section that matters most

Your `misc_ideas` lines 19 and 37 ask for exactly this. These four words were doing
overlapping work and every DPS calculation depends on separating them.

| Term | Means | Code |
|---|---|---|
| **Gun** | **A mount on the ship.** You own 1, 2, or 3. Each is one attunement slot. Not a weapon type — a physical position. | `gun_count`, `GunMount` |
| **Gun-shot** | **One trigger event, across every gun at once.** Three guns firing simultaneously is *one* gun-shot. | `GunShot`, `fire_gun_shot()` |
| **Projectile** | **One bullet from one gun.** A 3-gun ship produces 3 projectiles per gun-shot. | `Projectile` |
| **Fire rate** | Gun-shots per second. **Never** projectiles per second. | `fire_rate` |

The gun-shot is the atomic unit. It is what fire rate counts, what crit rolls once per
(v0 `04` §4.5), and what "basic attack" resolves to for the Valrune. Getting this wrong
inflates DPS by 3× on a three-gun ship, which is precisely the error the balance work
exists to prevent.

**Banned: `volley`.** [CONFIRM] It is an exact synonym for gun-shot and v0 uses both.
One word, not two.

**Banned: `beam`.** [CONFIRM] `misc_ideas` line 6 says "all elements apply to all beams",
meaning projectile streams. But v0 `04` §4.9 gives GAMMA "beams" as an identity, which
sounds like a genuine continuous hitscan weapon. Those are different things and cannot
share a word. My proposal: the stream from a gun is a **projectile stream**, and if GAMMA
gets a real continuous weapon it is a **lance**.

**`fire` is a verb only.** The gun fires. There is no noun "fire" — the element is
**PLASMA**, the status is **burn**, the field object is a **flame trail**. This kills the
flame/shoot collision you flagged.

### Basic attack vs ability

This distinction is load-bearing. Evasion, ethereal, and poison all scope themselves by
it, and D-017 only holds together because the boundary is crisp.

| Term | Means | Code |
|---|---|---|
| **Basic attack** | An entity's **default repeating weapon fire**: governed by a *rate*, costs nothing, needs no unlock, is the entity's baseline damage. Valrune → the gun-shot. Baddie → its routine `shoot` action. | `is_basic_attack` |
| **Ability** | Anything governed by a **cooldown or duration**, or granted by a node or the mage role tag. Everything that is not a basic attack or contact damage. | `Ability` |
| **Contact damage** | Damage from **collision** — ramming, flying into debris. **A third category. Neither a basic attack nor an ability.** | `contact_damage` |

The test is *rate vs cooldown*. A basic attack fires on a rate forever. An ability has a
cooldown.

**Evadable = basic AND single-target.** [CONFIRM] Both conditions. This matters because
the CRYO attunement replaces your projectile with a wave blast — still a basic attack, but
now AoE, so not evadable. Note the source files disagree here: `buffs_debuffs` says
evasion avoids "single-target attacks" while D-017 says "basic projectile shots". Requiring
both reconciles them and keeps D-017's guarantee that AoE is never dodged.

Contact damage is never evadable — dodging a ram you flew into would read as a bug.

---

## 3a. Spawned objects

Things that exist in the world but are neither the Valrune nor a baddie.

| Term | Means | Code |
|---|---|---|
| **Drone** | An **undamageable, untargetable** companion that flies to `r1` of the nearest baddie and shoots it, while trying to stay within `r2` of the Valrune (D-020). Has no HP. | `Drone` |
| **Hologram** | A **type of drone**, not a decoy. It does not absorb hits. | `drone_type` |
| **Field object** | A persistent thing occupying space: mine, flame trail, fissure, zone, portal, wire net. Has a lifetime. | `FieldObject` |
| **Trail** | A field object emitted continuously from the Valrune's stern as it moves. | `TrailEmitter` |
| **Lifetime** | How long a spawned object exists before despawning. Distinct from a status **duration**. | `lifetime` |

A drone is not a baddie and not a ship — it cannot be damaged, targeted, or collided
with. The `decoy` role tag produces baddie-side illusions, which *are* damageable 1-HP
baddies. Those are unrelated to drones despite sounding similar.

---

## 4. Elements and attunement

| Term | Means | Code |
|---|---|---|
| **Element** | One of nine. Base: PLASMA, CRYO, FORGE, VOLT, CAUSTIC. DLC: CHRONO, GAMMA, VOID, ETHER. | `Element` |
| **Attunement** | **An element assigned to a gun.** The act and the assignment both. | `attunement` |
| **Attunement slot** | One gun's element position. Always qualified — never bare "slot". | `attunement_slots` |
| **Attunement effect** | The on-hit effect an attuned element applies. Scales 1.0 / 1.5 / 2.0 with element level. | `attunement_effect` |
| **Attunement crit** | The on-crit effect. Belongs to the attunement, not the node (D-010). | `attunement_crit` |
| **Type chart** | The strong/weak/cross-circle multiplier table. | `TypeChart` |
| **Matchup** | One resolution of attacker types against defender affinity. | `matchup_multiplier` |

It is **ETHER**. Never AETHER (D-005).

Element names are **SCREAMING_CASE in prose**, `snake_case` as ids (`plasma`, `ether`).

---

## 5. Progression

Two currencies, and they are never mixed up:

| Term | Means | Code |
|---|---|---|
| **Credit** | Currency from contracts. Buys the stat tree. | `credits` |
| **Element point** | Currency from minibosses and bosses. **10 total.** Raises element levels. | `element_points` |
| **Upgrade line** | One purchasable track in the stat tree (Damage, Hull, Gyros…). Fourteen of them. | `UpgradeLine` |
| **Rank** | **One purchased step within an upgrade line.** Attack Speed has 24 ranks. | `rank` |
| **Stat tree** | All fourteen lines together. | `StatTree` |
| **Element level** | An element's investment, 0–3. Costs element points. | `element_level` |

**Rank is credits. Tier is element points. They are never the same word.**

### Nodes and abilities

| Term | Means | Code |
|---|---|---|
| **Combination** | A set of 1–3 elements. 9 singles + 36 duals + 84 triples = **129**. | `combination` |
| **Node** | **One (combination × passive-or-active) position.** May be empty. **249 possible** (D-002). | `Node` |
| **Ability** | The effect a filled node grants. Has three tiers. | `Ability` |
| **Tier** | The I / II / III version of an ability. Driven by element level. | `tier` |
| **Passive** | Always-on. No activation. | `PASSIVE` |
| **Active** | Player-triggered, on a cooldown. | `ACTIVE` |
| **Loadout** | The five equipped nodes. | `Loadout` |
| **Loadout slot** | One of the five: 3 passive, 1 active, 1 auto-active (D-022). Always qualified. | `loadout_slots` |
| **Auto-active** | An active fired automatically the moment its cooldown ends. | `is_auto_active` |

A node is a *position*; an ability is *what fills it*. An empty node is a legitimate,
intended state (D-003) — not a bug and not a gap to be filled with a placeholder.

**Banned: bare `slot`.** Always `attunement slot`, `loadout slot`, or `node`.

---

## 6. Ability anatomy

From your `objects.csv`, cleaned up.

| Term | Means | Code |
|---|---|---|
| **Trigger** | What causes the ability to fire: activation, on-hit, on-crit, on-kill, proximity, always. | `trigger` |
| **Target spec** | Where the ability **reaches** — the region it selects baddies from. | `target_spec` |
| **Arrival spec** | Where the effect **lands**, which may differ from where it reached. | `arrival_spec` |
| **Shape** | `none, circle, oval, line, 3-point … 7-point`. | `shape` |
| **Duration** | How long the effect persists. | `dur` |
| **Cooldown** | Recharge time. **Begins when duration ends** (`misc_ideas` line 13). | `c` |
| **Charge** | A stored use. Cooldown starts when the *first* charge is spent (line 9). | `charges` |
| **Tick** | One periodic application. **`TICK_DOT = 0.2s`** (D-026). | `tick` |
| **Uptime** | Fraction of the window an effect is active: `dur / (dur + c)`. | `uptime` |
| **Proc** | One firing of a triggered effect. | `proc` |

Every active has a duration and a cooldown, even if the duration is zero
(`misc_ideas` line 5).

---

## 7. Damage

| Term | Means | Code |
|---|---|---|
| **Damage packet** | **The unit of damage resolution.** Carries a type set, resolves the type chart exactly once (D-007). | `DamagePacket` |
| **Damage type** | Named for the element: plasma damage, cryo damage. A packet may carry several. | `damage_types` |
| **Hit** | One damage packet landing on one baddie. What "on hit" counts. | `on_hit` |
| **Source** | The entity or object credited with an effect. Push is away from source, pull is toward it (line 27). | `source` |
| **Crit** | A gun-shot rolling critical. Multiplies **only the base gun packet** (D-008). Rolls **once per gun-shot**. | `is_crit` |
| **PRD** | Pseudo-random distribution. Crit chance climbs each gun-shot until it lands. | `prd_table` |
| **DoT** | Damage over time. Ticks every 0.2s. | `dot` |

**A crit is not a source** (`misc_ideas` line 24). Anything that says "when a source
causes X" excludes crits.

**No self-damage** (line 26). Player field objects never hurt the Valrune. Baddie
effects never hurt baddies. Baddie projectiles pass through other baddies.

### Variable naming

Your convention from `misc_ideas` line 12, adopted as-is:

`r` radius · `dis` distance · `dur` duration · `c` cooldown · `s` stacks ·
`dot` damage-over-time damage · `dam` damage · `n` any number without a classification

**`n` is banned in shipped data.** [CONFIRM] It is fine in your spreadsheets as
"a number to be determined later", but a `null` in JSON says the same thing
unambiguously and the validator can find it.

---

## 8. Statuses

| Term | Means | Code |
|---|---|---|
| **Status** | Any timed effect on an entity. The umbrella term. | `Status` |
| **Buff** | Beneficial status. Carries no immunity data (D-015). | `BUFF` |
| **Debuff** | Detrimental status. | `DEBUFF` |
| **Polarity** | buff or debuff. | `polarity` |
| **Tag** | Behaviour class driving immunity: `control`, `dot`, `vulnerability`, `avoidance`, `movement` (D-015). | `tags` |
| **Stack** | One application of a stacking status. New stack resets the timer for **all** stacks (line 39). | `stacks` |
| **The `+` form** | A stronger variant that also pierces further up the threat ladder (D-016). A **separate status**, not a stack level. | `burn_plus` |
| **ImmunitySet** | What an entity cannot receive. Derived from tags. | `ImmunitySet` |
| **OverrideSet** | What an effect source pierces. | `OverrideSet` |

**Statuses apply after damage resolves** (D-009), reversing `misc_ideas` line 11.

**Reapplication never shortens.** If a fresh application would reduce remaining duration,
it does not apply (line 32).

**Statuses are element-anchored** (line 38). Burn is only ever PLASMA. Corrosion is only
ever CAUSTIC. The `Misc` element in `buffs_debuffs.csv` is a **spreadsheet convenience,
not an element** [CONFIRM] — those statuses (shield, stasis, paralyze, invisible,
MaxHP_Loss, Override) are universal and need an `owner: null`, because there is no Misc
element and there never will be.

---

## 9. Geometry

| Term | Means | Code |
|---|---|---|
| **Playfield** | The battle area. **1000 units wide** (D-024). | `PLAYFIELD_WIDTH` |
| **Ground** | Synonym for playfield (line 33). **Banned** [CONFIRM] — it is space; "playfield" is unambiguous. | — |
| **Bow** | Front of a ship (line 8). | `bow` |
| **Stern** | Back of a ship. | `stern` |
| **Port / starboard** | Left / right, ship-relative. | `port`, `starboard` |
| **Heading** | The direction a ship faces. Distinct from travel direction — movement is screen-relative. | `heading` |
| **Ready line** | Where recall returns you. Throat: bottom-centre. Field: arena origin. | `ready_line` |
| **Radius band** | `r_short`, `r1`, `r2`, `r3`. A region **around** a point. | `r1` |
| **Distance band** | `dis_short`, `dis1`–`dis4`. Reach **away from** a point. `disN` matches `rN` (line 18). | `dis1` |
| **Width band** | `w1`, `w2`. | `w1` |
| **Push impulse** | Displacement over ~0.08s. **Does not override movement** (D-025). | `push_impulse` |
| **Pull speed** | Sustained units/sec toward a source. Overrides movement only with `gravity` (D-025). | `pull_speed` |

**Banned: `mr` / move rate.** [CONFIRM] Your line 17 floats it, line 16 then explains why
push and pull behave differently. One variable cannot carry both. Split per D-025.

Tooltips say "short / medium / long" **plus the unit count**. Data says `r2`.

---

## 10. Structure

| Term | Means | Code |
|---|---|---|
| **Contract** | One mission. 30 in the campaign. The unit of progression and payout. | `Contract` |
| **Stage** | One playable run of a contract, ~2–4 minutes. | `Stage` |
| **Sector** | Six contracts. Five sectors. | `Sector` |
| **Wave** | One authored group of baddies within a stage. | `Wave` |
| **Throat** | Vertical contract inside a wormhole. Horizontal wrap, one screen. | `THROAT` |
| **Field** | Circular arena contract. Hard boundary, camera follows. | `FIELD` |
| **Clause** | A voluntary difficulty modifier for higher pay. 4 slots. | `Clause` |
| **Hangar** | Between-contract screen: spend credits, set loadout. | `HangarScreen` |

**Banned: `level`, `mission`, `map`.** [CONFIRM] `level` especially — it already means
element level, and a third meaning would be fatal.

---

## 11. Baddie behaviour

From `misc_ideas` lines 21 and 22.

| Term | Means | Code |
|---|---|---|
| **Action** | One behaviour a baddie is performing, with a duration. **Actions overlap** — a baddie can be moving and shooting at once. | `BaddieAction` |
| **Idle** | An action. Doing nothing, for a duration. | `IDLE` |
| **Telegraph** | Visual warning before an action. **White** = movement, **red** = attack. Soft highlight extending in the direction, then a solid glow, then the action. | `Telegraph` |
| **Path type** | The shape a move action follows. | `path_type` |

A baddie is a **set of concurrent actions**, not a single state. This is why it is a state
*machine per action*, not one machine per baddie — Phase 3 diagrams it.

---

## Open questions

Nine calls I made that need your yes or no. Everything else above I am confident about.

| # | Question | My call | Cost if I'm wrong |
|---|---|---|---|
| 1 | **`player` = human, `valrune` = ship** — inverting your spreadsheet | Swap it | Rename across all docs; cheap now, expensive after code |
| 2 | **Ban `volley`**, use gun-shot only | Ban | Trivial |
| 3 | **Ban `beam`** — is GAMMA's "beam" a real continuous weapon, or just flavour for a projectile stream? | Need your answer | Affects whether we need a hitscan damage path at all |
| 4 | **Evadable = basic AND single-target** — so the CRYO wave blast can't be dodged | Both conditions | Changes evasion's value materially |
| 5 | **Ban bare `n`** in shipped data, use `null` | Ban | Trivial |
| 6 | **`Misc` is not an element** — those statuses get `owner: null` | Not an element | Schema shape |
| 7 | **Ban `ground`**, use playfield | Ban | Trivial |
| 8 | **Ban `mr`**, split into push_impulse and pull_speed | Split | Already decided as D-025; confirming the naming |
| 9 | **Ban `level`/`mission`/`map`** for contracts | Ban | Trivial |

Question 3 is the only one with real downstream cost. The rest are naming.

### Two things I could not find

- **Is there a term for the baddie equivalent of a loadout?** The mage role tag draws
  "one ability per placement from a pool" — that pool needs a name.
- **What is a baddie's basic attack called in fiction?** The Valrune has guns. The Hollow
  presumably have something, and `05` may already name it.
