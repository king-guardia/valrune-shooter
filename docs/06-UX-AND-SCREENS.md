> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 06 — UX, Screens & Game States

## 6.1 Application state machine

An explicit `GameStateMachine` autoload with named states. No implicit scene transitions — every transition is a logged, testable event.

```
BOOT
 └→ TITLE
      ├→ CAMPAIGN_MAP
      │    ├→ PRE_LAUNCH (clauses + loadout) → CONTRACT_LOADING
      │    │     └→ CONTRACT_ACTIVE
      │    │          ├→ CONTRACT_PAUSED
      │    │          ├→ CONTRACT_FAILED → (retry / map)
      │    │          └→ CONTRACT_COMPLETE → REWARDS → [AD?] → LOG_ENTRY → MAP
      │    ├→ HANGAR
      │    ├→ ENCYCLOPEDIA
      │    └→ SETTINGS
      ├→ STORE
      └→ SETTINGS
```

**Ad placement is a single node in this graph**: after REWARDS, before LOG_ENTRY, and only when the completed contract was a miniboss or boss, the frequency cap has elapsed, and no purchase entitlement exists. One place in the codebase. Never on CONTRACT_FAILED.

**HANGAR is the second-most-important screen in the game** — 25–35% of session time. Budget for it like combat.

## 6.2 Screen inventory

| Screen | Job | Must-haves | Deliberately absent |
|---|---|---|---|
| **TITLE** | Get out of the way | Continue (big), New, Settings, Store. Live drifting-ship background | News feed, daily rewards |
| **CAMPAIGN_MAP** | Progress and choice | Five sectors as node clusters on a Lattice map, completion state, credits + element points, **side sectors visible and marked as expansions**, entry to Hangar | Energy costs, timers, "recommended power" |
| **PRE_LAUNCH** | **Contract review** | Clause slots (4) presented as contract addenda you accept for higher pay · attunement slots · loadout summary with swap access · enemy archetypes present with affinity icons · performance bonus list · projected payout · one-tap accept | Anything purchasable |
| **CONTRACT_ACTIVE** | The game | HUD per §6.3 | Everything else |
| **CONTRACT_PAUSED** | Stop, breathe | Resume, Restart, Settings, Quit. Instant, top-left, outside thumb zones | Ads, offers |
| **CONTRACT_FAILED** | Diagnose and retry | **Names the enemy that killed you** with its silhouette, threat class, affinity, and a one-tap link to its Bestiary entry. Lives remaining, Retry (default focus), Hangar, Map | Killcam replay · **any monetization whatsoever** |
| **REWARDS** | Feel good | Credits counting up, performance bonuses itemized, Clause multiplier shown, element point award if miniboss or boss | XP bars · loot boxes · spin wheels |
| **HANGAR** | Build the ship | §6.4 | Randomness |
| **ENCYCLOPEDIA** | Explain and plan | Bestiary / Lattice / Core (`05` §5.6) | Hidden locked content |
| **STORE** | Honest transactions | The pledge, SKU contents, prices, restore purchases | Currency packs, timers, bundles-of-bundles |
| **SETTINGS** | Control | `03` §3.2.6, audio sliders, data/privacy, account link, About | |

## 6.3 In-game HUD

Seven elements maximum. Portrait:

```
┌────────────────────────────────┐
│ [pause]  ──progress──  [minimap]│  ← 8%
│                                │
│           PLAYFIELD            │  ← 62%, no UI renders here
│                                │
│ ▓▓▓▓▓▓▓░░ 340/500   ●●○         │  ← 5%: hull bar + number, shield pips
│  ♥♥♥                           │
│  [ MOVEMENT ]  [  ROTATION  ]  │  ← 25%, visuals only on touch
│                          (cd)  │
└────────────────────────────────┘
```

- **Hull** — a bar, not a number. Amber <50%, red <25%.
- **Shields** — discrete pips. A second bar would be illegible.
- **Hull shows a number as well as a bar.** With raw HP (10 → 500) and raw enemy damage, the number is meaningful — the player can count how many hits they have left. This is a change from r2.
- **No crit indicator.** PRD is invisible by design; surfacing the counter would turn a felt rhythm into a spreadsheet to watch.
- **Status indicators cap at 2 per enemy** (`02` §2.2b) — blended rim light and particle for the two most recently applied. Mechanics run at full fidelity underneath.
- **Surge is telegraphed, not metered.** No progress bar. The three shots before a Surge brighten and rise in pitch — that's the entire UI for it, and it's enough (~0.4s of warning).
- **Attunement colours are on the projectiles themselves.** A 1-1-1 build reads as three colours in flight; a 3× PLASMA build reads as one. No HUD element needed.
- **Lives** — three small markers, bottom-left. Only visible at ≤2 remaining, so a clean run has less clutter.
- **Contract progress** — thin line, top, wave N of M.
- **Ability cooldown** — non-interactive ring, bottom-right of the rotation zone.
- **Boss HP** — replaces the progress line, segmented into 3.
- **Credits** — not on the combat HUD.

