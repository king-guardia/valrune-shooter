# 16 — Status Effects

**Status: draft, revision 2 — duration policy set, lockout family collapsed.** Vocabulary is
fixed by [`14-CANON.md`](14-CANON.md) §8; numbers by [`15-CONSTANTS.md`](15-CONSTANTS.md).

This catalogs the statuses in `data/source/buffs_debuffs.csv`, settles the tag taxonomy (O-02)
and `invisible`'s owner (O-13), and specifies how immunity and override resolve.

**It also found eleven conflicts**, in §7. Two are outright bugs — one status that does nothing
and one name collision that produces wrong behavior rather than a compile error.

Revision 2 adds the duration policy (§4), the stacking model the Balance Lab needs (§3), and a
rework of the six statuses whose jobs overlapped. **The net is one status deleted and one
added**: `stasis` folds into a new `paralyze_plus`, leaving 35.

---

## 1. The catalog

35 statuses across 20 families. Ids are `snake_case`; `+` becomes `_plus`.

**Max class** is the highest threat class the status can land on, derived from the three
immune columns. `minion` also covers debris.

| Id | Family | Element | Polarity | Tags | Duration | Stacks | Max class |
|---|---|---|---|---|---|---|---|
| `corrosion` | corrosion | CAUSTIC | debuff | `reactive` | 10s | 5 | minion |
| `corrosion_plus` | corrosion | CAUSTIC | debuff | `reactive` | 10s | 5 | boss |
| `poison` | poison | CAUSTIC | debuff | `weakness` | 10s | 5 | minion |
| `poison_plus` | poison | CAUSTIC | debuff | `weakness` | 10s | 5 | boss |
| `slow` | slow | CHRONO | debuff | `control` | 3s | — | minion |
| `slow_plus` | slow | CHRONO | debuff | `control` | 3s | — | elite |
| `rime` | rime | CRYO | buff | `retaliation` | source | — | any |
| `rime_plus` | rime | CRYO | buff | `retaliation` | source | — | any |
| `evasion` | evasion | ETHER | buff | `avoidance` | source | — | any |
| `evasion_plus` | evasion | ETHER | buff | `avoidance` | source | — | any |
| `evasion_plus_plus` | evasion | ETHER | buff | `avoidance` | source | — | any |
| `ethereal` | ethereal | ETHER | buff | `avoidance` | source | — | any |
| `ethereal_plus` | ethereal | ETHER | buff | `avoidance` | source | — | any |
| `stagger` | stagger | FORGE | debuff | `control` | 0.1s | — | minion |
| `stagger_plus` | stagger | FORGE | debuff | `control` | 0.2s | — | elite |
| `radiate` | radiate | GAMMA | buff | `geometry` | 5s | — | any |
| `radiate_plus` | radiate | GAMMA | buff | `geometry` | 5s | — | any |
| `invisible` | invisible | **GAMMA** | buff | `avoidance` | source | — | any |
| `burn` | burn | PLASMA | debuff | `dot` | 5s | — | minion |
| `burn_plus` | burn | PLASMA | debuff | `dot` | 5s | — | boss |
| `gravity` | gravity | VOID | debuff | `control` | source | — | minion |
| `gravity_plus` | gravity | VOID | debuff | `control` | source | — | elite ⚠ |
| `shock` | shock | VOLT | debuff | `reactive` | 5s | — | minion |
| `shock_plus` | shock | VOLT | debuff | `reactive` | 5s | — | boss |
| `maxhp_loss` | maxhp_loss | — | debuff | `vulnerability` | source | 5 | boss ⚠ |
| `maxhp_loss_plus` | maxhp_loss | — | debuff | `vulnerability` | source | 5 | boss |
| `maxhp_gain` | maxhp_gain | — | buff | — | source | — | any |
| `maxhp_gain_plus` | maxhp_gain | — | buff | — | source | — | any |
| `paralyze` | paralyze | — | debuff | `control` | source | — | minion ⚠ |
| `paralyze_plus` | paralyze | — | debuff | `control` | source | — | elite ⚠ |
| ~~`stasis`~~ | — | — | — | — | — | — | **deleted** ⚠ |
| `override` | override | — | buff | — | endless | — | any |
| `override_plus` | override | — | buff | — | endless | — | any |
| `damage_immune` | damage_immune | — | buff | — | source | — | any |
| `untargetable` | untargetable | — | buff | `avoidance` | source | — | any |
| `ward` | ward | — | buff | — | endless | charges | any |

