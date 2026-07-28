# valrune-invaders — planning doc

## Vision

What is this game for?

- [ ] Internal team fun / morale
- [ ] Marketing / lead gen (embed on valrune site)
- [ ] Event booth / conference demo
- [ ] Other: ___

**One-liner:** *(e.g. "Classic invaders with Valrune-themed enemies and power-ups.")*

---

## Core gameplay

| Decision | Options | Choice |
|----------|---------|--------|
| Genre baseline | Space Invaders clone, Galaga-style, custom | |
| Player | Ship at bottom, classic movement | |
| Enemies | Rows/columns, dive attacks?, bosses? | |
| Power-ups | None, shields, rapid fire, Valrune "data" pickups | |
| Win/lose | Lives, score chase, levels, infinite | |
| Controls | Keyboard, touch, both | |

---

## Platform & stack

| Decision | Options | Choice |
|----------|---------|--------|
| Target platform | Web browser, mobile app, desktop, multi | |
| Framework | Vanilla canvas, Phaser, PixiJS, Godot, Unity, other | |
| Language | TypeScript, JavaScript, GDScript, C# | |
| Deploy | Static host (Vercel/Netlify/Firebase), app stores | |

**Recommendation to discuss:** Web-first (TypeScript + canvas or Phaser) — fast to ship, easy to embed on a Valrune landing page, no app store friction.

---

## Valrune branding

- Logo / wordmark usage
- Color palette (from Valrune brand guidelines if any)
- Enemy themes (e.g. "data bugs", "pipeline gremlins", generic aliens)
- Copy / Easter eggs (taglines, inside jokes for DataOps crowd)
- Sound: retro bleeps vs. licensed music

---

## Scope — MVP vs. later

### MVP (ship first)

- [ ] Player ship + movement + shoot
- [ ] Enemy grid + basic AI
- [ ] Collision + score + lives
- [ ] Start / game over screens
- [ ] Valrune logo on title screen

### v1.1+

- [ ] High score (local or leaderboard)
- [ ] Mobile touch controls
- [ ] Power-ups / levels
- [ ] Sound + music
- [ ] Share score / CTA to Valrune site

---

## Open questions

1. Who is the primary audience?
2. Hard deadline or event date?
3. Existing Valrune brand assets (SVG, colors, fonts)?
4. Need analytics (plays, scores, CTA clicks)?
5. Solo dev or will others contribute?

---

## Next steps

1. Lock platform + stack
2. Sketch wireframes (title, play, game over)
3. Scaffold src/ with chosen toolchain
4. Placeholder art → replace with branded assets
5. Playtest loop → polish → deploy