Respect display cutouts and gesture-nav insets. Test on a punch-hole device and an under-display-fingerprint device.

## 6.4 Hangar

Two panes, tab-switched.

**Pane A — Systems (stat tree)**
14 lines grouped Offence / Defence / Mobility. The **Guns** line shows the resulting damage split and slot count, not just a rank number — buying gun 3 is buying an attunement slot as much as it is buying damage. **Thrusters and Gyros show both the purchased ceiling and the current slider position**, since raising the ceiling and choosing where to fly inside it are separate decisions. Each shows current rank, pip track, next cost, and **the exact numeric delta** ("Attack Speed +0.25/sec → 4.75 shots/sec"). Nothing is level-locked — if you can afford it, you can buy it. No aggregate "power" score — it invites players to optimise a number instead of reading the actual effects.

**Pane B — Core (attunements, elements, loadout)**

Four sub-sections. This pane carries most of the game's depth and gets most of the UI budget.

1. **Attunement slots** — the ship schematic with 1–3 gun mounts, drawn to scale so the **centre mount is visibly larger** (52% of your damage; the UI must convey that without words). Tap a mount to assign any owned element or leave it null. Each mount shows its damage share, its element's on-hit effect, and that effect's current potency.

2. **Strong / weak readout** — beside the slots, each attuned element lists what it is **strong against** and **weak against**, plainly. Not a prediction against the next contract: contracts are replayable in any order, so a forward-looking number would be wrong the moment a player revisits Sector 2. Show the relationship; let the player apply it.

3. **Element levels** — nine tracks, current level, points available, spend and respec at 100 credits flat. **Before spending, show the full ripple:** "FORGE → level 2 raises LODESTONE to tier 2, SLAG to tier 2, the PLASMA+FORGE+VOLT triple to tier 1, and FORGE attunement potency 1.0 → 1.5."

4. **Node web + loadout** — a graph of all 129 nodes. Owned nodes show current tier and **which trigger layers are active at that tier** (crit only, or crit + Surge). Unowned and DLC nodes are visible and greyed with requirements stated. 1 active + 3 passive slots, filled by drag or tap, free and instant.

**Selecting a node plays a short looping clip of what it does.** Text cannot convey a spread pattern or a field shape, and with 129 nodes the player needs to evaluate options quickly. A 2-second silent loop against a dummy target is worth three sentences of description.

**Numeric tooltips on everything.** Every ability, node, status, and stat exposes exact figures on tap: damage to the primary target, damage to additional targets, radius, duration, slow percentage, stack cap, cooldown, and how each changes at the next tier. A build-crafting game whose numbers are hidden is a build-guessing game.

**The live preview is not optional.** A ship in the corner firing at a respawning dummy, showing the actual current build's visuals and numbers. It is how players feel their purchases, and it doubles as an excellent debug tool.

## 6.5 Onboarding

- **No tutorial pop-ups.** Sector 1 is the tutorial, taught by level design.
- **All tutorial hints are toggleable in Settings**, on by default, and can be re-enabled to replay them.
- The first time each control is needed, a 2-second ghost-hand animation over the relevant zone, once, dismissible. Three total (movement, rotation, recall). A fourth when the first active node is equipped.
- **The attunement system introduces itself at Miniboss 1**, when the first element point arrives. One Hangar highlight on the centre gun mount, once. The type chart is never explained in text — the PRE_LAUNCH matchup readout teaches it by showing the number change as you re-slot.
- Hangar's first visit: one highlight ring on the recommended first purchase. Once.
- Track completion per control so it never repeats across saves.

## 6.6 Accessibility

Built in from M1, not M5.

| Need | Features |
|---|---|
| Motor | Handedness swap; adjustable zone sizes; **Thruster and Gyro output sliders (0.5× to purchased maximum, which reaches 4× base)** so a player can fly as slow or fast as they like regardless of investment; Assist Aim; adjustable deadzones; **optional on-screen buttons** replacing tap-to-recall and tap-to-ability; no timed multi-touch |
| Visual | Three colour-blind palettes; shape-first enemy design; HUD scale; high-contrast mode; screen-shake and flash sliders |
| Cognitive | Freely switchable difficulty; no timers; full pause; optional damage numbers; Encyclopedia available mid-run |
| Auditory | Every audio cue has a visual equivalent. Nothing is audio-only |

**Photosensitivity:** this art style is a genuine seizure risk. Cap full-screen luminance changes, avoid >3Hz full-screen flashes, and surface the flash-reduction toggle **on the first-run screen**, not buried in settings. Test against WCAG 2.3.1 general flash thresholds. The optional no-bloom rendering path (`10` §10.1) is also the accessible path.

## 6.7 Localization readiness

Every user-facing string in `res://locale/en.csv` by key. Zero hardcoded strings. Auto-sizing containers assuming +40% for German. No text baked into textures. Locale-aware number formatting.

~4 hours at M1; ~3 weeks if retrofitted.