⚠ flags a conflict resolved in §7. `ward` is the renamed `Shield` status — see §7.2.

**There is no `Misc` element.** That column in the spreadsheet is an authoring convenience;
in data those statuses carry `owner_element: null`.

Every one of the nine elements owns at least one status family, so element immunity (§5) has
teeth everywhere.

---

## 2. Tags — O-02 resolved

D-015 proposed five: `control`, `dot`, `vulnerability`, `avoidance`, `movement`. **Testing
them against all 20 families, two failed and one had no members.** The corrected set is six:

| Tag | Means | Families |
|---|---|---|
| `control` | Restricts movement or action | stagger, paralyze, stasis, gravity, slow |
| `dot` | Damages the bearer on its own timer | burn |
| `reactive` | Adds damage **only when something else hits the bearer** | corrosion, shock |
| `vulnerability` | Bearer takes more damage, or has less HP to take it with | maxhp_loss |
| `weakness` | Bearer **deals** less damage | poison |
| `avoidance` | Bearer cannot be hit | ethereal, evasion, invisible, untargetable |

Three changes from D-015, each forced by a family that would not fit:

**`movement` is deleted — it had no members.** D-015 listed it as "push, pull, slow", but
push and pull stopped being statuses when D-075 made them ability parameters, and slow is
`control`. Keeping an empty tag would mean immunity authors have to remember two tags for
one concept, and eventually one of them gets forgotten on a boss.

**`weakness` is split out of `vulnerability`.** Poison makes the bearer *deal* less; corrosion
and maxhp_loss make it *take* more. Opposite directions, and an immunity that covers one
should not silently cover the other — a boss immune to being weakened is a different design
statement than a boss immune to being softened up.

**`reactive` is split out of `dot`.** Burn ticks by itself; corrosion and shock only fire when
something else lands a hit. That is a real mechanical difference, not a shade of one: **burn
kills a baddie that flees, corrosion does not.** Tagging them alike would make the balance
engine value corrosion as though it were guaranteed damage, when it is actually conditional
on your own follow-up.

`retaliation` (rime) and `geometry` (radiate) appear in the catalog for completeness, but
**neither drives immunity** — see the next rule.

### Buffs carry tags, but not for immunity

D-015 said buffs carry no immunity data, which is right: nothing needs protection from being
buffed. But `avoidance` covers ethereal and evasion, which are buffs, and OverrideSet has to
name them.

> **Tags on debuffs feed ImmunitySet. Tags on buffs feed OverrideSet.** Same field, two
> consumers, no overlap.

---

## 3. Families

A family is a status and all its `+` forms. **Every gameplay condition references the family,
never a status id** (canon §8) — "bonus damage to baddies with burn" always means burn and
burn+.

```
burn      -> burn, burn_plus
evasion   -> evasion, evasion_plus, evasion_plus_plus
paralyze  -> paralyze
```

Referencing a bare status id in an ability condition is a schema error the validator rejects,
because it is nearly always this mistake.

**Single-member families are legitimate** — paralyze, stasis, ward, untargetable. The family
layer exists so conditions stay correct when a `+` form is added later, not because every
family needs one.

### Stacking pools the family, not the id

Only three families stack: corrosion, poison, maxhp_loss, all at 5.

**Stacks cap per family, not per status id** (D-083). Three `corrosion` plus two
`corrosion_plus` is five stacks at the cap, not five plus five. Per-id caps would make mixing
the `+` and base forms a way to double the cap, which nobody would author on purpose and
everybody would discover.

**Each stack keeps its own magnitude**, so the mixed case above deals 3n + 2y rather than
averaging. A new stack resets the timer for **all** stacks in the family (canon §8).

### Modelling stacks in the Balance Lab

**Assuming full stacks is close enough for some cases and badly wrong for others**, and the
exact formula is two lines, so there is no reason to approximate.

Given cap `C`, application rate `R` per second, and an exposure time `T`:

```
ramp = C / R
if ramp >= T:   average_stacks = R × T / 2        never reaches cap
else:           average_stacks = C − C² / (2 R T) caps, then holds
```

**`T` is target lifetime, not the 120-second window** — and that is the whole point. Run it
against a 5-stack status:

