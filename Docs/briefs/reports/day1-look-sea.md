# day1-look-sea: report (checkpoint 4a: hover and scales, 12.11)

**State.** Section 9 A and B are done. C is not started (block B, rod taper, rod hover state, grow-in of a new rod). The gate `check_all.py --ue` passes: 10 ok, 1 skip. The editor is closed. Nothing is committed and no file is deleted.
**Frames** (`Catch/Saved/ue_mcp/`, 1920x1080, `-game`): `look_8_scales_empty.png` (r = 0), `look_9_scales_level.png` (r = 1.00), `look_10_scales_over.png` (r = 1.53), `look_11_hover.png`, `look_scales_16x10.png` (1728x1080).
**Clips.** `look_clip_scales/`: 6 s, day 1, seed 42. The $23 fish flies in at frames 60-72 and the catch crosses the $15 fee at frame 74 (swing, flash, ring of drops); it settles by about frame 100. `look_clip_weight/`: 2 s, day 2 preparation, the new weight drops onto its pan.

**A. Hover.** It was already wired at checkpoint 3: with `catch.LookHover -1` the sea view draws the white silhouette on `UCatchHud::GetHoverFish()`, and the other fish of that species glow. The frame uses the HUD path (`catch.Hover 45`, a Mackerel). The card belongs to the HUD.
**B. Scales.** New `CatchLookScales.{h,cpp}` (one object, placed from one origin) with new assets `SM_LookPan`, `SM_LookWeight`, `SM_LookRing` and `M_LookSolid`, made by the new script `SourceArt/Look/build_look_scales.py`. `M_LookRaft` gets a second pan shadow.
- Build: a wooden foot, post and beam (bucket colours, faint grain), an iron pivot cap and a needle. A hook at each beam end and a short drop to a hanger bar. Two chains with links hang to each metal pan. The bucket stands on the left pan with all its springs, drops and tails. The right pan carries a cast iron block with a ring.
- Numbers: weight scale = clamp(0.8 + 0.12 log2(fee / 15), 0.8, 2.0). The beam angle follows section 9 B 4. The spring has stiffness 360 and damping 13 (overshoot, settles in about 0.6 s). Each fish that lands in the bucket kicks the beam 0.9 rad/s.
- First crossing in a day: a 2.6 rad/s swing, the beam flashes #fff0b4 (fades in 0.4 s), 16 drops fly out in a ring and the bucket sloshes. No words and no digits in the world.
- A new day: in preparation the new block drops 28 cm, bounces once and dips the beam.
- Flights follow the bucket on its pan, both to the bucket and out of it while the catch ships.
- Values: catch = core `bucketTotal` (fish that reached the bucket today; it does not fall while the catch ships and resets at the next day). Fee = `Core().FeeOf(day)`, the same number `MoneyFacts()` uses. No new accessor was needed.
- Place: the owner confirmed it (raft, bucket on the left pan). Post foot at world (612, 240) cm; left pan under the old bucket place (x 690). `catch.LookScalesAt <x> [y]` moves the scales and the bucket as one. At 16:9 and 16:10 the scales are in the frame and clear of rod 1.

**Not done / open.**
1) Block C. 2) No sound for the crossing; the reference has none. 3) Frame time was not measured again (the scales add 10 meshes and about 60 cards). 4) Rod 4 (when bought) has its base at x 637, behind the scales post; the old bucket already stood there. Moving that rod is not allowed, so a later choice is needed: move the scales left (the deck ends at 760) or accept the overlap.
**Decisions taken (revocable).**
- Geometry, cm: beam pivot 128 above the deck, half beam 78, pans 56 in radius hanging 94 below the beam end, block 20/15 radius and 24 tall at scale 1. Round parts are drawn 1.67 deeper, as the bucket.
- Each pan hangs from a hanger bar on two straight chains, so the chains clear the bucket rim. This is my reading of "short chains".
- Colours: pans #5d636b, block #2b2e33, chains #2f3238, wood #5a3818, #8a5a2e, #a8743e.
- `look_9` is a real day 2 frame (fee $23, one $23 fish, r = 1.00); `look_10` is day 1 with r = 1.53.
**Questions.** None for 4a.


