# 16 — Status Effects

**Status: draft, revision 1 — awaiting Bryan's review.** Vocabulary is fixed by
[`14-CANON.md`](14-CANON.md) §8; numbers by [`15-CONSTANTS.md`](15-CONSTANTS.md).

This catalogs all 35 statuses in `data/source/buffs_debuffs.csv`, settles the tag taxonomy
(O-02) and `invisible`'s owner (O-13), and specifies how immunity and override resolve.

**It also found eleven conflicts.** Two are outright bugs — one status that does nothing and
one name collision that would produce wrong behavior rather than a compile error. Those are
in §7, which is the part worth reading closely.

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
| `gravity_plus` | gravity | VOID | debuff | `control` | source | — | elite |
| `shock` | shock | VOLT | debuff | `reactive` | 5s | — | minion |
| `shock_plus` | shock | VOLT | debuff | `reactive` | 5s | — | boss |
| `maxhp_loss` | maxhp_loss | — | debuff | `vulnerability` | source | 5 | boss ⚠ |
| `maxhp_loss_plus` | maxhp_loss | — | debuff | `vulnerability` | source | 5 | boss |
| `maxhp_gain` | maxhp_gain | — | buff | — | source | — | any |
| `maxhp_gain_plus` | maxhp_gain | — | buff | — | source | — | any |
| `paralyze` | paralyze | — | debuff | `control` | source | — | elite ⚠ |
| `stasis` | stasis | — | **debuff** ⚠ | `control` | source | — | any ⚠ |
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

---

## 4. Durations

Three kinds, and the schema needs to say which:

| Kind | Meaning | Count |
|---|---|---|
| Fixed seconds | The status defines it | 10 |
| `from_source` | The applying ability defines it | 22 |
| `endless` | Until removed or the contract ends | 3 |

Twelve statuses read "variable" in the spreadsheet, which means `from_source`. Two read
"endless" and one "until removed/endless" — the same thing.

**Every fixed duration is a clean multiple of `TICK` (0.2s)** after the stagger correction in
§7.4. That is not cosmetic: a duration off the tick grid produces a final partial tick whose
damage depends on floating-point drift, which breaks the determinism rule.

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

### 7.3 `stasis` is a total lockout marked as a buff

The spreadsheet has it as a buff, but the effect is:

> *cannot move other than forced movement; can't rotate, attack, use abilities; passives are
> stopped/paused; you cannot return to ready position; all player controls no longer do
> anything*

That is `paralyze` plus rotation lock plus passive suspension plus recall denial — **strictly
worse than a status the same file marks as a debuff.**

The rule that resolves it: **polarity describes the effect on the bearer, not the intent of
whoever applied it.** Stasis harms its bearer, so it is a debuff with tag `control`.

This matters beyond bookkeeping, because D-015 says buffs carry no immunity data. Left as a
buff, **nothing in the game could ever be immune to stasis** — including bosses, which the
`control` ladder is specifically built to protect.

One clause survives as a genuine benefit and should be kept explicit: **buff and debuff
durations pause for the duration of stasis**, so it does not burn through your other statuses.
That is a property of the status, not a reason to call it a buff.

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

### 7.9 `damage_immune` bundles two separable mechanics

> *cannot take damage or debuffs*

Debuff immunity is precisely what ImmunitySet does. Bundling it in means the framework has a
hardcoded special case sitting next to the general mechanism that already handles it.

**Decompose it:** `damage_immune` grants damage immunity, and separately adds every debuff tag
to the bearer's ImmunitySet. Same behavior, no special case, and it becomes possible to author
"immune to damage but still debuffable" — which is a boss phase somebody will eventually want.

### 7.10 `invisible` restates `untargetable`

`invisible` is `untargetable` plus no collision plus not being drawn. Rather than duplicating
the targeting rules — including that careful clause about launched abilities flying straight
instead of homing — **`invisible` should grant `untargetable`** and add only what is genuinely
its own.

Statuses granting statuses is a capability the schema needs anyway; this is a good first use.

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
| 2 | **`paralyze`** — intentionally a `+`-form-strength singleton, or does it want a base form? §7.8 | Phase 2 |
| 3 | **Is `rime`'s recoil a basic attack?** If it is, it can be evaded, and two rimed entities shooting each other need a recursion guard | Phase 2 |
| 4 | **Does `ward` block a whole gun-shot or one pierced hit?** Piercing (D-081) makes one gun-shot into five hits. Per hit, ward is five times weaker than authored | Phase 2 |
| 5 | **`stasis` — who applies it, and to whom?** The description reads as a boss mechanic on the player, but nothing in the ability set produces it yet | Phase 2 |
