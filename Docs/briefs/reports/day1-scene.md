# Report: day one, track 2 — the target picture of the sea (12.11.2026, done 09:00)

**What exists.** `/Game/Maps/Sea` is the editor and game startup level (verified at two editor restarts). It holds a sea block 2000 x 700 x 6800 cm (front cut, back wall, side walls, surface, seabed with rocks and plants), six flat light-ray strips, a raft placeholder (engine shapes, tag `RaftPlaceholder`), sun, sky light, unbound post volume (manual exposure), a backdrop, `ViewCamera` (FOV 24, pitch -14) and `SeaView` (`ACatchSeaView`). Code (`Catch` module only): `CatchWorld.h` holds the one scale factor (1 game unit = 2 cm) and the x/depth conversions; `CatchSeaView.h/.cpp` builds 300 temporary fish as one ISM (6 custom floats per instance; travel, tail wiggle, bob and edge fade are all in `M_Fish` WPO), switches water A/B (MPC_Sea `WaterB` + actors tagged `WaterA`/`WaterB`), moves the camera along depth, measures frame time and takes the frames. `ACatchGameMode`: screen line removed, spectator pawn. Build.cs gains `RenderCore`, `RHI`.
Console: `catch.Water A|B`, `catch.FishCount N`, `catch.FishBody 0|1|2`, `catch.FishCompare 0|1`, `catch.CameraDepth <units> [s]`, `catch.CameraDive [s]`, `catch.Measure [s]`, `catch.Bench [quit]`, `catch.Shots [quit]`.
Zones are colour only (one depth function in every material): Sunlight teal, Twilight blue, Midnight dark blue, Abyss/Trench near black, faint lift at the seabed. Below `OpenDepth` (look parameter, 1200) the water is darker; 3 big dark shadows swim there; 70 small lights drift below 760.

**Frames** (1920x1080, from the running game, `UnrealEditor.exe -game`; I looked at every one). Folder `F:\UE\EpicJam\Catch\Saved\ue_mcp\`:
`sea_A_surface.png`, `sea_A_middle.png` (camera depth 760), `sea_A_seabed.png`, `sea_B_surface.png`, `sea_B_middle.png`, `sea_B_seabed.png`, `fish_card_vs_body.png` (left column flat card, right column thin body; top row side-on, bottom row turned 35 deg), extra: `sea_A_surface_1000fish.png`, `sea_A_surface_low.png`, `sea_B_surface_low.png`, `motion_A_t0.png` + `motion_A_t1.png` (0.5 s apart: fish and surface lines moved; mean pixel change 2.9). Table: `frame_times.csv`.

**Frame time** (ms, mean over 3 s, 1920x1080 window, vsync off, editor closed; RTX 5080, i7-8700K; editor binaries in -game mode, not a packaged build):

| Water | Preset | Fish | Frame | Game | Draw | GPU |
|---|---|---|---|---|---|---|
| A | Default (Epic, Lumen + VSM) | 300 / 1000 | 4.33 / 4.30 | 1.93 / 1.90 | 4.32 / 4.29 | 1.81 / 1.84 |
| B | Default | 300 / 1000 | 4.33 / 4.37 | 1.90 / 1.90 | 4.32 / 4.36 | 1.81 / 1.84 |
| A | Low | 300 / 1000 | 3.26 / 3.31 | 1.74 / 1.75 | 3.25 / 3.30 | 0.79 / 0.81 |
| B | Low | 300 / 1000 | 3.24 / 3.26 | 1.72 / 1.74 | 3.23 / 3.25 | 0.79 / 0.81 |
| A / B | Default, no Lumen, no VSM | 1000 | 4.58 / 4.32 | 2.02 / 1.95 | 4.57 / 4.28 | 1.57 / 1.57 |

1000 fish cost nothing measurable; A and B cost the same; Low keeps the look (most materials are unlit).

**Recommendation.** Water A: stronger zone colours, clearer fish, brighter rays; B's glass hazes the middle depth. Option: borrow B's bright rim line at the water line into A. Fish: the thin 3D body (286 triangles vs 96): it stays round when it turns, cost is zero. ISM, not Niagara: ISM is already far under budget; I did not build a Niagara version to compare.

**Decisions I took (revocable).** Mesh made by a Python script, not Blender, written as OBJ, not FBX/GLB (the Blender link was down; OBJ imports cleanly). Unreal is left-handed: with the camera looking along +Y, screen-left is world +X, so `XToWorld` maps game x = 0 to world +1000 cm. Fish cast no shadows (all water surfaces are unlit). The raft placeholder lives in the level until code spawns the real raft. Shots and bench run in a separate `-game` process with the editor closed: the first such run with the editor open coincided with the editor process vanishing (no crash report, 08:42).

**Assets (all made 12.11 after 08:25 by this agent, nothing from outside; for DEVLOG.md).**
- `T_TempFish` (512x256 RGBA), `SM_TempFishCard`, `SM_TempFishBody` in `/Game/Sea/Fish`: drawn and modelled procedurally by `Catch/SourceArt/TempFish/make_temp_fish.py` (Python, numpy, Pillow); sources next to it.
- `MPC_Sea`, `M_Fish`, `M_BackWall`, `M_Glass`, `M_Surface`, `M_LightRay`, `M_Seabed`, `M_Rock`, `M_Plant`, `M_RaftTemp`, `M_Backdrop` in `/Game/Sea/Materials`: built by editor Python with Custom HLSL nodes, `Catch/SourceArt/SceneScripts/build_sea_materials.py`.
- Level `/Game/Maps/Sea`: `Catch/SourceArt/SceneScripts/build_sea_level.py`; geometry is engine basic shapes (`/Engine/BasicShapes` Cube, Plane, Sphere, Cone, Cylinder).
- Tools: Unreal Editor 5.9 with the MCP editor link (`Tools/ue_mcp.py`), Live Coding, Python 3.

**Questions for the owner.** 1) Water A or B? 2) Above the block the surface view shows a dark studio void; keep it or show a sky? 3) Below Midnight the water is almost black; keep or lift? 4) Fish now measure 2.5-4 % of the sea width on screen; bigger or smaller?
