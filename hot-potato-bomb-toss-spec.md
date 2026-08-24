# Hot Potato Bomb Toss — Game Design Specification

## Concept

A local multiplayer party game for **2–4 players on one keyboard.** A ticking bomb passes between players — whoever is holding it when it explodes is eliminated. Rounds are fast (15–30 seconds each), chaotic, and spectator-friendly. Powerups appear mid-throw and intercept the bomb in flight, warping the rules in unpredictable ways.

The game is a browser-based, single-screen 2D game. A full match lasts 1–3 minutes. The engagement loop is: laugh → yell at your friend → demand a rematch → the person watching says "let me play next."

---

## Player Setup

### Player Count

The game supports **2, 3, or 4 players.** Player count is selected on the lobby screen before the match begins.

### Arena Arrangement

Players are positioned around a **circular arena** at evenly spaced points:

- **2 players:** Left and Right (3 o'clock and 9 o'clock positions).
- **3 players:** Triangle formation (12, 5, 7 o'clock).
- **4 players:** Square formation (12, 3, 6, 9 o'clock).

Each player is represented by a distinct character (color-coded with a simple body shape — no complex sprites needed). Characters face inward toward the center of the arena. They do not move around the arena — they are **stationary at their position.** The only action is throwing.

```
        [P1]
         |
  [P4]---●---[P2]      (4-player layout)
         |
        [P3]
```

The center of the arena is the **powerup zone** — an open space where powerup icons spawn and where the bomb visually travels through.

### Player Identity

Each player slot has:

- A **color:** Player 1 = Red, Player 2 = Blue, Player 3 = Green, Player 4 = Yellow.
- A **name label** displayed under their character: "P1", "P2", "P3", "P4" (or custom names if name entry is implemented — optional).
- A **key indicator** showing their control key(s) next to their character at all times during gameplay.

---

## Controls

Each player has **two keys**: one to throw the bomb **clockwise** and one to throw **counter-clockwise** around the ring. With 2 players, both keys throw to the same (only) opponent, so it's effectively one button.

| Player | Throw Left (CCW) | Throw Right (CW) |
|---|---|---|
| P1 (Red) | Q | E |
| P2 (Blue) | Left Arrow (←) | Right Arrow (→) |
| P3 (Green) | B | M |
| P4 (Yellow) | Numpad 1 (End) | Numpad 3 (PgDn) |

Alternative simpler layout (if numpad is unavailable or awkward):

| Player | Throw Left (CCW) | Throw Right (CW) |
|---|---|---|
| P1 (Red) | Q | E |
| P2 (Blue) | Z | C |
| P3 (Green) | Comma (,) | Period (.) |
| P4 (Yellow) | Left Arrow (←) | Right Arrow (→) |

The specific key bindings matter less than two principles: keys must be **physically spaced** so 4 people can share a keyboard without hand collisions, and each player's two keys must be **adjacent** and operable with one hand.

Only the player currently holding the bomb can throw. Pressing a throw key when you don't have the bomb does nothing (no penalty, no advantage).

---

## The Bomb

### Appearance

A classic cartoon bomb: black sphere with a lit fuse/wick. The fuse visibly burns down as the timer counts. When the timer is low, the bomb flashes red and shakes with increasing intensity.

### Bomb Timer

Each round, the bomb spawns with a **random countdown timer** between **8 and 15 seconds** (the range shifts as the match progresses — see Difficulty Progression below). The timer is displayed in two ways:

1. **Fuse length:** The bomb's burning wick gets shorter as time passes — a glanceable analog indicator.
2. **Numerical countdown:** A large floating number above the bomb showing seconds remaining, ticking down in tenths (e.g., "7.3"). This number grows larger and turns red in the final 3 seconds.

The timer **always counts down** (unless modified by a powerup). It does not pause between throws — it ticks during flight, during holds, always. This creates urgency: every fraction of a second the bomb is in your hands is dangerous.

### Bomb Flight

When a player throws the bomb, it travels visually from the thrower to the target player. The flight is not instant — it takes **0.4–0.6 seconds** (scales slightly with arena distance). During flight:

- The bomb follows a **slight arc** (parabolic trajectory) between the two players, passing through or near the center of the arena.
- The bomb is "in transit" — no one is holding it, and if it passes through a powerup icon in the center, the powerup activates.
- The timer continues counting down during flight.
- If the bomb timer expires mid-flight, it explodes where it is, and the **player it was traveling toward** is the one eliminated. The rationale: you threw it in time, so the receiver takes the hit. (This also prevents a degenerate strategy of holding until 0.1s and throwing so it explodes in midair as a "draw.")

### Bomb Reception

When the bomb arrives at a player, they are now "holding" it. There is no catch action — reception is automatic. The player's character visually grabs the bomb and begins panicking (shaking, sweat drops, wide eyes). The player can then throw it immediately (no minimum hold time unless a Sticky Bomb powerup is active).

### Throw Cooldown

After a player throws the bomb and it is thrown back to them, there is a **minimum hold time of 0.5 seconds** before they can throw again. This prevents degenerate instant-volley play and gives the audience a moment to register what happened. This base cooldown is separate from the Sticky Bomb powerup (which is longer and stacks on top).

---

## Powerups

Powerups are the chaos engine. They spawn as floating icons in the **center of the arena** and are collected automatically when the bomb's flight path passes through them. No player directly picks up a powerup — the bomb does.

### Spawning Rules

- A powerup spawns in the center of the arena **every 3–5 seconds** (random interval).
- Only **one powerup** can exist in the arena at a time. If the current one hasn't been collected, no new one spawns until it is collected or despawns.
- Uncollected powerups **despawn after 6 seconds** (with a fade-out warning in the last 1.5 seconds).
- Powerups do not spawn in the **first 2 seconds** of a round (let players settle in).
- A brief "powerup incoming" chime plays when one spawns, drawing attention to the center.

### Powerup Activation

When the bomb passes through a powerup during flight, the powerup is consumed and its effect activates **immediately.** A large banner briefly flashes the powerup name and icon on screen so all players (and spectators) know what just happened.

Some powerups affect the bomb itself, some affect the thrower, some affect the receiver, and some affect everyone. This unpredictability is the point — the player who threw doesn't know if the powerup will help or hurt them.

### Powerup List

#### 1. Time Freeze ❄️
- **Effect:** The bomb timer **pauses for 2.5 seconds.** The countdown number freezes and turns blue. The fuse stops burning.
- **Who benefits:** The receiver (they get extra time to react and think). Also gives everyone a breather.
- **Visual:** Bomb gets a frosty blue tint and ice crystal particles. A clock-stop sound plays.

#### 2. Time Warp ⚡
- **Effect:** The bomb timer **runs at 3× speed for 3 seconds.** Three seconds of real time burns through 9 seconds of bomb time.
- **Who it hurts:** Whoever is holding the bomb during those 3 seconds. Could hurt the receiver, or the next person if the receiver throws quickly.
- **Visual:** Bomb glows hot orange, timer number blurs/strobes, crackling electricity particles.

#### 3. Rebound 🪃
- **Effect:** The bomb, mid-flight, **reverses direction** and flies back to the player who just threw it. They become the holder again and must re-throw.
- **Who it hurts:** The thrower. They thought they were safe — surprise.
- **Visual:** The bomb does a sharp U-turn with a boomerang trail effect. A comedic "boing" sound plays. Spectators love this one.

#### 4. Sticky Bomb 🍯
- **Effect:** The receiving player **cannot throw the bomb for 2.0 seconds** after catching it (on top of the normal 0.5s cooldown, so 2.5s total hold). Their character is visually stuck, struggling with a gooey bomb.
- **Who it hurts:** The receiver. They're forced to hold a ticking bomb with no escape.
- **Visual:** Bomb gets a dripping honey/slime texture. The receiver's character is visibly struggling, pulling at the bomb. A squelchy "stuck" sound plays.

#### 5. Hot Swap 🔀
- **Effect:** The bomb **teleports instantly to a random player** (could be anyone, including the person who just threw it, or someone who hasn't been involved in the current volley). No flight animation — just a poof at the origin and a poof at the destination.
- **Who it affects:** Everyone. Pure chaos. Keeps every player on edge even if they haven't been targeted in a while.
- **Visual:** Bomb vanishes in a purple smoke puff, reappears at a random player with a matching smoke puff and a "warp" sound. The recipient looks shocked.

#### 6. Blind Toss 🙈
- **Effect:** The **bomb timer number is hidden for 3 seconds.** The fuse is also obscured (covered by a "?" icon). Players must rely on instinct and the bomb's flashing/shaking intensity to gauge how much time is left.
- **Who it affects:** Everyone. Raises tension dramatically because no one knows the exact moment of explosion.
- **Visual:** Timer display is replaced with a large flashing "?". Bomb becomes a dark silhouette with question marks orbiting it. An eerie tension sound plays.

#### 7. Shield 🛡️
- **Effect:** The receiving player gets a **one-time auto-deflect.** Instead of catching the bomb, it automatically bounces to a random other player. The shield is consumed on deflect.
- **Who it benefits:** The receiver. They dodge one hot potato without pressing anything.
- **Visual:** A shimmering barrier appears around the receiving player for a moment, the bomb bounces off with a "ping" sound and a spark effect, then the shield shatters.

#### 8. Reverse ↩️
- **Effect:** The **throw direction meaning flips for all players for 4 seconds.** The "clockwise" key now throws counter-clockwise and vice versa. Affects everyone simultaneously.
- **Who it affects:** Everyone. Causes fumbled throws and panicked wrong-direction tosses. Muscle memory becomes the enemy.
- **Visual:** A large rotating arrow icon flashes on screen. All player key indicators briefly swap. A "rewind" sound plays. A visible countdown shows when normal controls return.

### Powerup Weights

Weighted to favor chaos and spectacle:

- Time Freeze: 12%
- Time Warp: 15%
- Rebound: 18% (crowd favorite — always funny)
- Sticky Bomb: 15%
- Hot Swap: 12%
- Blind Toss: 10%
- Shield: 8% (rare — feels special when it appears)
- Reverse: 10%

---

## Round Structure

The game is played in **rounds.** Each round ends when the bomb explodes. The player holding the bomb (or the one it was traveling toward, if mid-flight) is the round's loser.

### Round Flow

1. **Round start:** The bomb spawns at a random player. A 2-second grace period begins — the timer is not yet ticking. The text "GET READY..." appears, then "GO!" when the timer starts. All players see who has the bomb.
2. **Active play:** The bomb timer ticks. Players throw. Powerups spawn and activate. Chaos ensues.
3. **Explosion:** Timer hits zero. The bomb explodes with a dramatic animation centered on the holder. That player is visually "blasted" (comedic — charred face, ruffled feathers, stars circling head). They are eliminated from the current match.
4. **Round result:** A brief 2-second freeze-frame shows "P3 ELIMINATED!" (or equivalent) with a comedic animation. The eliminated player's avatar on the arena dims/grays out.
5. **Next round:** Remaining players reposition (evenly spaced around the ring), and the next round begins after a 2-second countdown.

### Match Structure

- A match continues until only **one player remains.** That player wins the match.
- With 2 players, a match is a single round (sudden death — winner take all).
- With 3 players, a match is 2 rounds (2 eliminations).
- With 4 players, a match is 3 rounds (3 eliminations).
- After a match ends, the winner celebration plays, and the game returns to the lobby/rematch screen.

### Alternative: Lives Mode (Optional)

Instead of single-elimination, each player starts with **3 lives** (bombs/hearts). Losing a round costs 1 life. A player is eliminated at 0 lives. This extends match length for a meatier experience, but single-elimination is recommended for the event (faster turnover, more people get to play).

---

## Difficulty Progression

Difficulty increases **within a match** as rounds progress (not within a single round — rounds are too short for in-round scaling).

| Round | Bomb Timer Range | Powerup Spawn Rate | Notes |
|---|---|---|---|
| Round 1 | 10–15 seconds | Every 4–5 seconds | Relaxed. Players learn the controls. |
| Round 2 | 8–12 seconds | Every 3–4 seconds | Tighter. Less breathing room. |
| Round 3 | 6–10 seconds | Every 2.5–3.5 seconds | Frantic. Powerups flying constantly. |
| Finals (2 players left) | 5–8 seconds | Every 2–3 seconds | Maximum chaos. Crowd is loud. |

The narrowing timer range is the main pressure lever. Powerup frequency increase compounds it — more powerups means more unpredictability per second.

---

## Visual Design

### Art Direction

Bright, bold, and readable from 6 feet away (spectators standing behind a seated player). Thick outlines, high-contrast colors, big expressive animations. Think "Bomberman meets Mario Party minigame."

### Arena

- A **circular arena** occupying the center of the screen, taking up roughly 60–70% of screen width.
- The arena floor is a simple flat disc (subtle radial gradient — lighter in the center where powerups spawn, darker at edges).
- The arena border is a thick ring (could be metallic, or styled as a game-show stage).
- The center of the arena has a subtle pulsing glow or spotlight to draw attention to where powerups will spawn.

### Player Characters

Simple, expressive, distinct. Each player is a **round/blob character** in their team color with:

- Two large eyes (for expressions — wide when panicking, squinted when throwing, spinning when eliminated).
- Two stubby arms (for holding/throwing the bomb).
- No legs (they don't move — they're planted at their position).
- A color fill matching their player color (Red, Blue, Green, Yellow) with a white outline for contrast.
- Idle animation: gentle bounce/breathing.
- Holding bomb animation: shaking, sweating, panicking (intensity scales with how low the bomb timer is).
- Throwing animation: big arm wind-up and release.
- Hit animation: puffed-up explosion cloud, charred/dazed face, cartoon stars.

### Bomb

- A classic black sphere with a lit fuse.
- The fuse shortens over time (proportional to timer remaining).
- When timer < 3 seconds: bomb flashes red at increasing frequency.
- When timer < 1.5 seconds: bomb shakes violently and emits spark particles.
- Flight trail: a dashed arc line trails behind the bomb as it flies. The trail fades after 0.5s.

### Powerup Icons

Each powerup has a distinct icon and a colored glow matching its identity:

| Powerup | Icon | Glow Color |
|---|---|---|
| Time Freeze | Snowflake ❄️ | Ice blue |
| Time Warp | Lightning bolt ⚡ | Hot orange |
| Rebound | Boomerang 🪃 | Green |
| Sticky Bomb | Honey drop 🍯 | Amber/gold |
| Hot Swap | Shuffle arrows 🔀 | Purple |
| Blind Toss | Eye-cover 🙈 | Dark grey |
| Shield | Shield icon 🛡️ | Silver/white |
| Reverse | U-turn arrow ↩️ | Pink |

Powerups float in the center of the arena with a gentle bob (up-down hover), rotating slowly, and a soft pulsing glow. When collected by the bomb, they burst into color-matched particles.

### Screen Layout

```
┌──────────────────────────────────────────────┐
│   ROUND 2 / 3           ⏱ BOMB: 6.4s        │  ← HUD bar
│                                              │
│              [P1 - Red]                      │
│                  ↕                           │
│   [P4 - Ylw] ← 💣🔀 → [P2 - Blu]           │  ← Arena with bomb
│                  ↕                           │  in flight, powerup
│              [P3 - Grn]                      │  in center
│                                              │
│   ❤❤❤          ❤❤❤        ❤❤❤        ❤❤🖤   │  ← Lives (if using
│    P1            P2         P3          P4   │    lives mode)
└──────────────────────────────────────────────┘
```

---

## Screen Effects (Juice)

### Bomb In-Hand

- Character shakes with increasing amplitude as timer drops.
- Sweat drop particles emit from character.
- At timer < 3s, the screen edges start pulsing red (subtle vignette).
- At timer < 1.5s, the screen vignette intensifies and the camera (or arena) subtly zooms in.

### Throw

- A sharp "whoosh" sound.
- Speed lines briefly appear in the throw direction.
- The throwing character does a big arm-swing animation with squash-and-stretch follow-through.

### Powerup Collection

- The powerup icon bursts into colored particles.
- A distinct sound per powerup type (short, punchy, recognizable).
- A banner with the powerup name + icon slides in from the top center and out again over ~1 second. Large enough for spectators to read.
- If the powerup is negative for the receiver, the banner is red/orange tinted. If positive/neutral, it's blue/green tinted. Helps spectators instantly read the mood.

### Explosion (Round End)

- **Big.** The bomb expands, the screen flashes white for 1 frame, then a cartoonish explosion cloud (orange, yellow, smoke puffs) fills the area around the loser.
- Heavy screen-shake (8–10px, 300ms).
- The losing character is shown charred/flattened in a comedic pose inside the smoke cloud.
- Confetti or debris particles scatter outward.
- A dramatic boom/explosion sound followed by a sad trombone or comedic "wah wah waaah."
- The other players do a brief victory flinch (covering their faces) then celebrate.

### Match Winner

- The winning player's character does a victory dance (jumping, waving arms).
- Fireworks/confetti particles fill the screen.
- "P1 WINS!" text in their player color, large, with a glow effect.
- A triumphant fanfare sound.

---

## Game Flow & Screens

### 1. Title Screen

- Game title: big, explosive typography. "HOT POTATO" with a bomb icon replacing the "O" in POTATO, fuse lit and sparking.
- A looping idle animation: a bomb bouncing between cartoon characters in the background.
- **"Press SPACE to Start"** prompt (pulsing).
- Below: "2–4 Players | One Keyboard | Zero Mercy" tagline.

### 2. Lobby / Player Select

- Central screen showing 4 player slots arranged in a row or grid.
- Each slot displays: player color, assigned keys, and a "Press [key] to join" prompt.
- A player joins by pressing either of their throw keys. Their slot lights up and their character does a "ready" animation.
- **Minimum 2 players** required to start.
- Once all desired players have joined, a "Press SPACE to Begin" prompt appears.
- Players can leave by pressing Escape (their slot dims and returns to "press to join").

### 3. Mode Select (Optional — Skip If Tight On Time)

- **Quick Match** (single elimination — default, recommended for events).
- **Lives Match** (3 lives each — longer, for dedicated play sessions).
- Selected by the player who presses Space. Arrow keys to toggle, Space to confirm.
- If this screen is skipped, default to Quick Match.

### 4. Gameplay Screen

- The arena occupies center screen.
- HUD bar at top: Round counter (e.g., "ROUND 2 / 3"), bomb timer (large, center), and match status.
- Player positions around the arena with their color and key labels.
- Lives display below each player (if using lives mode).
- Powerup banner area: top-center, overlays briefly when a powerup activates.

### 5. Round End Screen

- Brief overlay (2 seconds): "[Player color] ELIMINATED!" with their charred character.
- Surviving players reposition for the next round.
- Auto-advances to the next round after the 2-second display plus a 2-second "Round X — GET READY" countdown.

### 6. Match End Screen

- Full overlay: "[Player color] WINS THE MATCH!" with victory animation.
- Match stats displayed:
  - Total rounds played
  - Fastest throw (shortest hold time before throwing)
  - Most rebounds caught (times a Rebound hit them)
  - "Most panicked" — player who held the bomb the longest total across all rounds
- **"SPACE for Rematch"** — restarts with the same player configuration.
- **"ESC to Lobby"** — returns to player select for new players to join.

---

## Audio

### Music

- A looping upbeat track with a comedic, tense quality (think "Wii Party" or "Jackbox" energy).
- Tempo increases slightly in later rounds.
- In the final 3 seconds of a bomb timer, the music cuts to a rapid heartbeat/ticking sound that builds to the explosion. This is the single most important audio cue in the game — it makes spectators hold their breath.

### Sound Effects (Priority Order)

1. **Explosion** — deep boom + comedic aftermath (essential — the payoff moment)
2. **Throw whoosh** — short, sharp, satisfying
3. **Bomb receive** — a nervous "catch" thud
4. **Timer ticking** — accelerating tick in final 3 seconds (essential for tension)
5. **Powerup collect** — sparkle/chime
6. **Powerup-specific sounds:**
   - Rebound: boomerang whistle + boing
   - Sticky Bomb: squelch
   - Hot Swap: warp/teleport zap
   - Time Freeze: crystalline freeze sound
   - Time Warp: speed-up whir
   - Reverse: rewind tape sound
   - Shield: metallic ping
   - Blind Toss: eerie whoosh
7. **Round start countdown** — "3, 2, 1, GO!" (voice or beeps)
8. **Victory fanfare**
9. **Sad trombone** (for the eliminated player)

---

## Edge Cases & Rules

### Simultaneous Input

If two players press their throw keys on the exact same frame, only the bomb holder's input registers (since only the holder can throw). No conflicts arise.

### Bomb Arrives During Sticky Bomb Hold

If a player is stuck (Sticky Bomb active) and another powerup like Hot Swap redirects the bomb to a different player, the sticky status is cleared — it only applies to the specific catch it was active for.

### Shield + Rebound Interaction

If the bomb rebounds (Rebound powerup) back to the thrower, and the thrower has a Shield, the Shield activates and deflects it to a random third player. Shield is consumed. Both powerups resolve.

### Powerup During Hot Swap

If a Hot Swap teleports the bomb (no flight), no powerup can be collected because there is no flight path through the center. This is intentional — Hot Swap skips the powerup gamble.

### 2-Player Rebound

In a 2-player game, Rebound sends the bomb back to the thrower. The thrower catches it and must throw it again (to the only other player). Rebound is still useful here because it wastes the thrower's time (forced to hold and re-throw while the timer ticks).

### Last-Second Throw

If a player throws the bomb and it explodes mid-flight, the **receiver** is eliminated (the thrower got rid of it in time). The exception: if a Rebound activates and the bomb is flying back to the thrower when it explodes, the **thrower** is eliminated.

### Timer Reaches Zero While Hot Swap Is Activating

The bomb explodes at whoever it teleported to (the destination player of the Hot Swap). The teleport is considered instantaneous.

---

## Controls Summary

| Input | Context | Action |
|---|---|---|
| Player throw keys | Gameplay (holding bomb) | Throw bomb in chosen direction |
| Player throw keys | Lobby screen | Join the match |
| Spacebar | Title screen | Start game |
| Spacebar | Lobby (2+ players joined) | Begin match |
| Spacebar | Match end screen | Rematch |
| Escape | Lobby | Leave player slot |
| Escape | Match end screen | Return to lobby |
| Escape | Gameplay (optional) | Pause |

---

## Technical Notes (Web Implementation)

- **Target resolution:** 1280×720 minimum, responsive up to 1920×1080. Maintain aspect ratio with letterboxing.
- **Rendering:** HTML5 Canvas recommended. The arena, characters, bomb, and particles are all simple shapes — canvas handles this easily at 60 FPS. DOM-based rendering is viable but canvas gives smoother particle and animation control.
- **Frame rate target:** 60 FPS. All timing uses `requestAnimationFrame` with delta-time. Game logic is decoupled from frame rate.
- **Input handling:** Listen for `keydown` events. Debounce per-player to prevent key-repeat spam (only register a new throw on key-down, not key-held). Use a key-state map to track which keys are currently pressed.
- **Storage:** `localStorage` for match history / win tallies (optional). No critical persistence needed — this is a live party game, not a score-chaser.
- **Dependencies:** Zero dependencies recommended. Vanilla JS + Canvas. If a library speeds development, Kontra.js or a similar lightweight engine is acceptable.
- **Deployment:** Single HTML file that opens directly in a browser. No server, no build step, no install.

---

## Scope Priorities

### Must-Have (Playable Party Game)
1. Lobby screen with 2–4 player join (press key to join)
2. Arena renders with player characters at positions
3. Bomb spawns at random player with visible countdown timer
4. Throw mechanic: bomb flies from holder to target with arc animation
5. Explosion when timer hits zero, correct player eliminated
6. Round-to-round flow: elimination, repositioning, next round
7. Match winner screen with rematch option
8. Throw cooldown (0.5s minimum hold)
9. At least basic key labels shown next to each player

### Should-Have (The Fun Part)
10. 4–5 powerups implemented (Rebound, Sticky Bomb, Time Freeze, Time Warp, Hot Swap)
11. Powerup spawning in center, bomb collects on fly-through
12. Powerup activation banners
13. Screen-shake and explosion particles
14. Character panic animation (scales with timer)
15. Sound effects (at least explosion, throw, timer tick, and powerup collect)
16. Difficulty scaling across rounds (shorter timers, more powerups)

### Nice-to-Have (Polish)
17. All 8 powerups (add Shield, Blind Toss, Reverse)
18. Background music with tempo scaling
19. Match stats on end screen
20. Character expressions (eyes reacting to events)
21. Victory confetti / fireworks
22. Animated title screen
23. Lives mode as alternative to single-elimination
24. Edge case handling for complex powerup interactions