| Target | Lifetime | Applications/sec | Average stacks |
|---|---|---|---|
| Boss | 60s | 3.0 | **4.99** |
| Elite | 8s | 3.0 | **4.90** |
| Minion | 2s | 3.0 | **3.75** |
| Minion | 2s | 0.5 | **0.50** |

Against anything durable, full stacks is right. Against a minion hit by a slow ability, it
overstates value by ten times.

### This exposes a real conflict between stacking and the threat ladder

Follow the table above to its conclusion. **Stacking statuses need long-lived targets to be
worth anything.** But D-016 locks base-form debuffs to minions and debris, which are precisely
the targets that die before stacks accumulate.

So as currently authored, **base `corrosion` and base `poison` are close to worthless.** They
can only be applied to things that die in about two seconds, and their entire mechanic is
accumulation over time. All the value sits in `corrosion_plus` and `poison_plus`, which reach
bosses. The base forms are stubs.

Three ways out:

- **Exempt stacking debuffs from the ladder's base restriction**, letting base forms reach
  **elites** — long enough for three or four stacks — while `+` forms still gate bosses and
  minibosses. Preserves what D-016 is actually protecting.
- **Accept it** and price the base forms as pure chaff-clear, which means accepting that two of
  your four CAUSTIC statuses are filler.
- **Lower the cap and raise per-stack value** so ramp is fast enough to matter in two seconds.

**I lean the first.** D-016 exists to keep boss fights authorable, and letting corrosion stack
on an elite does not threaten that.

---

## 4. Durations — fixed by default, source-defined by exception

### What other games actually do, and why it does not transfer directly

The honest answer is that **most games put duration on the source**, but they get away with it
for a reason Valrune does not share.

| Game | Practice |
|---|---|
| **WoW** | Duration lives on the spell. Corruption is 14s, Immolate 18s |
| **MOBAs** | Duration lives on the ability. Each slow is its own slow |
| **Path of Exile** | Fixed base per ailment, modified by stats on the source |
| **Tower defense** | Duration lives on the tower upgrade |

**The catch: in all of those, each source applies its own private debuff.** WoW's Corruption
and Immolate are two different debuffs that happen to both be damage over time. They never
touch each other, so their durations can differ freely.

**Valrune is the opposite case.** You have *one* `burn`, applied by roughly fifteen abilities,
and every ability that reads "bonus damage to burning targets" reads the same family. Shared
statuses are the case where source-defined durations break down, for three reasons:

- **The refresh rule becomes incoherent.** Canon says reapplication never shortens duration. So
  reapplying a 3s burn over an 8s burn has to take the max — which means the longest source
  wins permanently and every other ability's duration silently stops mattering.
- **Balance surface explodes.** Fifteen abilities each carrying their own burn duration is
  fifteen numbers instead of one, and the Calibrator has to evaluate "applies burn" differently
  per source.
- **Nothing is learnable.** "Burn is five seconds" is a fact a player can hold. "Burn is
  somewhere between three and eight depending on what applied it" is not.

### The rule: riders are fixed, payloads come from the source

Your spreadsheet already sorts this way, which is why the question is worth formalizing rather
than deciding fresh. The discriminator is **whether the status is the ability's payload or a
rider on it**:

| | Definition | Duration | Examples |
|---|---|---|---|
| **Rider** | The ability does something else and the status comes along | **Fixed on the status** | burn, shock, corrosion, poison, slow, stagger, radiate |
| **Payload** | The status *is* the ability; its duration is the design lever | **`from_source`** | ethereal, evasion, rime, gravity, ward, invisible, damage_immune, untargetable, maxhp_gain |

"Become ethereal for 2 seconds" and "become ethereal for 5 seconds" are two different abilities
at two different prices, and the duration is the entire difference between them. That is a
payload. "Deals 40 plasma damage and applies burn" is a rider — the burn should be the same
burn every time, or the phrase means nothing.

**14 statuses fixed, 19 from source, 2 endless.**

### Abilities that want a longer rider use a multiplier band

Design space is preserved without handing out 89 free numbers. An ability may declare
`duration_scale`, drawn from a three-value band rather than raw seconds:

```
duration_scale: 1.0 (default) | 1.5 | 2.0
```

Banded so the Calibrator can still compare, and so "long burn" is a recognizable design move
rather than an arbitrary number. **The `+` form remains the preferred place to put a longer
duration**, since it already carries the escalation.

### Everything lands on the tick grid

