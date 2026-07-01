# Blob (working title) — Full Detailed Design Doc

**Platform target:** CrazyGames (web/HTML5) — fast load, instant play, endless high-score loop.
**Engine:** Godot 4.x — **2D project** (fastest to build, fastest web load, perfect for this top-down blob game).
**Graphics:** Code-only (2D primitives — circles, simple shapes, particles — no imported assets).
**Genre:** Single-player arcade "eat to grow" survival. You're a blob that eats smaller things to grow bigger, while avoiding bigger things that can eat you. Endless — survive and grow as long as you can for a high score.
**Level of scope:** Same weight class as a finished endless-runner (Subway-Surfers-with-cars level). One core mechanic + escalation. Deliberately finishable. No networking, no multiplayer — the classic "agar.io feeling" as a focused single-player arcade game.

> **THE #1 RULE (learned the hard way): Build in chunks and PLAY-TEST each chunk before running the next prompt. Building fast without testing between steps is what wrecked past projects. See Section 12 for the chunk plan. Do not skip testing.**

---

## 1. The Game in One Sentence
You control a blob on a 2D field; eat blobs smaller than you to grow, avoid blobs bigger than you (they eat you), and survive as long as possible while your growth constantly shifts what's dangerous and what's food.

## 2. The Core Loop (why it's fun)
1. You start small.
2. You eat things smaller than you → you grow slightly.
3. As you grow, things that used to be dangerous become edible — the world "opens up."
4. But bigger threats still exist, and getting big makes you slower and a bigger target.
5. Push to grow bigger for a higher score, while the risk escalates.
6. Get eaten (touch something bigger) → game over → see your score → "one more try."

**The satisfying hook:** the *power curve*. Being hunted early, becoming the hunter late, and the constant re-evaluation of "can I eat that, or will it eat me?" That shifting relationship is the whole fun. Every size threshold crossed feels like an achievement.

## 3. The Player Blob
- A circle (the blob), a distinct bright color (e.g. a friendly blue or green), always centered-ish on screen (camera follows it).
- **Size = score-ish.** The blob has a **mass/size value** that grows as it eats. Its on-screen radius scales with this value.
- **Movement:** the blob moves toward the mouse cursor (or in the direction of WASD/arrows — decide during build; **mouse-follow is the classic, most intuitive choice for this genre**). The blob accelerates toward the target smoothly.
- **Speed vs. size tradeoff (important for depth):** the bigger the blob gets, the **slower** it moves. This is the key balancing mechanic — growth makes you powerful (can eat more) but vulnerable (can't escape as easily). Without this, the game has no tension once you're big. Tune the curve so big blobs feel powerful-but-lumbering, not helpless.

## 4. What's On the Field
### 4.1 Food (small static/drifting pellets)
- Small dots scattered across the field (the "grass" of the ecosystem). Always smaller than the player.
- Eating one gives a small amount of growth.
- They **respawn / are continuously present** so the field never runs empty — there's always something to nibble.
- Purpose: steady, safe, slow growth. The baseline income.

### 4.2 Enemy Blobs (the ecosystem)
- Other blobs of **varying sizes** roam the field — this is the heart of the game.
- **Smaller than you** = food. Touch them (or engulf them) to eat them and grow more than a pellet gives (bigger prey = bigger reward).
- **Bigger than you** = predator. If a bigger blob touches you, **you get eaten = game over**.
- **Roughly your size** = tense standoff — neither can safely eat the other; a slight size edge decides it.
- Enemy blobs should themselves **eat pellets and smaller blobs and grow over time**, so the ecosystem evolves — the field gets more dangerous the longer a round goes, which is the natural difficulty ramp.
- **Enemy AI (keep it simple):** enemy blobs chase blobs smaller than themselves (including the player if the player is smaller) and flee from blobs bigger than themselves. That single rule — "chase smaller, flee bigger" — creates surprisingly lifelike, dynamic behavior. No complex AI needed.

### 4.3 (Optional, Phase 1.5) Hazards
- Optional spice: static hazards like spike balls that damage/shrink any blob that touches them, or a shrinking play-area border. **Not required for v1** — flag as a nice-to-have. The core eat/grow/avoid loop is enough to ship.

## 5. Eating & Growth Rules (the math that makes it feel right)
- **Can eat X if:** the player's size is meaningfully larger than X's size (e.g. player must be ~15–25% bigger to consume another blob — tune this). A tiny size difference shouldn't instantly consume; a clear size advantage should.
- **Growth on eating:** the player's mass increases by a portion of what they ate (pellet = small fixed amount; enemy blob = proportional to that blob's size — eating a big blob is a big jump).
- **Diminishing growth (optional but recommended):** to prevent runaway snowballing where a big blob becomes unkillable, growth from small food could matter less as you get huge, keeping the late game tense. Tune during playtest.
- **Score:** tie score to size reached and/or blobs eaten. Simplest: score = current mass, and track the max mass reached this run as the "high score" metric. Or score = total blobs eaten. Decide during build; mass-based is the most intuitive.

