# Changelog — Hot Potato: Bomb Toss

Build log for the game implemented from `hot-potato-bomb-toss-spec.md`.
Newest entries at the top.

---

## 2026-08-24 — v1.0 — Initial build

Full implementation of the spec: all Must-Have, all Should-Have, and all
Nice-to-Have items except match-history persistence (`localStorage`), which the
spec itself lists as optional and not needed for a live party game.

**Deliverable:** `index.html` — one file, 1937 lines, ~74 KB, zero dependencies,
zero build step. Opens directly in a browser (no external resources of any kind:
no CDN, no fonts, no images, no audio files).

### Architecture decisions

| Decision | Reasoning |
|---|---|
| Single HTML file, vanilla JS + Canvas2D | Spec's deployment requirement. No engine needed — everything on screen is a primitive shape. |
| Fixed 1280×720 internal canvas, CSS-scaled to fit | Guarantees identical layout everywhere; letterboxing is free via `object-fit`-style scaling in `fit()`. All coordinates are authored against one resolution. |
| Delta-time everywhere, `dt` clamped to 50 ms | Frame-rate independence per spec. The clamp stops a tab-switch from instantly detonating the bomb. |
| **All audio synthesised at runtime with WebAudio** | Spec asks for ~15 distinct sounds plus music. Shipping sample files would break the single-file rule, so every effect is built from oscillators + filtered noise (`A.tone` / `A.noise`), and the music is a 16-step scheduler (bass + hat + melody) whose BPM scales with the round. |
| **Vector powerup icons instead of emoji** | The spec's icon table uses emoji (🪃, 🙈…). Emoji render inconsistently across OSes and can box-glyph. Each of the 8 icons is drawn as a canvas path in `drawIcon()` so the art is identical everywhere and matches the "thick outline, readable from 6 feet" direction. |
| Arena drawn as an **ellipse**, not a circle | The spec wants the arena at 60–70% of screen width, but 720 px of height only affords a ~540 px circle (42%). An ellipse (872×400) hits the width target, reads as a game-show stage in perspective, and leaves headroom for the HUD and the lives strip. |

### Build order