**Every fixed duration is a multiple of `TICK_FAST` (0.1s)** after the stagger correction in
§7.4, and `duration_scale` may only produce values that stay on it — which is why the band is
1.0/1.5/2.0 and not 1.3.

Not cosmetic: an off-grid duration produces a final partial tick whose damage depends on
floating-point drift, breaking determinism.

---

## 5. Element ownership — O-13 resolved

`invisible` goes to **GAMMA**, your lean, and there is a mechanical argument for it beyond
ETHER being crowded.

Gamma already owns `radiate`, which manipulates how far effects reach. Invisibility is the
same idea pointed the other way — manipulating whether you register at all. And ETHER
already owns both avoidance families (`ethereal`, `evasion`); adding a third would make ETHER
the answer to every defensive question and leave GAMMA as a pure utility element with no
survivability at all.

### Element immunity cascades to statuses

> **Immunity to an element implies immunity to that element's statuses.**

A boss immune to PLASMA takes no burn and no burn+, with zero per-status authoring. This is
the entire reason `owner_element` exists as a field rather than a comment.

`owner_element: null` is correct for the eleven genuinely universal statuses — ward, stasis,
paralyze, maxhp_loss, maxhp_gain, override, damage_immune, untargetable and their `+` forms.
They are mechanics, not element expressions, and nothing should get them for free by being
immune to an element.

---

## 6. Resolution

```
can_apply(status, target, source):
    if target.immune_to(status) and not source.overrides(status):
        return false
    return true
```

**ImmunitySet** lives on an entity and holds `(status_id | family | tag | element)` entries.
Four entry kinds, each earning its place:

| Entry | Example | Why |
|---|---|---|
| `tag` | `control` | The threat-class ladder. One entry per boss, not twenty |
| `element` | `plasma` | The lava boss. Cascades to burn and burn+ free |
| `family` | `burn` | Bespoke: immune to fire but not to plasma generally |
| `status_id` | `burn_plus` | Escape hatch. Rare, and a code smell if common |

**OverrideSet** lives on an effect source and holds what it pierces, populated by the
`override` family.

**Resolution order is: immunity first, override second.** Override is strictly a
counter-mechanic — it cannot grant an application that was never possible, only restore one
that immunity took away.

### The threat-class ladder is derived, not authored

D-016's rule, restated against the corrected tags:

> **`control` debuffs never land on bosses or minibosses in any form. The `+` form unlocks
> elites only.**
>
> **`dot`, `reactive`, `vulnerability`, and `weakness` debuffs pierce everything in their `+`
> form.**

Auditing all 17 debuffs against the spreadsheet's immune columns, **15 match exactly.** The
two that do not are §7.7 and §7.8.

So the immune columns are not data to be maintained — they are a **generated consequence of
tag plus form**, and Phase 2 should generate rather than import them. That removes 51
hand-maintained booleans and, more usefully, makes it impossible to author a boss that is
accidentally stunnable.

The payoff for the player is legible: invest an element to tier III and your damage over time
starts biting bosses, while crowd control never will. Boss fights stay authorable and element
investment visibly pays off against them.

---

## 7. Conflicts found

Ordered by severity.

### 7.1 `override` in its base form does nothing at all

The spreadsheet defines it as:

> *your non-basic attacks override ethereal and evasion*

But both of the things it pierces only ever block basic attacks:

- `ethereal` — "cannot be hit by basic attacks"
- `evasion` — applies only to basic projectile shots (D-017)

**So a non-basic attack already lands against both.** Base `override` grants permission that
was never withheld. It is a purchasable no-op, and the kind that would survive playtesting
because nobody can perceive the absence of an effect that never existed.

**Proposed fix — move it one rung down the ladder rather than up:**

| Status | Pierces |
|---|---|
| `override` | Your **basic attacks** pierce `ethereal` and `evasion` |
| `override_plus` | **Everything you do** pierces the whole `ethereal` and `evasion` families, including `evasion_plus_plus` |

Now the base form does the obvious job — makes your gun work against a dodging target — and
the `+` form is a real escalation rather than a clarification. It also gives the `+` a unique
capability: `evasion_plus_plus` is otherwise unpierceable by anything in the game.

### 7.2 `ward` — the `Shield` status collides with the `shield` rank line

Two entirely different mechanics currently share one name:

| | Mechanic |
|---|---|
| `shield` the **rank line** | A regenerating HP pool, paired with `shield_recharge` |
| `Shield` the **status** | N charges, each negating one hit completely |

