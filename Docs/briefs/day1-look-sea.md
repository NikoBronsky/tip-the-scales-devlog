# Brief: day one, look step 1 — the sea, the fish, the bite and the catch look like the prototype

Status: written on 12.11.2026 at 10:57, handed out at 11:15 (owner's clock). Executor: Opus, effort high.

## 1. Goal

The owner compared the Unreal build with the browser prototype. His words:

- «Сейчас, если честно, все выглядит очень плохо. Насколько это супер круто выглядело в нашем артефакте, настолько сейчас все ужасно выглядит.»
- «Из хорошего у нас глубина. Но опять-таки шейдеры могли быть лучше, сильно лучше.»
- «Поклевка тоже плохо выглядит.»
- «мы берем текстурку, накладываем ее на выпуклую какую-то ... или цилиндрообразную штуку, и тогда она будет выглядеть объемно, и она при этом будет как бы 2D-шной. ... Например, ведро можно так сделать»

The picture of the prototype is the target. This step brings the world part of the day screen to that level: fish with identity, the water and its depth, the bite and the hang, the catch and the bucket. The HUD and the panels are another step and another executor.

## 2. What exists now (verified by the orchestrator)

- **The look reference: `Docs/design/look-reference.md`** (831 lines). It describes the prototype in words, hex colours, ratios and timings. It is your only source for the look. You never see the prototype and you do not look for it. Its section 10 ranks the ten traits of the look and lists the gaps of the current build. Read sections 1-5, 8 and 10 before you plan.
- The game: `Catch/Catch.uproject`, modules `CatchCore` (rules, do not edit) and `Catch` (Unreal layer). Scene code: `CatchSeaView.*` (fish as instanced meshes, six per-instance floats, water, camera), `CatchTackleView.*` (raft, bucket, rods, lines, hooks, flight), `CatchSession.*` (owns the core, one console command per action, `catch.Record`, `catch.Shot`, `catch.After`), `CatchWorld.h` (1 game unit = 2 cm; game x=0 is world +1000 cm). Reports of the earlier steps: `Docs/briefs/reports/day1-scene.md`, `day1-playable.md`, `day1-integrate.md`.
- The fish now: one temporary grey mesh for every species (`SM_TempFishBody`, 512 triangles; `SM_TempFishCard`), material `M_Fish` (masked, two-sided). The scene fog washes the fish pale. Schools and positions come from the core and are right.
- Measured on this machine: the frame time is the same at 300 and at 1,000 fish (4.3 ms).
- Owner's decisions that bind this step: fish body = the thin convex body (decision `fish-body`); props by the same method, a texture on a convex or cylindrical shape (`convex-shell-props`); water variant A with the bright rim at the water line (`water-look`); the game is one scene, the sea is a block like a diorama, the view tilts, depth matters more than width; up to 1,000 fish; polish and feel are the main thing, every action answers with motion, sound and effect.
- The game sounds are ready and verified by the orchestrator: `Catch/SourceArt/Sound/` holds 29 WAV files and a README with the play rules (pitch per step, gates, the creak and ratchet tickers, the master compressor). The game plays its eleven temporary sounds from `CatchTackleView` now (`Play` and the `Sound*` properties).
- State of the repository at hand-out: the integration step is committed and the core branch is merged into `main`. `python -X utf8 Tools/check_all.py --ue` is green. What the integration step changed in your files: the fish mirror is keyed by fish id with a slot pool; the place across the block comes from the core `layer`; `M_Fish` clips every pixel outside the block; a bright strip `WaterRim` with `M_WaterRim` sits on the front water line; the HUD has a hover card for a fish (`catch.Hover <id>` shows it without a mouse); `catch.AutoPull <s>` pulls a rod shortly before an escape, for runs without a player. Simulated clicks (`catch.SimClick`) are flaky: drive your check runs with console commands.
- The core has new data fields since the earlier reports (`Docs/design/core-interface.md`, section 10: the first day of a species, the deepest zone of a bare hook). Their defaults keep the game as it was. Do not set them.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. **One fish mesh for all fish**, by the owner's method: a closed thin convex shell with the outline of look-reference 4.1 at unit length, fins as thin plates, 100 to 150 triangles, side-projected UV. A second mesh for the Giant Squid. Both are written by one script in `Catch/SourceArt/Look/`. No texture per species: species differ by per-instance data (length, height ratio, body colour, glow, lights), as in the reference.
2. **The fish material owns its colour.** It takes the species colour, mixes it toward the water colour at the fish depth by the near/far rule of 4.3, and puts the result on screen mostly as unlit colour. Scene fog, scene lights and the tonemapper must not shift it. A small shading term from the shell normal (at most 12% lighter on top, darker below) gives the volume the owner asked for. Belly, eye, lights and halo follow 4.1-4.2.
3. **Colours are checked with numbers.** Sample pixels in your frames. A near fish body is within 15 (of 255) per channel of its species colour after its own fog rule. The front face of the water at each depth stop of 2.2 is within 15 per channel. Fix the exposure and the tonemapping until this holds.
4. Near/far: the place across the block and the size come from the near/far value p as 4.3 says. This replaces the random place across the block of the first playable version.
5. The turn is an instant mirror flip (4.4). No yaw blend.
6. Above the water stays the dark room with the soft glow (1.1). Keep what the reference says the Unreal scene already does well: light shafts, caustics on the top face, the glassy side wall, the block proportions.
7. Raft, bucket, rods, float, hook, bait: simple meshes by the owner's method with flat or script-drawn textures, made by scripts in `Catch/SourceArt/Look/`. Sizes and motion from section 5.
8. World feedback (splash, rings, bubbles, drops, the hang ring, the hunter ring) uses Niagara or pooled simple meshes, your choice. It must show in a `-game` run and in `catch.Record` frames.
9. Words on screen: English, as the look reference and the rules reference give them.
10. Performance guard: with 1,000 fish the frame stays under 6 ms on this machine (`catch.Bench`).
11. Sound: import the 29 files into `Catch/Content/Audio/`. One new class in `CatchLook*` files plays every sound from the core events by the README rules, through one submix with the compressor. It has a public function that the HUD step calls for its own sounds (menu tick, refused click). The creak value u comes from the hang timer of the hook in the core state. Each feedback row of section 8 gets its sound when you build its picture. Stop using the temporary sounds; do not delete them.
12. Live Coding is switched off for this project on purpose (`bEnabled=False` in `Catch/Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini`). A second executor may build and run the game in another worktree at the same time, and Live Coding would block its builds. Do not switch it on. To rebuild C++: save everything, close the editor, run `Tools/build-editor.ps1 -Project F:\UE\EpicJam\Catch\Catch.uproject`, open the editor again. Group your C++ changes so that you rebuild seldom. If a build waits for a mutex, the other executor is building: wait.
13. Checkpoint. Hand back after block A with the frames `look_7_fish_sheet.png` and `look_1_surface_day.png`, the pixel table for the fish and the frame time, with the editor closed and the gate green. Write the checkpoint into the report file too. The orchestrator checks the frames, shows one to the owner and resumes you for blocks C, B and D.

## 4. Work, in this order

Block A — fish (gap 1, traits 4 and 3).
1. The mesh script, the two meshes, the import script.
2. Per-instance data and the material: species colour, true length and height ratio (4.2), near/far look (4.3), belly and eye only when near, glow halo and lights of the deep species, the Anglerfish lure, the flat Flounder, the Gold Sunfish bloom.
3. Motion (4.4): tail beat and body sway with the rates of the table.
4. States (4.5): goes to a hook, nibble, on the hook (nose up, swing, fast tail), Cluster Hook extras, hunter strike with its ring and yellow eye, escape, leaving, hovered (if the hover card of the integration step exists, add the white silhouette; do not rebuild the card).

Block C — the bite and the hang in the world (gap 2, trait 6). Line, float and its dips (5.4), rod states (5.3, 5.7), the hang ring and the escape warning in the world (6.6), and the world part of every feedback row of section 8 from cast to escape: splash, rings, bubbles, shakes, the trembling line, the rod pump.

Block B — water and atmosphere (gaps 7 and 8, traits 1-3). The depth ramp of 2.2 on the front face and at 62% on the side face, zone lines and names (3.8), the veil of locked depth (3.9), the seabed (2.3, 3.6), the surface with wave dashes and the bright water line (3.1), marine snow and plankton lights (3.3), bubbles (3.4), plants on the walls (3.5). Ambient animals (3.7) only if time is left.

Block D — the catch (gap 5, trait 7). The raft across 77% of the sea (5.1), the wooden bucket with tilt, squash, slosh and drops (5.2), the flight of every fish on its own spinning arc and the landing rows of section 8, the world part of shipping (fish leave the bucket, coins fly). Popup words in the world only if the HUD executor has not taken them; ask the orchestrator through your report if unsure and leave them.

If time runs out, hand back complete blocks. Do not leave a block half done.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` is green with the editor closed.
2. Frames in `Catch/Saved/ue_mcp/`, 1920x1080, from a `-game` run: `look_1_surface_day.png`, `look_2_hooked_ring.png`, `look_3_hunter_strike.png`, `look_4_flight_bucket.png`, `look_5_twilight.png` (camera near depth 500), `look_6_veil.png` (the border of locked depth), `look_7_fish_sheet.png` (all 15 kinds near the camera in one frame; a debug command may place them).
3. A recording `look_clip` (12 s, 30 fps, `catch.Record`) of one stretch of play: bite, hang with the ring, pull, flight, bucket.
4. The table of pixel samples of decision 3 in the report, with the numbers.
5. `catch.Bench` with 1,000 fish: the frame time in the report, under 6 ms.
6. You compared each of your frames with the text of the reference yourself and wrote what still differs.

## 6. Boundaries

- Files you may create or change: `Catch/Source/Catch/CatchSeaView.*`, `CatchTackleView.*`, `CatchWorld.h`, new files `Catch/Source/Catch/CatchLook*.{h,cpp}`, `Catch/Content/Sea/**`, `Catch/Content/Maps/Sea.umap`, `Catch/Content/Audio/**`, `Catch/SourceArt/Look/**`, `Catch/SourceArt/SceneScripts/**`, `Docs/briefs/reports/day1-look-sea.md`.
- Not yours: `CatchHud.*`, `CatchPlayerController.*`, `Catch/Content/UI/**` (the HUD step), `CatchSession.*` (if you need a new debug command or an accessor there, add it in a `CatchLook*` file, or write the need into the report), `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/SourceArt/Sound/**`, the documents.
- You do not invent the game or the look. A number the reference does not give: write the question, choose the plainest reading, mark it in the report.
- Nothing from outside: no downloaded asset, font, texture or sound.
- Old assets stay: do not delete the temporary fish or the old materials. Stop using them.
- Do not commit, do not push, do not delete files. Close the editor before you hand back, with everything saved.

## 7. Report

English, `Docs/briefs/reports/day1-look-sea.md`, at most 35 lines: blocks done; the pixel table; the frame time; what still differs from the reference, most important first; decisions you changed; needs outside your files; questions.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Look questions the reference does not answer: write them down; the orchestrator asks the owner.

## 9. Checkpoint 4 (12.11, 15:03): the scales, the hover from the HUD, then block B

State. The orchestrator verified checkpoint 3 (gate with the editor build, console day 1, six frames) and committed it as `007230d`. Main now also holds HUD block D (merge `1fa9b3e`): depth gauge, bait menu, bait box, hint bar, selling bar and the painted hover card. The old "Open Small Pack" button is gone.

Answers to your questions and notes.
1. Rod: add a taper (the tip thinner than the handle). No reel: the reference has none.
2. The pull length is a core number. The owner is asked. Change nothing.
3. `Packaged/look_test_s1` stays: files are deleted only at the owner's request.
4. FXAA is accepted. The first-launch setting is accepted as plausible, not proven.
5. Leave the SequencerScripting warnings.

Work, in this order.

A. Hover. `UCatchHud::GetHoverFish()` (`CatchHud.h:75`) returns the id of the fish under the cursor, or -1. Draw the white silhouette of block A item 4 on that fish. Read only; the card itself belongs to the HUD.

B. The scales. A new owner decision, `scales-catch-against-norm` in `Docs/decisions/game-frame.answers.json`. The owner's words (Russian, verbatim): «стоило бы где-нибудь слева экрана или там где-то, ну, не может не слева, но в интересном, хорошем месте положить весы какие-то, которые бы показывали, какая сторона перевешивает. То есть нам же надо набрать определенную награду. Грубо говоря, слева будет вес в очках, а справа будет вес, который нужно достичь, чтобы пройти уровень.» The owner did not name the place. The place below is the orchestrator's proposal; the owner judges it on your frame. So build the scales as one object that can move as a whole.
   1. A two-pan balance stands on the raft deck at its left end, where the bucket stands now: a post, a beam, two pans on short chains. Wood and dark metal, in the style of the bucket.
   2. The bucket stands on the left pan and keeps all its behaviour (tilt, squash, slosh, drops, tails). The right pan carries one weight block (cast iron, a ring on top). The block grows with the fee of the day: scale = 0.8 + 0.12 x log2(fee / 15), held between 0.8 and 2.0.
   3. The two numbers: the value of the whole catch of the day so far (it must not fall while the catch ships) and the fee of today. Take them from the session facts that `UCatchHud::MoneyFacts()` reads (`CatchHud.h:317`). If you need an accessor, put it in a `CatchLook*` file.
   4. Beam angle from r = catch value / fee. r from 0 to 1: the weight side is down, angle = 10 degrees x (1 - r). r = 1: level. r above 1: the catch side is down, angle = 10 degrees x min(1, (r - 1) / 0.5).
   5. Motion. The beam follows its target as a damped spring: it overshoots and settles in about 0.6 s. Every fish that lands in the bucket kicks the beam. When r crosses 1 for the first time in a day: one larger swing, a flash on the beam and a ring of drops. No text and no digits in the world: they belong to the HUD.
   6. In the prep phase of a new day the weight block takes its new size with a small drop onto the pan.
   7. Fish fly into the bucket and leave it while shipping as now. The flight target follows the bucket on its pan.
   8. The scales are inside the frame at 16:9 and at 16:10, and they do not hide the first rod. If the left end is crowded, move the bucket and the scales, not the rods.

Hand back after A and B (checkpoint 4a): the gate, the frames `look_8_scales_empty.png` (r = 0), `look_9_scales_level.png` (r near 1), `look_10_scales_over.png` (r above 1.5), `look_11_hover.png`, and a 6 s clip `look_clip_scales/` in which the catch crosses the fee. The owner waits for this view.

C. After the orchestrator's answer: block B as section 4 gives it, the water ramp first. Then the rod taper, the rod hover state and the grow-in of a new rod.

Rules as before (section 6). Use `-Multiprocess` on game runs. Only one build at a time: the orchestrator does not build in main while you work. Close the editor before you hand back. No commits.

## 10. Checkpoint 5 (12.11, 15:33): two touch-ups of the scales, then block B

State. The orchestrator verified checkpoint 4a (gate with the editor build, console day 1, frames of the scales at r = 0 and r = 1.53, the hover frame) and committed it as `18b4d15`. The owner confirmed the place: on the raft, the bucket on the left pan.

Work, in this order.

A. The scales, two touch-ups.
   1. Rod 4. Its base (x 637) stands between the post and the left pan. Do not move rods: their places are core data. Lengthen the deck at its left end into a small weighing platform (same planks), and move the scales with `catch.LookScalesAt` values of your choice so that the right pan clears the base of rod 4 by at least 20 cm and the left pan stays over the top face of the water block. Make the new place the default.
   2. Contrast. The iron block (#2b2e33) and the right pan almost vanish against the dark sky when the pan is up. Give the block and the pans a lighter iron with a top highlight, so that the weight reads at a glance at 1920x1080. Keep it iron, not gold.
   Frames: `look_12_scales_place.png` (r = 0) and `look_13_scales_over.png` (r above 1.5).

B. Block B of section 4: the depth ramp first, then zone lines and names, the veil of locked depth, the seabed, marine snow and plankton lights, plants on the walls. Frames as section 5 item 2 names them (`look_5_twilight.png`, `look_6_veil.png`), plus one frame per further zone you reach with a debug camera.

C. Rod: the taper, the hover state, the grow-in of a new rod.

Hand back after A and B, or earlier if B grows beyond 90 minutes: then hand back what is complete and say what is left. Frame time with 1,000 fish again in the report (the scales added meshes).

Rules as before (section 6). `-Multiprocess` on game runs. The orchestrator does not build in main while you work. Close the editor before you hand back. No commits.

## 11. Checkpoint 6 (12.11, 19:04): the rod, ambient life, leftovers of block B

State. The orchestrator verified checkpoint 5 B (gate with the editor build, console day 1, frames of the veil, Twilight, the seabed and the locked labels) and committed it as `54f033a`. Frame time with 1,000 fish is 5.4-5.5 ms; that is accepted.

Answers to your questions.
1. Zone lines, names, plants and labels in front of the fish: accepted for now (revocable).
2. Glow halos of locked fish at 18 % under the veil: keep them. They hint at life below.

Work, in this order.
A. Leftovers of block B: the eye dot on glowing ghosts; the side face of the floor band; the lock label of the FIRST locked zone. With the camera at the surface the HUD hint bar covers the padlock of that label (see `look_6_veil.png` and the orchestrator's frame `s7_prep.png`: the bar spans about y 816-906 of 1080, the padlock sits at about y 890-930). If look-reference 3.9 leaves the place free, move the label of the first locked zone so that the padlock, the title and the second line are all clear of the hint bar at 1920x1080 and at 1728x1080. Do not move HUD elements.
B. Block C: the rod taper, the hover state of a rod, the grow-in of a new rod.
C. Ambient life (look-reference 3.7), as far as the reference describes it: no new rules, look only.
D. First-run text. You saw grey blocks in place of text for the first seconds of a fresh `-game` run. Find out whether the text material can be warmed up before the first frame that shows it (for example by drawing the labels once behind the title or hint), and do it if it is cheap. Say what you found.

Frames: `look_20_rod.png` (taper and hover), `look_21_new_rod.png` (grow-in, mid-way), `look_22_life.png`, `look_23_first_lock.png` (prep at the surface, the first lock label visible), plus the bench number.

Hand back after A and B, or after 90 minutes. Rules as before (section 6). The rules core is being converted in another worktree: do not edit `Catch/Source/CatchCore` or `Catch/CoreTests`.

## 12. Checkpoint 6 accepted (12.11, 19:23); the sea look track pauses here

The orchestrator verified checkpoint 6 (gate with the editor build, console day 1, seven frames) and committed it as `81567f7`.

Answers to the two questions.
1. Hovering a rod card in the HUD does not light the rod in the world yet. The HUD files change with the conversion to points and coins; the link comes with that task.
2. The 3-unit tip of the rod: accepted (revocable).

Polish list of the sea look (not assigned; the next executor of the view module takes it when the main path allows):
- `look_22b_shadow.png`: the glow halo of one deep fish near the left wall ends in a hard vertical edge (about x 186 of 1920).
- The label of the first locked zone stands about 5 px above the bottom of the screen at 1920x1080 in the prep phase.
- Not looked at closely: the eye dot of glowing ghosts, the flipper motion of the turtle, the grow-in of a rod after a real purchase.
- `look_24_seabed_side.png`: the front face of the floor band reaches a little past the left glass edge.