## 6. The Camera
- **Follows the player blob**, keeping it roughly centered.
- **Zooms out slightly as the player grows** — so as you get bigger, you see more of the field. This is a classic, important touch: it makes growth *feel* impactful (the world literally gets bigger around you) and keeps big blobs playable (you can see approaching threats). Tune the zoom curve so it's smooth, not jarring.
- The field is larger than the screen — the player explores it as they move.

## 7. The Field / Play Area
- A bounded 2D field (e.g. a large rectangle), bigger than the visible screen so there's room to roam and flee.
- **Edges:** either solid walls (blobs bump into them — can be used to corner prey or get cornered) or a soft boundary. Walls add tactical depth (trap smaller blobs against them). Recommend walls.
- **Background:** a simple flat or subtly-gridded background (a faint grid helps convey movement/scale). Code-drawn, lightweight.
- **Size:** big enough to feel like an ecosystem, small enough that encounters happen often (empty fields are boring). Tune so you're rarely more than a few seconds from a meaningful encounter.

## 8. Difficulty & Escalation
The game naturally escalates without needing scripted waves, because the ecosystem evolves:
- Enemy blobs grow over time by eating, so the field gets more dangerous the longer you survive.
- **Optional explicit ramp:** spawn new enemy blobs periodically, occasionally spawning some that are already large (a "big fish enters the pond" tension spike).
- The player's own growth-slows-you-down mechanic means late game is inherently tenser.
- Goal: every run should build toward a satisfying "I'm huge and powerful but everything's trying to take me down" climax, ending in an eventual death that feels earned. Then: one more try.

## 9. Juice / Game Feel (critical — this is what makes it addictive vs. boring)
Simple games live or die on feedback. Budget real attention here:
- **Eating a blob:** a satisfying pop — the eaten blob bursts into particles, a brief scale-punch on the player blob, maybe a soft "gulp" flash. The moment of eating must feel *good*.
- **Growing:** smooth size interpolation (grow into the new size over a fraction of a second, not an instant snap), plus the camera zoom-out reinforcing it.
- **Near-miss with a bigger blob:** maybe a subtle danger vignette/flash when a bigger blob is very close — heightens tension.
- **Getting eaten (game over):** a dramatic burst — the player blob pops apart into particles, a screen shake/flash, then the game-over screen. Death should feel punchy, not just "you stopped."
- **Color:** enemy blobs could be tinted by threat — e.g. blobs you can eat subtly greenish, blobs that can eat you subtly reddish, relative to your current size. This is a HUGE usability win (instant read of "food vs. threat") and looks great. Strongly recommended.

## 10. UI / Screens
- **Start screen:** game title/logo, a "PLAY" button, and the current high score displayed. Lightweight, fast first paint (web game).
- **In-game HUD:** current score/size (a number), and optionally a small indicator of how you rank in size vs. others on the field ("#3 biggest"). Keep it minimal — the game is the focus.
- **Game-over screen:** "You were eaten!" (or similar), final score, high score (with a "NEW BEST!" callout if beaten), and a "PLAY AGAIN" button that instantly restarts. Fast restart is essential for the "one more try" loop — no slow menus between attempts.
- **High score persistence:** store the best score locally so it survives between sessions.

