> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 03 — Core Gameplay

## 3.1 The core loop

```
Campaign map → select contract → PRE-LAUNCH (clauses, loadout) → STAGE (~2:30)
     ↑                                                          ↓
     └──────── Hangar (spend credits, swap loadout) ←── results ┘

Contract 3 of each sector = MINIBOSS → 1 element point → ad
Contract 6 of each sector = BOSS     → 1 element point → ad
```

## 3.2 Controls — full specification

The riskiest and most original part of the design. Build it first (M0). All values are starting points to tune by feel.

### 3.2.1 Zone anchoring

The lower ~38% of the screen splits into two **input zones**. The first touch landing in a zone claims it until released; further touches in an owned zone are ignored. A touch that drags across the divider keeps its original zone. Zones run full half-width each, from 62% height to the bottom edge. Touches above 62% do nothing, so panicked taps don't break your grip.

```
┌──────────────────────────────┐
│ [pause]           [minimap]  │
│                              │
│          PLAYFIELD           │
│                              │
├──────────────┬───────────────┤  y = 62%
│  MOVEMENT    │   ROTATION    │
│   (drag)     │    (drag)     │
│ tap = recall │  tap = ability│
└──────────────┴───────────────┘
```

**Handedness swap** in settings mirrors the zones. Default: movement left, rotation right.

### 3.2.2 Movement — left zone, drag

A floating stick anchored where you touch. **Direction is screen-relative, not ship-relative** — up is up regardless of which way the ship points. Space has no direction, so the phone's frame is the reference.

| Parameter | Default |
|---|---|
| Stick radius | 15% of screen width |
| Deadzone | 8% of radius |
| Response curve | Squared (fine control near centre) |
| Model | Direct velocity, not force |
| Max speed | 520 u/s Throat, 440 u/s Field |
| Inertia | None in Throat (exact dodging); 0.18s settle in Field (drift feels good) |

Godot 4.7 ships a production-ready `VirtualJoystick` node — use it as the base, wrapped in a `MovementZone` abstraction.

### 3.2.3 Rotation — right zone, drag, **rate-based**

Touch anchors a horizontal soft bar. Drag left → counter-clockwise, drag right → clockwise. **Distance from the anchor sets angular velocity**, not angular position — a small strip cannot hold 360° of position without absurd sensitivity.

| Parameter | Default |
|---|---|
| Max angular velocity | 300°/s (upgradeable to ~420°/s) |
| Response curve | Squared |
| Release behaviour | **Decays to zero over 0.15s.** Never holds the last rate |
| Deadzone | 4% of bar half-width |

**Soft anchor at north.** Within ±12° of screen-up, angular velocity is damped by up to 60% and a short haptic pulse fires. It makes north easy to land on without fighting the player. **Disabled above 200°/s** — a deliberate fast spin blows straight through it. Toggleable in settings.

**Rotation rate is a stat upgrade.** It makes the *control* feel better rather than the numbers get bigger, and players notice that disproportionately.

### 3.2.4 Taps

**Tap detection:** released within 150ms AND moved under 8dp AND the zone was idle ≥120ms before the touch. The idle requirement stops a fumbled re-grip registering as a tap.

**Movement suppression window:** input is suppressed until the touch crosses **6dp** of movement, then applies. Crossed almost instantly in real use, so latency is imperceptible, but a genuine tap never nudges the ship or blips the rotation.

| Tap | Effect |
|---|---|
| **Left zone** | **Recall** — ship travels to the ready line (Throat: bottom-centre; Field: arena origin) and heading snaps to north |
| **Right zone** | **Activate ability** |

**Recall** is a fast dash (0.35s) with i-frames for the first 0.15s, not a teleport. Any movement input cancels it instantly. Cooldown 6s, reducible to 3s. Terminal rank of the Recall Systems line upgrades it to **Blink** (instant, 0.1s). Blink targets along current heading at fixed distance — never a tap-on-playfield, which would make tap/drag disambiguation load-bearing under pressure.

**Recall is unavailable while holding the movement stick** — you must release to tap. Consistent with "movement overrides recall," and confirmed as intended.

**The ability button doesn't exist as a button**, so its cooldown needs a display: a non-interactive ring in the bottom-right corner of the rotation zone, pulsing in the node's colour when ready.

### 3.2.5 Auto-fire

Always on. Fire rate is a core stat: **2.0/sec at start, 8.0/sec at max** (`04` §4.3).

**Gun count is an attunement decision, not just damage** (`04` §4.4). One gun fires a single stream; two guns **helix** around a shared axis; three guns put a heavy centre stream between two lighter wing streams. Each gun is tinted by its slotted element, so a mixed build reads as multiple colours in flight and a committed build reads as one.

**The gun does not fire when no valid target is on screen.** Saves battery, cuts audio fatigue, and reads as intentional.

### 3.2.6 Settings

Handedness swap · movement stick size · rotation sensitivity · soft anchor on/off · aim assist on/off · Assist Aim (auto-track nearest threat, accessibility) · optional on-screen buttons for recall/ability · haptics · screen-shake 0–100% · flash reduction · colour-blind palettes.

## 3.3 Camera

| Mode | Behaviour |
|---|---|
| Throat | Fixed. Playfield = one screen. ≤3% parallax drift on background layers keyed to ship X |
| Field | Follows ship, 0.12s lag, 18% look-ahead in the velocity direction. Fixed zoom. Soft-clamped so the arena boundary never passes screen centre |

Screen shake is trauma-based (accumulating, decaying), hard-capped lower than feels right in isolation — it compounds badly at 60 enemies — and fully disableable.

## 3.4 Physics