A pool that absorbs damage and a charge that negates a hit are not variations on a theme —
they behave differently against one big hit versus many small ones, which is exactly the
distinction a player needs to reason about. Worse, this collides at the code level in a way
that produces wrong behavior rather than an error: `entity.shield` is ambiguous, and both
readings compile.

**Renaming the status to `ward`** is the cheaper side to change — the rank line is referenced
by the whole progression tree, the status by a handful of abilities. "Ward charges" also
reads naturally, and `barrier` was the alternative but risks confusion with field objects.

### 7.3 Three statuses are competing to be the lockout

`paralyze`, `stasis`, and `gravity_plus` all stop a target from acting, and you are right that
this is redundant rather than layered:

| | Cannot move | Cannot act | Cannot rotate | Forced movement | Durations pause |
|---|---|---|---|---|---|
| `gravity` | ✓ | | | ✓ toward center | |
| `gravity_plus` | ✓ | ✓ | | ✓ toward center | |
| `paralyze` | ✓ | ✓ | | | |
| `stasis` | ✓ | ✓ | ✓ | | ✓ |

`gravity_plus` is `paralyze` with a pull bolted on. `stasis` is `paralyze` with two extra
clauses. **Three statuses, one mechanic, escalating by accretion.**

**Collapse to one family, and give `gravity` its own lane.**

```
paralyze       cannot move or act. Lands on minions.
paralyze_plus  also cannot rotate; passives and status durations pause. Lands on elites.
```

`stasis` is **deleted**, its two distinctive clauses becoming what makes `paralyze_plus` a `+`
form. Nothing is lost — the duration-pause clause is genuinely useful and worth keeping, it
just does not need its own status to live in.

This resolves three things at once: the polarity error (stasis was marked a buff despite being
strictly worse than a debuff), O-29 (paralyze had a `+` form's reach with no base form, and now
has both), and O-32 (nobody could say who applied stasis, because nothing did).

**`gravity_plus` stops denying actions.** Its escalation becomes strength of pull rather than a
borrowed lockout, which leaves gravity as what VOID should own — **trajectory control, not
paralysis.** Being dragged somewhere while still able to shoot is a different and more
interesting problem for the player than being switched off.

### 7.3b Gravity should affect projectiles, and that is the whole answer

Your instinct here is the best idea in this batch. **Nothing else in the game manipulates
projectiles in flight**, and it gives VOID an identity that no other element can copy:

- **Defensively**, a gravity well bends incoming fire away from you.
- **Offensively**, it curves your own shots around cover or into a cluster.
- **It interacts with `homing` without duplicating it.** Homing corrects toward a target;
  gravity displaces regardless of intent. Two forces summing is well-defined, and a homing shot
  fighting a gravity well is a genuinely interesting moment.

**It belongs to the field object, not the status.** The `gravity` status is "this entity is
being pulled"; projectile bending is a property of the VOID field that does the pulling. Same
field, two effects, one authored place. The push/pull composition in D-075 already almost
expresses it.

Cost is low: projectiles already carry position and velocity, so applying an acceleration to
those inside a radius is a handful of lines and is fully deterministic. The performance
question is how many gravity fields can exist at once, which is a wave-authoring limit rather
than an engine problem.

### 7.4 `stagger` is shorter than a tick

Authored at **0.05s and 0.09s** against `TICK = 0.2s` (D-048). Neither can be represented on
the tick grid, so both would round to something 2–4× their intent.

The design goal is right — §6 of the constants calls `push_1` "a stutter, not a launch" at
0.08s, and stagger is that same beat. It just needs to land on the grid:

| Status | Was | Now | Frames at 60fps |
|---|---|---|---|
| `stagger` | 0.05s | **0.1s** (`TICK_FAST`) | 6 |
| `stagger_plus` | 0.09s | **0.2s** (`TICK`) | 12 |

Still brief, still reads as an interruption rather than a stun, and now deterministic.

### 7.5 `slow` and `evasion` violate the percentage ban

D-014 restricts percentages to a closed allowlist of crit chance, crit damage, and Overclock.
But `slow` is 0.85× and 0.65× speed, and `evasion` is 15/25/35%.

**Flat values are genuinely wrong here.** A flat −18 u/s slow would nearly stop a 120 u/s
minion and be imperceptible on a 1000 u/s Valrune. The mechanic is multiplicative by nature,
the way crit damage is.

