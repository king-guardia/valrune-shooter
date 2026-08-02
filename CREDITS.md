# Credits and asset licenses

Maintained from commit one, **including CC0 assets.** When you are 200 assets deep and
need to know whether one laser sound requires attribution, this is a five-second lookup
instead of an afternoon.

## Rules

- **Log the asset at download time**, not when you use it.
- **Screenshot or PDF the license page at download time.** Sites change terms, assets get
  pulled, and "I think it was CC0" is not a defense. Store under `art-source/licenses/`.
- **Prototype and shipped assets live in separate trees.** `game/assets/proto/` and
  `game/assets/ship/`. CI fails the build if any scene references `proto/`.
- Source files (`.ai`, `.svg`) live in `art-source/`, outside the Godot project. Only
  exports go in `game/assets/`.

## Assets

None yet. Placeholder art arrives at M1 (Kenney.nl "Space Shooter Redux", CC0).

| File | Source | Author | License | URL | Downloaded | License copy |
|---|---|---|---|---|---|---|
| — | — | — | — | — | — | — |

## Tools and engine

| Thing | License | Notes |
|---|---|---|
| Godot Engine 4.7.1 | MIT | No revenue share, no seat fees |

## Fonts

Planned, not yet vendored. All OFL: **Chakra Petch** (display), **Barlow** or **Inter**
(body), **JetBrains Mono** or Barlow tabular figures (HUD numbers — tabular is mandatory,
proportional digits make counters jitter).

Deliberately avoiding **Orbitron** — it is on approximately every space game ever made.