`CharacterBody2D` with explicit velocity. **Never rigid-body physics** — it makes shooters mushy and breaks determinism.

## 3.5 Contract types and edge rules

### THROAT (vertical)

You're inside a wormhole in transit; the Hollow pour in from the far end.

- Playfield is exactly one screen, portrait.
- **Horizontal edges WRAP.** Fly off the left, appear on the right. A throat is a cylinder, so this is what the geometry actually does. It also removes the worst feeling in vertical shooters — getting cornered.
- **Vertical is clamped** to the bottom 62%, with a soft push-back and a visible flux-boundary shader band above.
- **Enemies reaching the bottom wrap to the top** after a 2s re-entry delay with a telegraph. Nothing leaks, no second fail state to explain — they just come back and clutter your screen.
- Player and enemy bullets wrap horizontally; despawn vertically.

Implement wrap as a `WrapAround2D` component in M0 — retrofitting it is painful. Entities near a boundary render twice (at x and x±width) for a seamless transition.

### FIELD (arena)

- Circular arena, radius 3.5× the screen's short dimension (~9× screen area).
- **Hard boundary, no wrap** — a disc can't wrap cleanly. Elastic push-back scaling with penetration depth, visible hex shimmer, HUD arrow. **No damage** — wall-killing encourages timid play.
- Enemies bounded identically; ranged enemies won't fire from outside.
- **Minimap:** top-right, circular, ~22% screen width. Arena bounds, ship + facing cone, enemies colour-coded by threat class, objectives, pickups. Never covers the top 15% where boss telegraphs appear.
- **Off-screen chevrons** for threats within 1.5s of mattering. Cap 6, prioritized by time-to-impact.

### Mix

~65% Throat, 35% Field. Throat is the identity; Field is the change of pace. Bosses alternate, starting Throat.

## 3.6 Enemy roster

All 8 distinguishable by **silhouette alone at 40px**, before colour.

| # | Name | Behaviour | Intro | Counter |
|---|---|---|---|---|
| 1 | **Mote** | Straight drift. The chaff | W1 | Anything |
| 2 | **Lash** | Sine weaver, faster, harder to lead | W1 | Spread, prediction |
| 3 | **Spike** | 0.8s lock-on telegraph, then rams at 3× speed | W1 | Sidestep; punishes standing still |
| 4 | **Husk** | Tanky; splits into 2 Motes at fixed angles | W2 | Area damage, positioning |
| 5 | **Anchor** | Stationary turret. **First enemy that shoots** | W3 | Cover, kill priority |
| 6 | **Warden** | Slow; shield bubble grants nearby enemies immunity | W3 | Kill first; pierce |
| 7 | **Seeder** | Drops proximity mines on a fixed pattern | W4 | Route control |
| 8 | **Choir** | Carrier; spawns 1 Mote every 2.5s | W4 | DPS check |

**Enemies shoot only from Sector 3.** The Hollow start as mindless rammers and *learn to shoot from you* after the second boss — a difficulty ramp that doubles as a story beat for free.

**Threat colour coding:** rammers magenta, shooters orange, support violet, boss white-hot.

## 3.7 Boss and miniboss rules

**Minibosses** (contract 3 of each sector): single phase, an existing enemy's behaviour with inflated stats plus one new attack, boss art reused at smaller scale. ~3 days each to build. ~3 minutes to fight.

**Bosses** (contract 6): exactly 3 phases, segmented HP bar, ~5 minutes. Every attack telegraphed ≥0.6s with distinct colour and sound. Destructible sub-parts where possible. No unavoidable attacks. No healing. No boss kills a full-hull player in under 3 hits. Each teaches one thing (`05` §5.4).

One checkpoint at the phase-1 transition.

## 3.8 Combat feel ("juice")

Not polish — this is the product. Budget a full milestone (M4).

- Hit-stop: 40ms on kills, 15ms on hits, scaled by damage, **capped per second** or it reads as lag.
- Muzzle flash and heat particle per shot.
- Impact sparks along the surface normal, coloured by damage element.
- Kill: bright flash, outward ring, debris inheriting enemy velocity.
- **Crits must read instantly**: fatter tracer, distinct impact sound, extra hit-stop, brief bloom on the impact. At 30% chance and 8 shots/sec they land over twice a second, so they are emphasis rather than ceremony — but a 3-gun volley crits as a unit for 2.1n × 2.5 = **5.25n in one frame**, which is a real spike worth selling.
- **Crit cross-propagation must be visible**: when a mixed-attunement crit lands, the player should see each element's effect chaining across the same targets. This is the moment a mixed build justifies itself, and if it reads as noise the whole system is invisible.
- **Element matchups must be legible mid-fight**: a 1.30× hit shows a brighter impact and a rising pitch; a 0.75× hit shows a dull thud and a muted spark. The player should learn matchups by feel, never from a menu.
- **Surge is the ceremony**: three shots of rising brightness and pitch, then a much larger impact with shake. ~0.4s of warning to point the ship at something worth hitting.
- Damage numbers **off by default**.
- Hull damage: 0.15s red vignette, haptic, distinct low sound. Never obscures the playfield.
- Low hull (<25%): persistent pulsing edge vignette + audio heartbeat, perceptible peripherally.
- Boss phase transition: 1s slow-motion, camera punch, audio dropout then swell.
- Every player action gets feedback within 50ms.

## 3.9 Dev-only metrics

Time-to-first-death per contract · deaths per contract · contract duration · screen-quadrant occupancy · recall uses · rotation input rate · crit rate vs. nominal (PRD sanity) · effective element multiplier distribution per contract · attunement configs actually used per sector · **which enemies do the killing** (the single most useful number you will have).
