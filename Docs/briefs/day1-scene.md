# Brief: day one, track 2 — the target picture of the sea

Status: drafted before the jam start (12.11.2026, 04:16, owner's clock); refreshed and handed out after the start (12.11.2026, 08:23), when the project existed and built. Executor: Opus, effort high. This executor is the only one who writes in the Unreal Editor.

## 1. Goal

Build the picture the whole game lives in: a block of sea seen from the side with a tilted camera, like a diorama. Show it in two water variants with 300 temporary fish and measure the frame time. The owner picks the look from frames at about 10:00.

Owner decisions behind it: «Срез с наклоном - ок»; the sea is a block, like a diorama; one scene and one screen; depth matters more than width; the game must look 3D though real 3D is not needed everywhere; a fish is a simple model with a side texture and the shader makes it swim; the target is up to 1000 fish; «Главное — ощущение и полировка».

## 2. What exists now

- The project `F:\UE\EpicJam\Catch\Catch.uproject` (Unreal Engine 5.9 at `F:\UE_5.9`, C++) exists since 12.11, 06:10. The editor target builds in about one minute, the game packages in about three minutes, the packaged game starts. Module `Catch` is yours. Module `CatchCore` belongs to track 1: do not touch it.
- What is in the project now: `Source/Catch/Catch.h` (log category `LogCatch`), `Source/Catch/CatchGameMode.h/.cpp` (`ACatchGameMode`: on `BeginPlay` it logs the core version and prints one build-check line on screen; keep the log line, remove the screen line), `Config/DefaultEngine.ini` (startup maps are `/Engine/Maps/Entry` until you set `/Game/Maps/Sea`; `GlobalDefaultGameMode=/Script/Catch.CatchGameMode`; static lighting off; DX12 with SM6). There is no `Content` folder yet.
- The Unreal Editor is open on the project and the link answers (`python -X utf8 Tools/ue_mcp.py status`). You are its only user. Compile C++ with `LiveCodingToolset`. When Live Coding cannot take a change (see the capability document, block D), restart the editor yourself: save all, exit, `Tools/build-editor.ps1 -Project F:\UE\EpicJam\Catch\Catch.uproject`, start again. A restart costs about one minute.
- Track 1 writes the rules core in a separate folder (`F:\UE\EpicJam_core`). `Source/CatchCore` in your folder does not change before 11:00. Track 1 never builds the editor target.
- Units of the game logic: `Docs/design/core-interface.md`, section 3. You do not need the rest of that note today.
- Editor link: `python -X utf8 Tools/ue_mcp.py` (help at the top of the file). `py <file>` runs Python inside the editor. Screenshots go to `Saved/ue_mcp/`.
- What the link can do, with ready calls: `Docs/research/ue59-editor-link-capabilities.md`. Read it first.
- Facts verified on 12.11: an editor frame does not advance time, so particles and shader motion show only in a frame or a recording from the running game; Live Coding does not write to disk, so save all before exit and build from the console before the next start; the editor rejects calls while busy and `Tools/ue_mcp.py` retries.
- Design frame: `Docs/plan.md` section 4 (items 4, 5, 6, 13, 16, 22) and section 7 (fish model). Depth zones and their names: `Docs/design/content-list.md` section 6.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. Light, fog, camera, water and post-processing live in one saved level, `/Game/Maps/Sea`. It is the startup level for the editor and the game. Code creates only fish, tackle and the raft.
2. Camera: fixed, perspective with a narrow field of view, tilted down so that the surface reads as a plane and the cut reads as a wall of water. The camera moves only along depth.
3. World scale: the content list uses abstract units (width 1000, depth 0-3400, zone borders 300, 700, 1200, 1800, 2500). Pick one factor to centimetres and put it in one place in code.
4. Water, two variants to compare by frame and by frame time:
   - A: an opaque cut with a back wall, depth tint and distance fog.
   - B: a thin transparent glass in front, opaque fish behind it.
   Light rays are flat glowing strips, not volumetric fog.
5. Fish: instances of one mesh (instanced static mesh or Niagara mesh renderer; choose by measurement). The swimming is vertex motion in the material, driven by per-instance data. Temporary mesh: a few polygons with a side texture drawn by you after 06:00. Show one frame that compares a flat card with a thin 3D body.
6. Six depth zones as colour and light of the water, from sunlit to black, with the seabed at the bottom. Locked depth is dark; big shadows may move there.
7. Scalability: the look must hold on the Low preset. Lumen and virtual shadow maps are optional; measure with and without.
8. Every asset is made after 06:00 by you (materials, a temporary mesh, a temporary texture). No asset from outside. Write each asset into your report for `DEVLOG.md`.

## 4. Work blocks

1. Level `/Game/Maps/Sea`: sea block, surface, back wall, seabed, raft placeholder, camera, light, fog, post-processing. Startup level set in config.
2. Water variants A and B as two material sets, switchable by one console variable or one actor property.
3. 300 temporary fish in schools across the first three zones, swimming by shader. A console command sets the count (300, 1000) for the measurement.
4. Camera travel from the surface to the seabed by one console command (the game will drive it later).
5. Frames from the running game for each variant: surface, middle depth, seabed. Frame time for 300 and 1000 fish on the default and the Low preset (`stat unit` values or the capture toolset).
6. Report with the frames and one recommendation.

## 5. Done means

1. The packaged-style check passes: `Tools/build-editor.ps1 -Project F:\UE\EpicJam\Catch\Catch.uproject` builds; the level opens at editor start.
2. Six frames (two variants, three depths) and one flat-versus-thin fish frame lie in `Catch/Saved/ue_mcp/` and are listed in the report. You looked at each frame yourself.
3. A table of frame times: variant, fish count, preset.
4. `python -X utf8 Tools/check_all.py` is green (English only in the project; no build folders in git).

## 6. Boundaries

- You do not invent the game: no rules, no numbers of play, no screens. Scene and look only.
- Do not touch `Catch/Source/CatchCore/`, `Tools/`, `Docs/` (except your report) and the folder `F:\UE\EpicJam_core`.
- Frames go to a folder that git ignores (`Saved/ue_mcp/` or `Catch/Saved/ue_mcp/`). Write the real paths into the report.
- Player-facing text is English. No logos except the Unreal Engine logo.
- Do not commit. Do not delete files. Unused generated files do not go into `Content`.
- Save everything before the editor closes.

## 7. Report

English, in `Docs/briefs/reports/day1-scene.md`, at most 40 lines: what exists, the frame list, the frame-time table, your recommendation for the water and the fish body, every asset you created and how, questions.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Look-and-feel questions for the owner: write them down; the orchestrator shows him frames.
