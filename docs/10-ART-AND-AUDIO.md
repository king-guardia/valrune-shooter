> **STATUS: v0 — pending revision.** Written before the ability/status spreadsheets were reconciled into the plan. Several decisions in this document have since been **reversed**. Where this file conflicts with [`docs/decisions/`](decisions/), **the decision log wins.** Full revision happens in Phase 7.

# 10 — Art Direction, Production Pipeline & Audio

## 10.1 The single insight that solves your art problem

**The neon look is ~80% rendering, not drawing skill.**

A flat, ugly, six-sided polygon outlined in `#00E5FF` at 3px, rendered on a near-black background with HDR emission and a well-tuned bloom pass, looks *expensive*. The same shape without bloom looks like a school project. The glow is a shader, not an art skill.

This means your actual art job is much smaller than you think: **produce clean, readable silhouettes in flat colours, and let the engine do the beauty.** That is a job a non-designer can absolutely do with Illustrator and discipline.

Practical consequence: **do not bake glow into your sprites.** Illustrator's Outer Glow and Gaussian Blur are the wrong tool here — baked glow can't respond to gameplay (damage flashes, element recolouring, boss phases), it bloats texture memory, and it double-glows when the engine's bloom hits it. Export crisp flat art; glow it in Godot.

### The Godot setup (High preset)

```
WorldEnvironment
  glow_enabled = true   # OFF on Flat, per-object halos on Medium
  glow_intensity = 1.1
  glow_bloom = 0.35
  glow_hdr_threshold = 1.0
  glow_blend_mode = Additive
  glow_levels: enable 2, 3, 4 (tight + wide falloff)
  background = flat #04060B

CanvasItemMaterial on ships/bullets
  blend_mode = Add (for energy elements) or Mix (for hulls)
  Colors specified with values ABOVE 1.0  →  e.g. Color(0.0, 3.4, 4.2)
```

**HDR values above 1.0 are the trick.** `(0, 3.4, 4.2)` reads as white-hot cyan with a wide halo; `(0, 0.85, 1.0)` is the same hue, dim. You control "how energetic does this feel" with a single float. Wire it to gameplay: Surge shots at 5.0, crits at 3.5, normal at 2.0, depleted shields at 0.6.

**On the Flat path the same float drives stroke width and colour value instead of emission**, so the energy hierarchy survives with no post-processing. Build the mapping once, in one place, and both paths get it.

## 10.2 Style definition

**Reference vocabulary (for yourself, for Cursor, and for any artist you hire):**

> Vector-neon. Hard-edged geometric silhouettes with 2–4px emissive outlines and dark translucent fills, on near-black. Everything glows. Nothing is textured. Motion is expressed through trails, streaks, and afterimages rather than through animation frames. Reads like an oscilloscope, a vector arcade cabinet, or an air-traffic display that got beautiful.

**Radiant Defense is the closest colour reference** — flat saturated hues on near-black, every unit and tower type instantly separable by colour alone, no texture, no gradients, and readable at a glance with dozens of units on screen. That is exactly the problem this game has, solved. Study how it assigns colour by *function* rather than by aesthetics.