1. **Shell + config** — canvas scaling, math helpers, player/colour/key tables, powerup table with spec weights.
2. **Audio engine** — tone/noise primitives, 20+ named effects (one per powerup as the spec's priority list requires), music scheduler with tempo scaling and a ducking hook for the final-3-second heartbeat.
3. **Particles + input** — particle pool (dot/spark/smoke/confetti/ring/text/star), banner queue, per-key-code input map built from the active layout, `e.repeat` ignored so key-repeat can never spam a throw.
4. **Game logic** — ring geometry, bézier bomb flight, throw/arrive/explode, all 8 powerups, round & match flow, difficulty table.
5. **Rendering** — background/stage lights, arena, characters with 7 facial expressions, bomb with burning fuse, HUD, banners.
6. **Screens** — title, lobby, mode select, ready overlay, round end, match end with stats, pause.
7. **Test, then fix** (see below).

### Rules the spec left open — and how they were resolved

- **3-player positions.** The spec says both "evenly spaced points" and "12, 5, 7 o'clock", which contradict each other (12/5/7 is not even). Chose **evenly spaced (12, 4, 8 o'clock)** so clockwise and counter-clockwise are symmetric for every player — otherwise one player's two keys would feel unequal.
- **Powerup collection is near-deterministic.** The flight arc's control point is solved so the curve passes *through* the centre point at t=0.5, so a live powerup is essentially always collected by a throw. This follows the spec's "the bomb visually travels through the centre" — spawn rate, not aim, is the frequency lever. A small random jitter (±22 px) keeps arcs from looking identical.
- **Hot Swap vs. Shield.** Spec doesn't say. Ruled that a teleport **bypasses** a shield and leaves it intact for a later catch — the shield is described as deflecting a *thrown* bomb, and Hot Swap explicitly skips normal flight rules.
- **Who does ESC remove in the lobby?** Spec says "players can leave by pressing Escape" but there is one ESC key for four people. Implemented as **remove the most recently joined player**; ESC with nobody joined returns to the title screen.
- **Throw cooldown scope.** Read the 0.5 s minimum hold as applying to *every* catch, not only to a bomb thrown straight back. The round-opening holder has no lock (the 2 s "GET READY" grace covers it).
- **Round-start grace.** Bomb spawns at a random player and the timer does not tick for 2.2 s ("GET READY…" → "GO!").
- **Key layouts.** Both of the spec's tables are included; press `1` in the lobby to swap between STANDARD (Q/E · ←/→ · B/M · Num1/Num3) and NO-NUMPAD (Q/E · Z/C · ,/. · ←/→). Numpad codes accept `Numpad1`/`Numpad3` and their NumLock-off aliases `End`/`PageDown`.

### Bugs found during testing and fixed

All found by driving the real build in a browser and capturing frames, not by reading code.

1. **Bomb countdown hidden behind the HUD.** The floating number is drawn 62 px above the bomb; for the player at 12 o'clock that put it under the HUD bar or off-screen. First fix (clamping it down) collided with the powerup banner, second fix (flipping it below the bomb) landed on the character's face. **Final:** the number slides out to the *side* of the bomb into open arena when the bomb is high on screen.
2. **Powerup banner covered the 12 o'clock player and their bomb.** Moved the whole arena down and flattened it (`cy` 400→430, `ry` 236→200) and shortened the banner so it settles in the gap between the HUD bar and the top player's head.
3. **Key-cap labels overflowed their boxes.** `NUM1`/`NUM3` didn't fit a 34 px cap → relabelled `N1`/`N3` and widened caps to 42 px.
4. **Direction-hint arrows collided with the lives strip.** The ↺/↻ hints sat below the key caps and overlapped the bottom bar for the 6 o'clock player. Moved them to the outside of the caps, same line.
5. **Eliminated player was almost invisible at the moment of elimination.** Dead players render at 45% alpha so they read as spectators; that also faded the victim during their own explosion. Now the victim is spotlit at full alpha for the round-end beat, and only fades from the next round on.
6. **Lives mode showed no explosion reaction.** The charred/dazed pose keyed off `!alive`, so a player who lost a round but kept a life just stood there. Now keyed off "victim of this round" as well, and reset on the next round.
7. **Stale effect chips on the end screens.** A REVERSED/BLIND countdown chip could still be ticking in the HUD during the round-end and match-end overlays. Effects are now cleared on explosion.
8. **Winner was a small dim blob at the edge of the arena.** The match-end overlay now draws the champion large and centre-stage with a hop-and-wave dance on a sparkle ring, with the stats table re-laid-out beneath.
9. **Title screen's idle bomb toss flew straight through the "PRESS SPACE TO START" text.** Lowered the arc and widened the two characters' spacing.

### Verification

Driven programmatically in Chrome against the real build (the game loop, input
handler and renderer were called directly, one frame at a time):

- **Menu flow** — title → lobby → 4 joins → mode select → match start, plus rematch, quit-to-lobby, pause/resume, per-player leave, and layout toggle. All correct.
- **Every powerup forced during a live flight** — Time Freeze (2.5 s, timer drained 0.00 s/s), Time Warp (3× confirmed: 3.0 bomb-seconds burned per 1.0 real second vs. 1.0 normally), Rebound (flight reverses to the thrower), Sticky (lock = 2.5 s on catch), Hot Swap (instant, random holder, clears sticky), Blind (3 s), Shield (deflects on arrival to a random third player, consumed), Reverse (4 s, throw direction inverted).
- **Spec edge cases** — mid-flight explosion kills the *receiver*; mid-flight explosion after a Rebound kills the *thrower*; Hot Swap clears a pending Sticky; a non-holder pressing keys does nothing; the 0.5 s cooldown blocks a throw at 0.3 s and allows it at 0.6 s; 2-player = 1 round with both keys pointing at the only opponent; 3-player = 2 rounds in a triangle at −90°/30°/150°.
- **Difficulty progression** — round 1 timers land in 10–15 s, and a 2-player field is tagged FINAL with 5–8 s timers.
- **Soak test** — 33 full matches across 2/3/4 players in both modes with a bot throwing at random, ~150 rounds total, every frame both updated *and* rendered: **0 exceptions, 0 matches failed to terminate**, all 8 powerups observed firing.

### Not implemented

- `localStorage` win tallies / match history (spec: optional).
- Custom player names (spec: optional).

### Notes

- `.claude/launch.json` is a dev convenience for serving the folder locally while
  testing; it is not needed to play the game.
