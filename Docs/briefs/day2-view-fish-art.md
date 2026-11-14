# Brief: view, part 3 - the fish pictures in the game: in the sea, on cards, moving by shader

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 14:00 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message. This brief lives only in the main checkout.

## 1. Goal

Every fish kind gets its picture. One picture per kind serves twice: as the look of the fish in the sea, where a shader makes it swim, and as the picture on cards, hover cards and the fish table.

Why it matters. The owner (game director):

- 13.11, playtest of build 7: «Для рыбки тоже было бы неплохо иллюстрацию. И да, я все еще не вижу этого иллюстрации рыбы».
- 13.11, about 12:50, on the sheet of the 15 pictures, to the question "Do we take this style into the game?": «да пожалуйста, главное чтобы они шевелились (шейдером сделаешь?».
- 13.11, about 12:55: «можно ли как-то сделать, чтобы не только хвостовой плавничок шевелился, например, глаза. Это не обязательно, просто если это легко сделать на раз-два, то прикольно было бы. А если это сложно, ну, то не надо.»
- The frame of the game: the fish is a simple model with a side texture; the shader makes it swim; the target is up to 1000 fish; the game must look like 3D; every action answers with motion.
- He judges every frame against a browser prototype he liked; a placeholder look must be labelled as such in the report.

Decisions on record: `fish-illustrations`, `fish-art-style-accepted`, `fish-eyes-move-if-cheap`, `fish-kinds-get-patterns`.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\`.

- The pictures: `Catch/SourceArt/Fish/` (the orchestrator put them there; nothing references them yet). 15 files `<Kind>.png` (RGBA, cut out, trimmed, longest side 1024), `atlas.png` (2048x2048, 4x4 cells of 512, each fish fitted into its cell with a 4 px margin and centred), `atlas.json` (per kind: `index`, `cell`, `content` rectangle in atlas pixels, `size`, `aspect`). Order by index: Sprat, Mackerel, SeaBass, Lanternfish, Cod, Tuna, Hatchetfish, Anglerfish, Oarfish, GulperEel, Snailfish, GiantSquid, Flounder, Coelacanth, GoldSunfish. Every head points to the right. The Giant Squid has its mantle on the left and its arms on the right. The tool that made them is `Tools/fish_cut.py` in the main checkout; you may copy it into this worktree's `Tools/` and rerun it with other sizes or margins if the material needs it (say so in the report).
- The fish in the sea: `Catch/Source/Catch/CatchLookFish.cpp`. Instanced static meshes (`Init(InFish, InSquid, InFx)`), per-instance custom data (`SetCustomData`, `D_Count` floats: facing, swim rate and more), the fish kinds, a special ratio for the Flounder. Part 1 put the drawn mouth of each fish on the core mouth point (report `Docs/briefs/reports/day2-view-menu-fixes.md`, block 2, with the probe `catch.BiteProbe` and the cheat `catch.BiteRun <seed> <fish>...`).
- The fish on the interface: `Ui::PaintFishPicture(kind, rect)` draws a painter fish on chum tiles, fish cards, the shop hover card, the sea fish hover card and the fish table.
- How the project makes meshes, materials and textures today (assets under `Catch/Content`, or built in code): find out first; `Docs/research/ue59-editor-link-capabilities.md` lists what an agent can do in the editor.
- Reports of the two earlier view parts: `Docs/briefs/reports/day2-view-menu-fixes.md`, `Docs/briefs/reports/day2-view-core23.md` (proof modes of `run_text_proof.py`, cheats, the build note `bUseUnity = false`).

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. One atlas texture and one material for all fish kinds in the sea. The kind picks its cell through per-instance custom data. The alpha of the picture cuts the fish out (masked, with a soft edge if the cost is small). The existing depth tint, fog and light of the sea keep working on the picture.
2. The swim is made by the material. Preferred: bend the picture in the pixel shader (move V by a sine that runs along U, zero at the head, largest at the tail), so a plain quad is enough and 1000 instances stay cheap; keep a transparent margin in the quad for the bend. If the fish mesh already has enough segments for a vertex bend, a vertex bend is fine too. Phase and rate come from the per-instance data that exists.
3. Per-kind swim data in ONE table (amplitude, wave count along the body, rate, where the wave starts). Special kinds: the Oarfish and the Gulper Eel carry two or more waves along the body; the Giant Squid keeps its mantle steady and waves its arms; the Flounder ripples its fringe and stays flat; the Gold Sunfish is tall and flaps its upper and lower fin; the Anglerfish may sway its lure. Every kind must move visibly at the size it has in the sea.
4. Size and shape: the length of each kind stays what the game uses today; the height follows the `aspect` of the picture. The picture mirrors when the fish swims left. The mouth of each picture (measure it per kind, near the right end of the content) sits on the core mouth point; the bite must stay at the hook.
5. Everything that shows a fish uses the picture: the hooked fish, the fish that flies to the bucket, the shipping, the sea hover card, the shop tiles, the fish cards, the fish table. `Ui::PaintFishPicture` draws the atlas cell; the painter fish stays in the code as the fallback when the texture is missing.
6. Import. Bring `atlas.png` into `Catch/Content` as a texture (mips on, sRGB, alpha) and build or change the material by a script, not by hand: run the editor headless with a Python script (commandlet), or through the editor link if an editor is needed. Keep the script in `Catch/SourceArt/Fish/` so the import can be repeated. Do not open the editor for anything else.
7. Eyes, optional and last. Only when blocks 1-4 are done and proven: measure the eye centre and radius per kind in the picture, and let the material shift the pupil a little inside the eye disc (a slow look-around, or a look along the swim direction). If it does not work within about 40 minutes, leave it out and say so; do not ask.

## 4. Work by blocks (in this order; extend the report after each block)

1. Find out how fish are drawn today; import the atlas; one fish kind in the sea shows its picture and swims.
2. All 15 kinds in the sea with the swim table; mirror, sizes, mouth points; the hooked fish, the flight to the bucket, the shipping.
3. The interface: `Ui::PaintFishPicture` and every place that calls it.
4. Proof and cost: frames, the bite probe, the frame time with many fish.
5. Eyes (optional).

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- The existing proofs still pass with 0 errors: `run_text_proof.py plain en 1920`, `flow en 1920`, `screens ru 1280`.
- The bite probe: the median gap between the mouth and the hook stays under 3 units for Sprat, Sea Bass and Giant Squid (`catch.BiteRun 42 Sprat SeaBass GiantSquid`).
- Motion proof from a RUNNING game (an editor frame does not move time): for every kind, two frames about 0.3 s apart, put side by side in one sheet; the body shape differs between the two. Frames you looked at yourself.
- Frames you looked at yourself, 1920x1080: the sea with several kinds in one frame, a hooked fish, a fish in flight to the bucket, the shop with fish cards and chum tiles, the fish table, the sea hover card; one frame at 1280x720.
- Cost: the frame time (or frames per second) with about 1000 fish before and after, measured the same way; say how.
- The report lists every asset added under `Catch/Content`, with its size on disk, and every place that still shows the old painter fish.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`; another executor builds the core in `F:\UE\EpicJam_core`; do not kill a process you did not start.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, new assets under `Catch/Content/` for the fish (the atlas texture, the fish material, a mesh if you need one), `Tools/fish_cut.py` (a copy), and your report file.
- The only pictures you may use are the 15 in `Catch/SourceArt/Fish/`. No downloads, no generated pictures, no fonts, no new sounds.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, the level, other assets, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game. Swim amplitudes and other look numbers are yours; list them.
- Do not delete or hide old work: the painter fish and the old fish material stay in the project. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/`.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-fish-art.md`, in English. Create it right after reading, with one status line per block, and extend it after each block. Sections: how fish were drawn before, what changed by block, the swim table, assets added, frames with paths, checks with their output lines, cost before and after, not checked, placeholders that remain, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