## 11. Definition of Done (Phase 1 — SHIP THIS)
DONE when:
- [ ] A 2D field (bounded, larger than screen) with a camera that follows the player blob.
- [ ] Player blob moves (mouse-follow), and its on-screen size scales with its mass.
- [ ] Bigger player = slower movement (the speed/size tradeoff works).
- [ ] Pellets exist across the field, are eaten on contact, give small growth, and stay populated.
- [ ] Enemy blobs of varying sizes roam the field with "chase smaller / flee bigger" AI.
- [ ] Eating rules work: player eats blobs sufficiently smaller (growing proportionally); touching a bigger blob = game over.
- [ ] Enemy blobs also eat and grow over time (the ecosystem evolves = natural difficulty ramp).
- [ ] Camera zooms out smoothly as the player grows.
- [ ] Threat-based color cue: player can tell at a glance which blobs are food vs. danger (e.g. green/red tint relative to their size).
- [ ] Score tracked (size/blobs eaten), displayed in HUD.
- [ ] Start screen (PLAY + high score), game-over screen (final + high score + PLAY AGAIN), fast restart.
- [ ] Local high-score persistence.
- [ ] Juice: eat-pop particles + scale-punch, smooth growth, and a punchy game-over burst (shake/flash + particles).

**When all boxes are checked, the game is shippable. STOP adding features and ship it.**

## 12. Build Plan (chunks — TEST EACH BEFORE THE NEXT PROMPT)
This is the discipline that gets it finished. Build in order; PLAY-TEST each chunk before running the next prompt. Do NOT run them back-to-back.

**Chunk 1 — Blob + movement + pellets:**
Field, camera-follows-blob, mouse-follow movement, size scales with mass, pellets that spawn and are eaten for small growth, size-slows-you-down. (Test: moving around eating pellets and slowly growing feels good.)

**Chunk 2 — Enemy blobs + eat/be-eaten:**
Enemy blobs of varying sizes with "chase smaller / flee bigger" AI; eating rules (eat smaller = grow, touch bigger = game over); enemies eat/grow too. (Test: the eat-or-be-eaten ecosystem works and is tense.)

**Chunk 3 — Camera zoom + threat colors + score + escalation:**
Camera zoom-out on growth, green/red threat-tint on blobs, score tracking + HUD, difficulty ramp (enemies growing/spawning over time). (Test: a full run has a real arc from small-and-hunted to big-and-hunted.)

**Chunk 4 — Screens + juice + polish:**
Start screen, game-over screen, high-score persistence, fast restart, and the full juice pass (eat pops, scale-punch, smooth growth, game-over burst). (Test: it's a complete, satisfying, shippable package.)

Four chunks. Test each. The testing between chunks is not optional — it's the whole strategy.

## 13. Explicitly OUT of Scope for Phase 1 (resist these!)
- **Multiplayer / online (the real agar.io)** — NOT in v1. This is single-player vs AI blobs only. Multiplayer is a whole separate (hard) effort; the single-player version is a complete, fun game on its own.
- Multiple maps/modes.
- Power-ups / abilities (e.g. split, boost, shoot) — the real agar.io has "split" and "eject mass"; these add depth but are Phase 2. v1 is pure eat/grow/avoid.
- Skins/cosmetics/progression between runs (nice Phase 2 hook; not needed to ship).
- Sound/music (adds juice later; not required to ship — though even simple SFX helps a lot).
- Mobile/touch controls.
- Hazards/shrinking border (optional Phase 1.5 spice, not required).
- Anything not in the Definition of Done.

## 14. Why This Is Genuinely Achievable For You
- It's a proven-simple genre at the exact level you've already shipped (endless-runner tier).
- **2D + code-only + no networking** = none of the traps that caused "feels incomplete" before (no 3D perf worries, no wall-collision bugs, no netcode).
- The AI is literally one rule ("chase smaller, flee bigger") — deceptively rich, trivially cheap.
- The depth/fun comes from a single elegant mechanic (the size relationship) + juice, not from content sprawl.
- It has a naturally addictive power-curve hook, so a small build punches above its weight in fun.

## 15. The One Rule
This exists to be FINISHED and SHIPPED, like your other small games. The fun is in one elegant mechanic (size = power AND vulnerability) plus great game feel — not in features. Build in four chunks, test each. Multiplayer and power-ups stay out of v1. When in doubt, CUT, don't add. A shipped blob-eater beats an unshipped one.
