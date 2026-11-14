# Addendum 3 to brief day3-video-draft: draft 3 after the director's five notes (14.11.2026 14:58)

Executor: the same Opus executor, effort high. The brief, addendum 1 and the pointer rules still hold. This addendum replaces the shot plan of addendum 2 (its sections 3, 4.6, 4.7 and 5a). You do not commit.

## 1. The director's words (14.11, after watching draft 2)

"очень странные ролики. во первых монтаж резкий - музыка обрывается и опять играет с рандомного места. По идее у нас должна быть одна звуковая дорожка на всё. Во вторых вместо нарастания безумия у нас потом опять спокойный геймплей врезан. В третьих у нас не очень разнообразные кадры. нам нужно всего лишь один раз показать магазин (а у нас он показывается несколько раз). В четвёртых у нас всего лишь одна глубина. В пятых у нас только один вариант безумия, а должно быть много неулучшенных удочек дёргают рыбу. Одна удочка с  супердлинной леской (крючки до самого низа). Гарпуны, взрывы, цепочки."

The orchestrator's reading, and its own mistake: "a combination of charms" did not mean one build. The madness is several different builds, each a different way to play.

## 2. What changes

1. **One music track.** The whole video has one continuous music bed: the game's own music, `F:\UE\EpicJam_ui\Catch\SourceArt\Sound\S_Music.wav` (check that it is the track the build plays; nothing from outside the game). Record every take with the game's music at 0 so the takes carry effects only: the settings screen has music steps (`catch.HudClick` with the hit names `MenuSettings`, `MusicDown`, `SettingsBack` in `CatchHudMenu.cpp:73-74`); if the game stores the setting, put it back to its old value after the last take. Mix: the bed runs from the first frame to the end card without a cut and fades out on the card; the effects of each segment get short fades (30-60 ms) at the cuts; the whole mix to -16 LUFS. Where it costs nothing, put the cuts on beats of the bed.
2. **The escalation never steps back.** Every shot has more happening than the shot before it (more rods, more fish on hooks, bigger pulls, more effects). No sparse shot after a busy one. Order the shots by what is on screen, not by how they were recorded.
3. **The shop appears once**, in part 2, as one continuous shot of at most 5 seconds in which the pointer buys two or three things. After that the build grows without the shop: the shelf in frame shows the new charms.
4. **Depth.** The video shows the sea at three or more different depths. At least once the camera travels from the surface down through the zones to the seabed and the viewer sees how deep the sea is.
5. **Part 4 is a series of different mad builds**, each its own short shot of 2.5-4 seconds, each busier than the one before:
   - wide: five rods of level 1 (the game has five rods at most; do not upgrade them) that pull fish one after another (Auto-Reel, Rod Rack; add what makes bites frequent);
   - deep: one rod with a very long line, five hooks spread down to the seabed (Trotline, Extra Edge, Bottom Rig, Deeper Line to the last zone, Old Faithful); the camera follows the line down and the pull up;
   - chains: Tiny Terror, Fresh Scent, Food Chain, Apex Hunter; CHAIN x5 or more;
   - harpoons: Harpoon in a crowded sea (Big Shoals), the bounces readable;
   - blasts: Dynamite with Cluster Hook and Big Shoals, a blast that sends a crowd into the bucket;
   - finale: the fullest build you can stage with everything firing, the biggest pulls, fish raining into the bucket.
   Parts 1-3 get shorter to make room. Total stays 45-60 seconds, four captions as before, then the end card.
6. **Variety.** No two neighbouring shots with the same framing and the same build. Use depth, the number of rods and different builds.

## 3. The build to record from

`F:\UE\EpicJam_ui\Packaged\v15\Windows` (the orchestrator tells you when it is ready): `catch.CheatRun` then honours `-new`, so Dynamite and Harpoon can be bought in a staged late build; `catch.Tips 0` hides the teaching tips, the day hint bar and the preparation hint. Use `catch.Tips 0` in every take. Part 1 can then be a real day 1 with one rod.

## 4. Done means

- `Saved/video/draft3.mp4` (and `draft3_coldopen.mp4` from the same set, the flash taken from the finale), 45-60 s, 1920x1080, 60 fps, with sheets.
- One unbroken music bed; you listened for clicks at the cuts by looking at the waveform or the levels around each cut, and say what you checked.
- The five points of section 2 hold; list in the report, per shot, the build (charms on the shelf), rods, depth and the busiest event.
- The checks of the brief and of addendum 1 hold; no desktop or notification in any frame (look at the corner sheets yourself this time; a set with a notification is not used).
- The gate is green.

## 5. Report

Append a section "Addendum 3" to `Docs/briefs/reports/day3-video-draft.md`: the shot table, how the bed and the effects were mixed, how each build was staged, what you could not stage and why, what is not verified. Time box: about 90 minutes.