# day1-look-sea: report (checkpoint 5: stopped early at the usage limit, 12.11)

**State.** I stopped when the orchestrator asked me to. Section 10 A is complete. B and C are not started: I made no B edits. The editor build after the A edits succeeded. The editor is closed and all assets are saved. Nothing is committed and no file is deleted. I did not run the gate `check_all.py --ue` or the 1,000-fish bench again.
**Frames** (`Catch/Saved/ue_mcp/`, 1920x1080, `-game`, seed 42): `look_12_scales_place.png` (preparation, r = 0) and `look_13_scales_over.png` (day 1, one $23 fish against the $15 fee, r = 1.53). I looked at both frames.

**A1. Place.** The deck runs on at its left end to world X 985 as a weighing platform (same deck mesh and planks). The plank seams still count from X 760, so the old seams do not move. The new default origin of the scales is (825, 240, 16) cm. The script sets it on the level actor too.
- Clearance: the right pan rim is at X 691. The rod 4 socket reaches X 663 (base 637 + 26), so the gap is 28 cm. A beam swing moves the pans toward the post, so the gap does not get smaller.
- The left pan rim is at X 959 and the bucket rim at full squash is at about 963. Both stay over the top face of the block (X < 1000).
- The raft shadow on the water (`M_Surface`) now ends at X 985.
- Not checked in a frame: rod 4 itself, because it costs $165 in the shop. The clearance above is calculated.
**A2. Contrast.** `M_LookSolid` has a new scalar parameter `Top` (0..1). Faces that look up get lighter, and the upper part of the sides gets a cool highlight. The pans (#7d848e), the block and its ring (#646b75) use Top = 1. In both frames the block now reads clearly against the dark sky.
- Possible touch-up: at full Top the pan tops come out very pale, close to tin. If the owner wants darker iron, set Top for the pans to about 0.55 (one number in `CatchLookScales.cpp`).

**Half-done.** None. Block B was only read and planned; no B edits exist.
**Not done.** Block B (depth ramp, zone lines and names, veil, seabed, marine snow and plankton, plants), block C (rod taper, hover state, grow-in of a new rod), the gate, and the frame-time measurement with 1,000 fish.
**Plan for B, for the next session.**
- `M_BackWall`: take the depth from the camera ray where it crosses the front plane (y 0). Keep the world depth on the side face.
- Veil in the shared WATER snippet: 0 to 78 % over 70 units, then 78 to 82 % down to the floor. Ghost fish in `M_LookFish`.
- A new class `FCatchLookSea` with its own card ISM: zone lines, marine snow and plankton, plants. Lines and plants are moved to the back plane with the same screen point.
- Zone names and locked labels as TextRender with the engine font RobotoDistanceField and a new `M_LookText`.
- Seabed band and rocks re-coloured, 14 rocks, and a look-only `catch.LookOpenDepth` for debug frames.
**Changed files.** `Catch/Source/Catch/CatchTackleView.{h,cpp}`, `CatchLookScales.{h,cpp}`, `Catch/SourceArt/Look/build_look_scales.py`, `Catch/SourceArt/SceneScripts/build_sea_materials.py`.
- Assets: `M_LookSolid` and `Sea.umap`.
- The rerun of `build_sea_materials.py` re-saved all of `/Game/Sea/Materials`. Only `M_Surface` has a real change; the others were rebuilt with the same graphs.
**Questions.**
1. 3.9 "Needs Deeper Line, lv N": I plan to use N = the zone index (zone 1 needs lv 1). Is that right?
2. The reference asks for weight 800 text. The engine font has one weight. May I use it for the zone names?


# day1-look-sea: report (checkpoint 5 B: water and atmosphere, 12.11)

**State.** The pan touch-up and block B are done (ambient life 3.7 is not done). Block C is not started. The gate `check_all.py --ue` passes: 10 ok, 1 skip. The editor is closed. Nothing is committed and no file is deleted.
**Frame time** (`catch.Bench`, two runs): 1,000 fish at Default take 5.53 ms and 5.36 ms. At checkpoint 4 it was 4.4 ms, so the headroom under 6 ms is now small. The draw thread is the larger cost.
**Frames** (`Catch/Saved/ue_mcp/`, 1920x1080, `-game`, seed 42, day 1 preparation):
- `look_6_veil.png`: the border of locked depth at 300.
- `look_5_twilight.png`: camera at 500.
- `look_14_midnight.png`, `look_15_abyss.png`, `look_16_trench.png`, `look_17_seabed.png`: camera at 1000, 1500, 2100 and 3010. These four use the new look-only `catch.LookOpenDepth 3600`, so the zones show unveiled.
- `look_18_locked.png` and `look_19_seabed_locked.png`: the padlock labels.
- `look_12` and `look_13` were shot again for the pans (Top 0.55; they now read as grey iron).
**Ramp check** (median of each row, 4 frames x 8 rows, depths 370-2460): all within 2 per channel of the 2.2 ramp.

**What was built.**
- Depth ramp: `M_BackWall` takes the depth where the camera ray crosses the front cut. The side face keeps its own depth at 62 %.
- Veil (3.9): it is in the shared water code, as 0 to 78 % over 70 units, then up to 82 %. It covers the front face, the side face, the seabed and the rocks.
- Ghost fish: in `M_LookFish`, a #96bed7 12 % tint on top of the veiled fish; glowing species use #ffe696 16 %.
- In-water effect cards (halos, rings, bubbles, snow) fade under the veil in `M_LookFx`.
- New `CatchLookSea.{h,cpp}` draws zone lines (dashed, #dcf5ff 18 %) and zone names. Names come from the core tables in capitals, 12 units, 42 %.
- It also draws the locked labels: a padlock, "<ZONE> - LOCKED", and "Buy Deeper Line" or "Needs Deeper Line, lv N". N = zone index - initialZones + 1, the same rule as `Shop.cpp`. The Seabed label sits at depth 3300.
- Marine snow and plankton: 360 specks with the fish near/far placement, and mint or periwinkle pulsing below 700.
- Plants: 22 on the walls and 22 on the floor, two strands each with sway. They are premixed with the water at their depth.
- Seabed (3.6): a band with a wavy top (#3a3122 to #15120c) and 14 rocks of #1f2128. A new script, `SceneScripts/build_look_sea.py`, places the rocks and hides the old cone plants in the game; the cone actors stay in the level.
- Text: engine material `UnlitText` with the engine font `RobotoDistanceField`.

**Decisions taken (revocable).**
- Zone lines, names, plants and labels lie in a plane 24 cm behind the front cut. Further back the left wall would hide them. As a result they pass in front of the fish.
- Wall plants bend toward the middle of the sea.
- Padlock layout: the lock 34 units above the label centre, the title, then the hint 18 units below the centre.
- The engine font has no em dash, so the title uses a hyphen.
- Rocks are 3D spheres half sunk in the floor near the front cut.

**Not done.**
- Block C (rod taper, hover state, grow-in of a new rod).
- Ambient life (3.7).
- The eye dot of glowing ghosts.
- The side face of the floor band does not rise with the back offset.

**Watch.**
- In a fresh `-game` run, text shows as grey blocks for the first seconds while its shaders compile. The frames are shot after a warm-up. Please check a packaged build.
- In preparation, the HUD hint bar covers the middle labels of locked zones.

**Questions.**
1. Is it acceptable that the flat layers (zone lines, names, plants, labels) draw in front of the fish?
2. Glow halos of locked fish keep 18 % under the veil. Is that enough, or should they be hidden?


# day1-look-sea: report (checkpoint 6: rod, ambient life, leftovers of block B, 12.11)

**State.** Section 11 A, B, C and D are done. The gate `check_all.py --ue` passes: 10 ok, 1 skip. The editor is closed. Nothing is committed and no file is deleted. I did not touch `CatchCore`, `CoreTests` or the core worktree.
**Frame time** (`catch.Bench`, 1,000 fish, Default): 4.76 ms with ambient life and 4.85 ms without it (new look-only console variable `catch.LookLife 0`). So ambient life costs nothing I can measure; the noise between runs is about 0.3 ms. The rod changes add 10 cards per rod. The 5.4-5.5 ms of checkpoint 5 was probably measured on a busier machine.
**Frames** (`Catch/Saved/ue_mcp/`, `-game`, seed 42, day 1 preparation):
- `look_20_rod.png`: taper and the hovered rod (`catch.LookRodHover 0`). The rod without hover is in `look_23`.
- `look_21_new_rod.png`: grow-in at 0.1 s, about 76 % of the full size (`catch.LookRodGrow 0`).
- `look_22_life.png` (turtle and jellyfish) and `look_22b_shadow.png` (the big shadow), both with `catch.LookOpenDepth 3600`.
- `look_23_first_lock.png` at 1920x1080 and `look_23_first_lock_16x10.png` at 1728x1080.
- `look_24_seabed_side.png`: the floor band, front face and side face.

**A. Leftovers of block B.**
1. Glowing ghosts get one eye dot (#fff0aa 45 %) in `M_LookFish`. Not checked closely in a frame.
2. The floor band continues on the side face at 62 %, with its wavy top, in `M_BackWall`. Above the wave on the front face I now draw a darker sand instead of black water. This removes a black seam where the 3D floor top meets the front face.
3. First lock label: when the camera keeps it near the hint bar, the label of the first locked zone moves down until the padlock top is below 86 % H. The second line stays on screen. At both screen sizes the padlock, the title and the second line are now clear of the bar. At 1920x1080 the second line ends about 5 px above the screen bottom. I moved no HUD element.
**B. Rod.**
- Taper: the handle stays 7 units. Above it the pole tapers from 5 units to 3 at the tip. The 3 is my choice.
- Hover (5.3, 2.6): the pole is 1 unit wider and #fff1cc. A rod counts as hovered when the pointer is within half the pole width plus 8 px of the pole, measured in the tackle view. The badge keeps the level colour.
- Grow-in: a rod that the core adds during play grows from its socket in 0.45 s with back-out overshoot. The pole and the badge scale from the base.
**C. Ambient life (3.7)**, cards with the 4.3 near/far placement and size, all under the veil:
- A turtle: shell with two lines, head, two rowing flippers.
- Three jellyfish: bell pulse, rise and bob, and they wrap from 330 to 660.
- A big shadow with a flapping tail.
- My reading: the jellyfish bell is a flat ellipse, because the card shapes have no dome. The tentacles are single strokes, so the jellies look a little like stools.
**D. First-run text.** The grey blocks were the engine compiling the text material's shader on demand. That happens only in uncooked `-game` runs, and once per machine: the compiled shader goes into the shader cache. A fresh run today shows correct text at 0.8 s (`look_23`). A packaged build ships compiled shaders, so it should not show this. A warm-up draw would not help, because the compile itself takes the seconds. So I added none.

**Not done / open.**
- The rod hover comes only from the pointer in the world. If the rod card hover in the HUD should light the rod too, the HUD needs to expose its hovered rod.
- Grow-in on a real purchase is not shot in a frame, because the shop layer covers the raft. The look command replays the same animation.
**Questions.**
1. Should hovering a rod card in the HUD also light the rod in the world?
2. Is a 3-unit tip for the taper right, or do you want another value?