**Style anchors worth studying** (study the *principles*, don't copy assets or designs): Geometry Wars (particle density, colour-as-meaning), Resogun (voxel/neon energy contrast), Rez and Tempest (wireframe minimalism), Nova Drift (**the single closest reference for this project** — a build-crafting space shooter where every module visibly changes the ship and its projectiles; study how it makes 100 modules visually legible), Ex-Zodiac (low-poly retro space), Vector-era arcade (Asteroids, Battlezone) for the pure line aesthetic.

**Complexity target:** you said "more complex than total simplicity," and that's the right call. Aim for **8–16 line segments per ship silhouette** — enough to have character, few enough to read at 48px and to draw a variant of in 20 minutes.

## 10.3 Colour system

Colour carries **gameplay meaning** (pillar P3). Assign roles first, aesthetics second.

| Role | Colour | Hex | Rule |
|---|---|---|---|
| Background | Near-black blue | `#04060B` | Nothing else is this dark |
| Player ship | Cyan-white | `#7FFFF5` | **Reserved.** Nothing else in the game is cyan-white |
| Player projectiles | Cyan | `#00E5FF` | |
| Enemy: rammer | Magenta | `#FF2D78` | |
| Enemy: shooter | Orange | `#FF8A1F` | |
| Enemy: support/shield | Violet | `#B44BFF` | |
| Enemy projectiles | Hot red | `#FF3B3B` | **Reserved.** Only things that damage you are this red |
| Pickups | Green-gold | `#9BFF57` | |
| Boss | White-hot | `#FFFFFF` @ HDR 5.0 | |
| Surge | White | `#FFFFFF` @ HDR 5.0 | Shared with boss; contexts never overlap |
| UI chrome | Slate | `#1A2230` / `#5C6B80` | |

**Element colours** (used for procs, trails, ability VFX — layered *on top of* the role colours, never replacing enemy-projectile red):

| Element | Primary | Secondary |
|---|---|---|
| PLASMA | `#FF6A1F` | `#FFD24A` |
| CRYO | `#2E9BFF` | `#BFEFFF` |
| FORGE | `#FFB03A` | `#B07030` |
| VOLT | `#5CFFE0` | `#E6FFFA` |
| CHRONO (DLC) | `#C8A2FF` | `#F0E6FF` |
| GAMMA (DLC) | `#FFFFFF` | spectral gradient |
| VOID (DLC) | `#6B2FFF` | `#1A0033` |

**Two hard rules:**
1. **Red belongs to things that hurt you.** No decorative red. No red UI. This single rule does more for readability than any amount of polish.
2. **Colour-blind safety:** magenta/orange/violet enemies must also differ in *silhouette*. Run every enemy through a deuteranopia simulator and confirm you can still tell them apart. Ship three alternate palettes.

## 10.4 The production pipeline for a non-designer

This is the part you asked for most directly. Five techniques, in order of value.

### Technique 1 — Modular ship kits (do this first)

Don't draw 30 ships. Draw **parts** and combine them in-engine. `enemies.json` references parts by id (`08` §8.6), so enemy visuals are composed from data.

```
6 hulls  ×  6 wing sets  ×  5 engines  ×  4 accent marks  =  720 combinations
```

Each part is 10–20 minutes in Illustrator. That's roughly **7 hours of drawing for your entire ship roster**, including enemies, bosses, and DLC. Store parts as separate sprites, compose as child nodes with defined anchor points, tint each layer independently.

Two payoffs beyond speed:
- The player's ship visibly changes with element tiers — engine wash shifts to PLASMA orange, wing accents grow FORGE plating. The best "my build matters" feedback available.
- **Skins are nearly free** — a skin is a parts + palette swap. One constraint: every skin must keep one identity element (the cyan-white core) so the player ship stays instantly findable at peak density.

Define anchor points as a convention: every hull has `wing_mount_l`, `wing_mount_r`, `engine_mount`, `accent_mount` at fixed relative coordinates. Then composition is data (`08` §8.4 gets a `parts` field), not art.

### Technique 2 — Constrained construction in Illustrator

Constraints substitute for taste. Adopt all of these:

- **Grid:** 24×24px, snap everything. Nothing off-grid.
- **Strokes only, or strokes with 15%-opacity dark fills.** No gradients, no textures, no shadows.
- **Two stroke weights:** 3px (primary silhouette) and 1.5px (interior detail). Never a third.
- **Angles:** multiples of 15°. Curves only via `Effect > Warp` on straight segments.
- **Symmetry:** draw the left half, reflect it. Ships are symmetrical; enemies can break symmetry to signal "wrong."
- **Colour:** one hue per object, from §10.3. Value variation only.
- **Silhouette test:** fill the shape solid black at 48×48px. If you can't identify it, redesign it. Do this *before* adding detail — most amateur game art fails here.

Illustrator workflow: build on a 512×512 artboard → `Object > Expand Appearance` → export as **SVG** → Godot imports SVG natively with a scale setting, so you get resolution independence for free across the Android device zoo. (Fall back to 4× PNG export for anything with effects SVG can't carry.)

### Technique 3 — Procedural / generative art in-engine

A large fraction of this style's visual appeal can be *code*, which plays to your strengths rather than your weaknesses:

- **Trails:** `Line2D` with a gradient and width curve, following the ship. Two lines of code, enormous visual payoff.
- **Bullets:** capsules or elongated quads with additive material. Zero drawn art.
- **Explosions:** `GPUParticles2D` with a ring emission shape + a expanding `Line2D` shockwave. Zero drawn art.
- **Backgrounds:** parallax star layers generated at runtime; nebulae as a fragment shader with layered noise; the wormhole throat as an animated tunnel shader. **Do not draw backgrounds.**
- **Boss parts:** procedurally arranged rings of a single drawn segment, rotating at different rates. Looks intricate, is one sprite.
- **Damage:** modulate the emission float and add a vertex-displacement shader. No damage-state sprites needed.

Realistically, **60% of what the player sees can be generated.** Budget your drawing effort for ships and UI only.

### Technique 4 — Stock and CC0 as a base layer

- **Placeholder immediately:** Kenney.nl "Space Shooter Redux" (CC0, no attribution required) — use it from day one so you can build gameplay for months without touching Illustrator. Ship-blocking on art is the #1 way solo projects die.
- **Adobe Stock:** you have it. Search "vector spaceship top view outline," "HUD elements vector," "neon geometric icon set." Buy *shape libraries*, not finished art, and rebuild them into your grid/stroke system. **Check the licence:** Adobe Stock's standard licence generally permits use in applications, but there are restrictions around redistributable templates and some assets — read the specific licence for each asset and keep receipts.
- **Also worth browsing:** Quaternius (CC0), Game-Icons.net (CC BY 3.0 — 4,000+ single-colour icons, ideal for upgrade/ability icons, recolours to your palette trivially), OpenGameArt (mixed licences, read carefully), itch.io asset packs.
- **AI image generation:** poor fit for this specific need. It produces raster images with inconsistent style and no clean vectors, and it can't hold to a 24px grid or a fixed stroke weight. It *is* useful for mood boards and for the static story panels in `05` §5.5. Check your store's policy on AI-generated assets and disclose where required.

### Technique 5 — Hire narrowly, once

If after M2 the art still isn't working, spend $400–1,200 on a freelancer (Fiverr/Upwork/Reddit r/gameDevClassifieds) for a **style bible + the 6 hull parts**, then produce everything else yourself against it. This is dramatically cheaper and more effective than commissioning a full asset set, and it gives you a reference to match.

## 10.5 Typography

| Use | Font | Why |
|---|---|---|
| Display / titles / boss names | **Chakra Petch** (OFL) | Angular, sci-fi, not overused |
| Alt display | **Michroma** or **Saira Stencil One** (OFL) | |
| UI body & descriptions | **Barlow** or **Inter** (OFL) | Actually readable at 14sp on a phone. Do not use a display font for body text |
| Numbers / HUD / stats | **JetBrains Mono** or **Barlow tabular figures** | **Tabular figures are mandatory** — proportional digits make counters jitter |

Avoid **Orbitron**. It's free and fits, and it is on approximately every space game ever made; it will make your game look generic instantly.

Sizes: body 14sp min, tappable labels 16sp, numbers on the HUD 18sp+. Test at 1.3× system font scale — Android users do use it.

## 10.6 Branding

- **Wordmark:** the game name in Chakra Petch, letterspaced +8%, with a single horizontal "signal line" cutting through it — echoing the Heading Ring. One idea, executed cleanly, beats five ideas.
- **Icon:** must read at 48px in a phone drawer. That means: the player ship silhouette, cyan, on near-black, with a single glow. **Not** the wordmark. Not a busy scene. Test it at 48px against your actual home screen before committing.
- **Store screenshots:** show *gameplay at peak density*, plus one Hangar screenshot (the build screen is your differentiator — lead with it, not with a generic explosion).

## 10.7 Audio — full inventory

Audio is half of "feel," and in a game whose entire identity is *which combination did I build*, it is half of build identity too. A player must be able to tell a CINDERSTORM proc from a LODESTONE rail **with the phone in their pocket.**

**Priority order:** (1) shooting feels good, (2) hits feel good, (3) kills feel good, (4) danger is audible, (5) build identity is audible, (6) music exists.

### 10.7.1 The count

| Group | Assets | Notes |
|---|---|---|
| Base weapon | 4 | Fire (×3 pitch variants for round-robin), dry-fire/idle-out |
| Crit | 2 | Fire snap, impact |
| **Surge** | 4 | Three rising telegraph tones + the heavy resolve |
| **Element attunements** | 9 | One signature layer per element |
| **Element nodes** | ~40 | 36 dual signatures + triple overlays |
| Enemy impacts | 4 | Light / medium / heavy / shielded (not per enemy — by mass class) |
| Enemy deaths | 5 | Light / medium / heavy / split (Husk) / mine detonation |
| Boss & miniboss | 12 | Telegraph ×3 (colour-coded to attack class), sub-part destruction, phase transition ×2, death, spawn roar, shield break, charge loop, impact-on-boss |
| Player hit | 5 | Hull hit, shield absorb, shield break, shield recharge complete, contact/ram |
| Player state | 4 | Low-hull heartbeat loop, death, respawn, Discharge cycle on/off |
| Movement | 5 | Engine loop (throttle-modulated), rotation thruster puff, Recall dash, Blink, boundary push-back |
| Pickups | 4 | Shield Pod, Coolant, Repair Cell, Salvage |
| UI | 12 | Tap, back, purchase, element point spend, node equip/unequip, clause equip, tier-up flourish, error, screen transition, challenge earned, credits count-up loop, contract-complete sting |
| Story panels | 2 | Terminal type-on loop, panel advance |
| **Total SFX** | **≈ 98** | |
| Music | 10 | See §10.7.3 |

Roughly **110 audio assets** for v1.0. That sounds like a lot; most are 200ms, and Sonniss plus a good pack covers 70% of them with light editing.

### 10.7.2 Node audio — the 1:1 question

You asked whether every combination needs unique sound. **Yes for the proc, layered for the impact.** The compromise that gets you distinctness without 42 bespoke sounds:

Each node gets a **signature layer** — one short, characteristic sound that is unique to it and never used elsewhere. Impacts are then built as `shared_impact_base + node_signature_layer`, mixed at runtime. That's 14 unique signature layers plus 4 shared bases, and the result reads as 14 distinct sounds while costing a third of the work.

| Node | Assets | Contents |
|---|---|---|
| 4 singles | 8 | Signature layer + proc |
| 6 duals | 15 | Signature + proc; SQUALL (active) also needs a cast and a 8s loop |
| 4 triples | 12 | Signature + proc; the 3 actives need cast, loop/sustain, and resolve |
| Shared impact bases | 4 | Light / medium / heavy / pierce |
| **Total** | **~39** | |

**Give each element a sonic palette** so combinations are legible by ear before the player learns them:

| Element | Palette |
|---|---|
| PLASMA | Crackle, gas ignition, low roar |
| CRYO | Glass, ice fracture, filtered noise sweep |
| FORGE | Metal impact, anvil, deep resonant clang |
| VOLT | Electrical arc, static snap, whipcrack |
| CAUSTIC | Hiss, fizz, wet corrosion |
| CHRONO | Pitch-bend, reversed tail, tape warble |
| GAMMA | Pure tone, Geiger tick, sustained ring |
| VOID | Sub-bass swell, inward whoosh, silence |
| AETHER | Breath, hollow reverb, detuned shimmer |

Then SCALD (PLASMA+CRYO) is literally *ignition into a glass shatter*. The combination sounds like its constituents, which means players decode new nodes without being taught. This is the single highest-leverage decision in the audio design.

**CI enforces uniqueness:** no two nodes may share an `sfx` id (`08` §8.3).

### 10.7.3 Music

Dark synthwave / industrial techno. **10 tracks, loop-based:**

| Track | Use |
|---|---|
| Menu / title | |
| Hangar | Calmer, harmonically related to the combat tracks |
| Sector 1–2 combat | Shared |
| Sector 3 combat | |
| Sector 4–5 combat | Shared |
| Miniboss | Shorter loop, higher intensity |
| Boss | Layered so intensity can ramp per phase |
| Final boss | |
| Victory sting | ~4s, non-looping |
| Defeat sting | ~2s, non-looping |

**Layered stems where affordable** — a base loop plus an intensity layer that fades in above a threshold of on-screen enemies. Doubles the perceived variety for one extra stem per track.

Licensed pack ($30–150) or commission (~$150/track). Commission the boss and final-boss tracks if you commission anything; those are the two the player will remember.

### 10.7.4 Mixing rules

- **Pitch-vary ±4% per shot** with round-robin variants, or auto-fire at 8/sec becomes a fatiguing buzz. This is the most important line in this section.
- **Voice limiting with priority tiers.** At Sector 5, nine statuses and dozens of node signatures can stack a dozen distinct sounds per second — the mix collapses into noise and the audio budget blows. Cap concurrent instances per sound id (hits 6, deaths 4, node signatures 3) and assign priority tiers:

| Priority | Sounds | Behaviour |
|---|---|---|
| 1 | Danger cues — boss telegraph, low hull, rammer lock | Never ducked, never culled |
| 2 | Player feedback — crit, Surge, hull hit, shield break | Ducks tier 3 |
| 3 | Node signatures and status applications | Culled first under load |
| 4 | Ambient enemy and impact chatter | Culled aggressively |

This matters more than it looks: the late game is where the audio design either sells the build or becomes a wall of hiss.
- **Duck music −4dB** under boss telegraphs, low-hull warnings, and Surge.
- **Danger cues sit in a reserved frequency band** and are never masked. Boss telegraph, low hull, and rammer lock-on must cut through everything.
- **Separate volume sliders** for music / SFX / UI. Honour device silent mode. Pause audio on app background — people play this on a bus.
- **OGG Vorbis for music, WAV for short SFX.** Avoid MP3 — its encoder gap breaks seamless loops.

## 10.8 Asset pipeline hygiene

- Source files (`.ai`, `.svg`) in `art-source/`, **outside** the Godot project. Only exports go in `game/art/`.
- Git LFS for PNG and audio from the first commit.
- Consistent naming: `ship_hull_03.svg`, `enemy_spike.svg`, `vfx_ring_expand.png`.
- Texture atlases for anything shipped as PNG (Godot's built-in atlas import).
- One `theme.tres` for all UI. Every colour references a named theme constant, never a literal — this is how you ship three colour-blind palettes for free.
- **Every node's VFX scene and SFX id must be unique.** CI warns on duplicates (`08` §8.3). If PLASMA+VOLT looks like PLASMA with a blue tint, the combo doesn't exist emotionally. Chrono Trigger animated ~30 techs individually because that's where the payoff lives.

## 10.9 Asset sources

The split that matters: **buy audio, icons, and fonts; build ships and enemies yourself.** Style coherence is the differentiator, and a purchased spaceship pack will never match the modular kit. Buy *shape libraries and building blocks*, then rebuild them onto your 24px grid and stroke system.

### Placeholder art — use from day one

| Source | Licence | Use |
|---|---|---|
| **Kenney.nl** | CC0, no attribution | "Space Shooter Redux" is nearly purpose-built for your prototype. Consistent style across packs. Use through M2 so art never blocks gameplay |
| Quaternius | CC0 | Reference silhouettes |

### Icons and shape libraries — high value

| Source | Licence | Use |
|---|---|---|
| **Game-Icons.net** | CC-BY 3.0 | ~4,000 monochrome vector icons. **Bookmark this hardest.** You need icons for 14 stat lines, 129 nodes, 9 elements, 9 statuses, 9 Clauses, and all UI. Monochrome SVG recolours to your palette with zero adaptation |
| SVG Repo | Mixed, much CC0 | ~500k vectors, filterable by licence |
| Lucide / Phosphor / Iconoir | MIT / ISC | Interface chrome |
| Vecteezy / Freepik | Attribution on free tier | Large vector libraries; licences are fussier than they look |
| **Adobe Stock** | Standard licence | You have it. Best for vector shape kits and HUD element libraries. **Restrictions exist around redistributable templates — read the per-item licence and keep receipts** |

### Audio

| Source | Licence | Use |
|---|---|---|
| **Sonniss GDC bundle** | Royalty-free, game use | Annual free giveaway, tens of GB of professional SFX. Absurd value |
| **Freesound** | Mixed CC0 / CC-BY | Community DB; filter by licence |
| Kenney audio | CC0 | Consistent naming, good for UI and prototype |
| Pixabay audio | CC0-equivalent | Good search |
| Zapsplat | Free w/ credit, or paid | Large library |
| **jsfxr / Bfxr / ChipTone** | Yours outright | *Generate* retro SFX in-browser |
| Uppbeat / Epidemic | Subscription | Music |

**For the Surge telegraph and the 14 node signature layers:** generate a base in jsfxr, layer a Sonniss impact underneath. Cheapest path to distinctive audio.

### Fonts

**Google Fonts** (OFL) covers doc §10.5. Also **Fontesk**, **Velvetyne** (experimental geometric display faces nobody else is using), **Collletttivo**.

### Shaders

**Godot Shaders** (godotshaders.com) — community shaders, mostly MIT/CC0. Glow variants, distortion, CRT. Directly usable.

**Shadertoy** — technique reference only. **Licences vary per shader and many are all-rights-reserved by default.** Learn the technique, rewrite it.

### Marketplaces

**itch.io asset section** — the most stylistically interesting work, often pay-what-you-want, creator-set licences that vary wildly. **OpenGameArt** — huge volume, inconsistent quality, mixed licences (CC0, CC-BY, GPL, OGA-BY). Good when nothing else has the thing you need.

### Licence hygiene — from the first download

**`CREDITS.md` in the repo root, maintained from commit one**, even for CC0. Filename, source, author, licence, URL, date. When you're 200 assets deep and need to know whether one laser sound requires attribution, that's a five-second lookup instead of an afternoon.

**Separate prototype from ship-approved.** `assets/proto/` and `assets/ship/` as distinct trees. The classic failure is shipping with a non-commercial loop or an uncredited CC-BY sprite buried under 18 months of iteration.

**CI check:** fail the build if any scene references a path under `assets/proto/`. One hour to write, and it makes the whole category of licence mistakes structurally impossible.

**Screenshot or PDF the licence page at download time.** Sites change terms, assets get pulled, and "I think it was CC0" is not a defence.