The allowlist does not need two more entries so much as **a better-stated rule**:

> Percentages are allowed where the quantity is **inherently a chance or a multiplier**. Every
> additive stat bonus is flat.

That covers crit chance, crit damage, Overclock, evasion, and slow without needing a
maintained list, and it still bans the thing D-014 was actually aimed at — `+8% velocity`
style rank lines that compound into increasing returns.

### 7.6 `radiate`'s band shift is now wildly non-uniform

Radiate promotes an ability's geometry one rung: `r_short → r1 → r2 → r3`. That was roughly
even on the old evenly-spaced ladders. On the D-073 ladders it is not:

| Shift | Gain |
|---|---|
| `r_short → r1` | +50 units |
| `r2 → r3` | +150 units |
| `dis2 → dis3` | +455 units |
| `dis3 → dis4` | **+800 units** |

**The same buff is worth sixteen times more on one ability than another**, decided entirely by
which rung the ability happened to be authored at. And there is no rule for what happens at
the top of a ladder.

Three ways out, and this needs your call:

- **Cap at the top rung.** Simple; makes radiate dead on max-range abilities.
- **Percentage increase instead of a band shift** — say +50% radius. Uniform by construction,
  but produces off-band geometry, which §5 of the constants exists to prevent.
- **A flat additive bonus** — say +100 units of radius or reach. Uniform, stays roughly
  on-band, and is worth proportionally more to small abilities, which is arguably correct
  since they are the ones that need help.

**I lean the flat bonus.** It respects D-014, it cannot break at the top of a ladder, and it
does not require radiate to know what band its target ability used.

### 7.7 `maxhp_loss` breaks the ladder in its base form

Every other base debuff is immune on all three upper classes. `maxhp_loss` is immune on none —
its base form lands on bosses, which by D-016 only `+` forms should do.

Almost certainly an oversight rather than intent, since `maxhp_loss_plus` exists and would be
identical in reach. **Recommend the base form drop to `minion`**, matching every other
`vulnerability` debuff and giving the `+` form a reason to exist.

### 7.8 `paralyze` has a `+` form's reach with no base form

`paralyze` lands on elites — the profile of a `+` form — while being the only member of its
family. So the cheapest possible application of paralyze reaches as far as a fully invested
`stagger_plus`.

Two readings, both defensible:

- **Intentional**, because paralyze is the Chrono time-stop replacement (D-003) and is meant
  to be a heavy, expensive effect with no cheap version. Then it should be **priced like a
  `+` form**, and the Calibrator needs to know that.
- **Incomplete**, and it wants a base `paralyze` that stops at minions, with the current
  behavior becoming `paralyze_plus`.

I lean intentional — a single strong control effect is cleaner than padding the family for
symmetry — but the pricing consequence has to be recorded either way, or the Calibrator will
compare it against base forms and pass it.

### 7.9 The five defensive statuses are four flags, not five statuses

`ethereal`, `damage_immune`, `untargetable`, and `invisible` overlap because they were each
defined whole rather than composed. They separate cleanly on **four independent axes**:

| Status | Targetable | Collides | Takes damage | Visible |
|---|---|---|---|---|
| `ethereal` | yes | no, basic only | no, basic only | yes |
| `ethereal_plus` | yes | no | no | yes |
| `damage_immune` | yes | yes | **no** | yes |
| `untargetable` | **no** | yes | yes | yes |
| `invisible` | **no** | **no** | AoE breaks it | **no** |

Defining the flags and letting statuses set them removes the overlap without removing any of
the statuses, because **each one occupies a genuinely distinct cell.**

**The distinction that makes `damage_immune` worth keeping: ethereal *misses*, damage immunity
*lands for zero*.** An attack that misses applies nothing — no burn, no corrosion, no on-hit
rider. An attack that lands for zero still applies everything else. That is a real difference
and it is exactly the boss phase you would want: *you cannot hurt it yet, but you can stack
corrosion for when the shield drops.*

Which means **`damage_immune` should stop blocking debuffs.** The spreadsheet reads "cannot take
damage or debuffs", and dropping the second clause is what makes it distinct from `ethereal+`
instead of a duplicate of it. It also simplifies D-091 — no ImmunitySet manipulation, just
`damageable: false`.

**`untargetable` earns its place as the anti-homing status**, which is what you built it for.
It breaks target acquisition without granting any damage protection, so skillshots and AoE
still land. Now that `homing` is a rank line every player buys, an untargetable baddie is real
counterplay rather than a curiosity — and it is a natural GAMMA signature.

**`invisible` grants `untargetable`** rather than restating the targeting rules, including that
careful clause about launched abilities flying straight instead of homing. It adds only what is
its own: not being drawn, no collision, and AoE breaking it.

So yes — **invisibility breaks homing**, via `untargetable`, and it does so without you having
to give any baddie homing for it to matter. It breaks *yours*.

### 7.10 `ward` is worth sixty times more against a boss than a swarm

Charge-based hit negation has one structural flaw: **a charge blocks a hit regardless of the
hit's size**, so its value is set entirely by the largest thing you can spend it on.

| Absorbing | Charges | Damage blocked |
|---|---|---|
| Chaff hits at 3 damage | 3 | 9 |
| Boss special at 200 damage | 3 | 600 |

**And piercing makes it worse.** D-081 turns one gun-shot into five hits, so in a 200-baddie
swarm ward evaporates in a fraction of a second. It is simultaneously useless where you are
being chipped to death and dominant where you are being hit once every five seconds.

Three fixes, and they are not equivalent:

- **Make it an HP pool.** Predictable, but that is the `shield` rank line already — and the
  whole reason ward got renamed in §7.2 was that the two are different mechanics.
- **Scope it to basic attacks**, mirroring evasion's D-017 fix. Consistent, but `evasion` and
  `ethereal` already cover basic attacks, so ward would become the third answer to a question
  already answered twice.
- **Give the charge an internal cooldown.** ← recommended

```
ward: N charges. Consuming one puts ward on a 1.0s internal cooldown,
      during which damage passes normally.
```

**This fixes both ends with one number.** In a swarm ward blocks one hit per second rather than
evaporating, so it is a real if modest defensive window. Against a boss it still blocks the big
telegraphed hits, which is the job no other defensive status can do — evasion cannot touch
specials by D-017, and ethereal+ only helps if you timed it.

It also **resolves O-31 cleanly**: a five-hit pierced gun-shot consumes one charge, not five,
because the cooldown starts at the first hit.

The "certain abilities can strip a ward" idea is good and stays available — it is a natural
elite mechanic — but it is additional scope rather than a fix, so it should wait until ward is
in and felt.

### 7.11 Every description is written in second person

Roughly a third of the `affect` strings say "you" or "your" — rime, radiate, override, stasis,
invisible. The rest are written from the baddie's side.

**Nearly all of these work on either side**, and some are more interesting for it: a baddie
with `rime` punishes you for shooting it, which is a good elite mechanic that the current
phrasing hides.

Phase 2 should rewrite every description in bearer-neutral terms — "the bearer", "attackers" —
rather than adding a `bearer` field to constrain them. The exception, if one survives review,
is stasis's control-lockout clause, which genuinely has no baddie-side meaning.

---

## 8. What this changes elsewhere

| Document | Change |
|---|---|
| `14-CANON.md` §8 | Tag list grows to six; `movement` deleted |
| `15-CONSTANTS.md` | `stagger` durations to 0.1s / 0.2s |
| Decision log | D-014 percentage rule restated; D-015 tags corrected; D-016 audited |
| `data/source/buffs_debuffs.csv` | Source of truth for text, but the immune columns become **generated** |

---

## Open questions

| # | Question | Blocks |
|---|---|---|
| 1 | **`radiate`'s band shift** — cap, percentage, or flat bonus? §7.6. Leaning flat | Phase 2 |
| 2 | **Stacking versus the threat ladder** — base `corrosion` and `poison` can only land on targets that die before stacks accumulate. Let base stacking debuffs reach elites? §3 | Phase 2 |
| 3 | **Is `rime`'s recoil a basic attack?** If it is, it can be evaded, and two rimed entities shooting each other need a recursion guard | Phase 2 |
| 4 | **Gravity affecting projectiles** is new scope, not a correction — worth confirming before it lands in the schema. §7.3b | Phase 2 |
| 5 | **Does anything strip a `ward`?** Good elite mechanic, deliberately deferred until ward is in and felt | M1 |

Resolved this pass: O-29 (`paralyze` gains a base form), O-31 (ward's internal cooldown),
O-32 (`stasis` deleted).
