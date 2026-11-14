# Brief: sea ambience and music (the sound bed)

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 01:20 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`, base `f7e7a97`. This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-sound-bed.md`).

## 1. Goal

The game has 29 short action sounds and nothing under them: no sea noise, no music. Add a sound bed: a looping sea ambience that changes with the camera depth, and one looping music piece. All of it is made by script, like the 29 sounds.

Why it matters. The owner played the packaged build on 13.11 at about 01:00 and said: «а звука нет?», then «а всё, я понял. нет шума и музыки». Earlier decisions of the owner: «У нас везде полишинг, мы везде звук взаимодействия» and «Море живое». On sources he said: «по-хорошему все, что нам нужно, ты можешь, насколько я понимаю, сделать сам».

## 2. What exists now (verified by the orchestrator)

- `Catch/SourceArt/Sound/make_sounds.py`: numpy synthesis, fixed seed 20261112, writes 29 files `S_<Name>.wav` (48 kHz, 16 bit, mono), two previews and `README.md`. A run always gives the same bytes.
- `Catch/SourceArt/Sound/README.md`: play rules and the table of the 29 files. Level scale: the bite peaks at -3 dBFS; the other files carry their relative level; RMS of the files is -17 to -35 dBFS.
- `.gitignore` already ignores `Catch/SourceArt/Sound/preview_*.wav`. `*.wav` and `*.uasset` go through Git LFS.
- `Catch/SourceArt/SceneScripts/build_look_camera_sound.py`: imported the 29 files to `/Game/Audio`, made the submix `SM_CatchMaster` with the compressor preset `SFX_CatchCompressor` (threshold -2 dBFS, ratio 8), sent every wave to it, and set `look_sounds` on the `CatchTackleView` actor of `/Game/Maps/Sea`. Its line 17 points at the main checkout (`F:/UE/EpicJam/...`), and the script also moves the camera and the side walls. Do not run it again.
- `Catch/Source/Catch/CatchTackleView.h:56`: `TArray<TObjectPtr<USoundBase>> LookSounds` (hard references, so the cook takes the sounds).
- `Catch/Source/Catch/CatchTackleView.cpp:224-227`: builds the list and calls `Sound.Init(this, List)`; `:348` calls `Sound.Tick(...)`; `:577` calls `Sound.OnEvent(...)`.
- `Catch/Source/Catch/CatchLookSound.cpp`: `FCatchLookSound` is a plain C++ class (not a UObject). `Init` maps a sound by its asset name without `S_` (lines 22-24) and logs `Look sound ready: N files.` (line 27). `Play` uses `UGameplayStatics::PlaySound2D` (line 59). It has its own clock `Now`. There are no looping sounds and no audio components.
- Camera depth: `ACatchSeaView::GetCameraDepth()` (`CatchSeaView.h:117`), 0 at the surface view, up to `GetSeabedCameraDepth()` (`CatchSeaView.h:115`, 3230 game units). The HUD moves the camera with the mouse wheel (`CatchHud.cpp:2528`).
- Core phase: `Session->State().phase` (`catchcore::Phase`), already read in `CatchLookSound.cpp:148`.
- Scripted runs of the game from the console give no audible sound for the orchestrator, and the orchestrator cannot hear anyway. Proof of this work is numbers, log lines and the owner's ears.
- Lesson of 12.11: a packaged build can differ from the editor. Sounds that nothing references are not cooked. Keep the three new sounds in `LookSounds`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Source. Everything is synthesis in Python with numpy, fixed seed. No downloads, no generators, no samples from outside, nothing from the browser prototype. The music is an original piece: do not imitate any existing track or the score of any game.
2. New script `Catch/SourceArt/Sound/make_bed.py`. It must not change the bytes of the 29 existing files. It may import building blocks from `make_sounds.py` only if that import does not run the old main.
3. Three loop files, stereo, 48 kHz, 16 bit, each a seamless loop (the tail of reverbs and decays wraps into the head; no click, no gap, no level dip at the seam):
   - `S_AmbSurface.wav`, 24 to 32 s. Open sea beside a wooden raft: slow swells of filtered noise, small laps against wood, light wind. No birds, no voices.
   - `S_AmbDeep.wav`, 24 to 32 s. Under water: a low muffled rumble (most energy below 300 Hz), slow movement, a few soft bubbles.
   - `S_Music.wav`, 64 to 96 s, a whole number of bars. Mood: a calm, warm, slightly playful morning at sea; it must not tire after 30 minutes of looping. Suggested build: 72 to 84 bpm, a major or pentatonic key, plucked strings by Karplus-Strong (nylon or ukulele colour) with a sparse melody and an arpeggio, a soft pad, a soft bass, no drums or only a very soft shaker. Keep it mellow (little energy above 6 kHz) so that the bright action sounds stay on top.
4. Levels on the README scale. The files carry their final relative level and the game plays them at volume 1.0 times the state factors of decision 6: music RMS about -27 dBFS with peaks under -10 dBFS; each ambience RMS about -33 dBFS with peaks under -14 dBFS. Revocable by a factor of two either way if your numbers show a problem.
5. Routing. The loops must not pump the master compressor. Put the three loops on their own submix without effects (for example `SM_CatchBed`). If you can show they sit far under the threshold in every state, `SM_CatchMaster` is also acceptable.
6. Playback rules (built in the sound class, numbers in one place):
   - The three loops start once when the tackle view begins play, fade in over 2 s and never restart on a new run or a new day.
   - Depth mix. s = smooth step of the camera depth from 0 (surface view) to the depth where the whole view is below the first zone. Surface layer volume = 1 - 0.9 s. Deep layer volume = 0.1 + 0.9 s. Take the zone depth from the core tables or the sea view, not from a new constant.
   - Music by phase: day 0.6; preparation, shop and summary 1.0; run over 0.35. A change glides over 1.2 s.
   - Two master factors, default 1.0: music and ambience. Console commands `catch.MusicVolume <0..2>` and `catch.AmbienceVolume <0..2>`. A later options menu will set the same two values; there is no menu or key in this task.
7. Lifetime. Looping audio components need an owner that the garbage collector sees. Hold them in `UPROPERTY` fields of `ACatchTackleView` (or another UObject you name), not in raw pointers inside `FCatchLookSound`.
8. Assets. Import to `/Game/Audio/S_AmbSurface`, `S_AmbDeep`, `S_Music`; set `looping` on the three waves; append them to `look_sounds` of the `CatchTackleView` actor in `/Game/Maps/Sea`; save. Do this with a new editor script `Catch/SourceArt/SceneScripts/build_sound_bed.py` that is safe to run again and takes the wav folder from the project directory, not from a fixed path.
9. Proof without ears. `make_bed.py` prints, and writes into a new README section, one row per loop: length, peak dBFS, RMS dBFS, seam check (largest sample step across the wrap against the median step inside the file), share of energy in four bands (under 200 Hz, 200 to 2000, 2000 to 8000, above 8000), left-right correlation. For the music also: key, tempo, bars, form.
10. Previews for the owner (ignored by git through the existing `preview_*.wav` rule):
    - `preview_bed.wav`, about 45 s: surface ambience alone, a glide to the deep layer and back, then the music over the surface ambience with a few of the existing action sounds at their real level, so the balance can be judged.
    - `preview_music.wav`: one full loop, then the first 8 s again, so the seam can be heard.
11. Debug. Log one line when the bed starts and one line on each phase change (not every tick), for example `Look bed: surface 1.00 deep 0.10 music 0.60`. Add `catch.SoundDump`: one log line per loop with "playing or not" and the current volume.

## 4. Work by blocks

- A. `make_bed.py`, the three wav files, the README section, the two previews.
- B. C++: loops, depth mix, phase mix, the two console volumes, the dump. Files: `CatchLookSound.h/.cpp`, `CatchTackleView.h/.cpp`, and the file where the other `catch.*` commands are registered.
- C. Editor: `build_sound_bed.py`, import, looping flag, submix, `look_sounds`, save the level.
- D. Checks and the report.

Order that works with the build lock: write the C++ and build the editor with the editor closed; then open the editor, run block C, save all, close the editor; then the gate and the scripted run.

## 5. Done means

1. `python -X utf8 Catch/SourceArt/Sound/make_bed.py` writes the three loops, the previews and the README section; a second run gives the same bytes; `git status` shows no change in the 29 old wav files.
2. The README section holds the proof table of decision 9, and every seam check passes by your stated threshold.
3. `/Game/Audio` has the three looping waves; `LookSounds` of the tackle view in `Sea.umap` has 32 entries.
4. In a scripted run the log shows `Look sound ready: 32 files.`, the bed start line, a bed line on the change to the day and to the shop, and the three `catch.SoundDump` lines with "playing". Moving the camera down with `catch.CameraDepth` (see `CatchSeaView.h:21`) changes the surface and deep volumes in the dump.
5. The plain scripted day gives the same numbers as before this task: 102 events, 31 pts against target 10, coins 8, 0 lines with `Error:`.
6. `python -X utf8 Tools/check_all.py --ue` is green: 10 ok, 0 fail.
7. The editor is closed at the end. Nothing is committed.

## 6. Verification commands

The Bash tool can be broken in this environment. Use the PowerShell tool (Windows PowerShell 5.1: no `&&`, no `2>&1` on native programs). Edit source files only with the Edit tool; a PowerShell `Get-Content`/`Set-Content` round trip broke non-ASCII characters in this repo on 12.11.

```
python -X utf8 Catch/SourceArt/Sound/make_bed.py
powershell -NoProfile -ExecutionPolicy Bypass -File Tools\build-editor.ps1 -Project F:\UE\EpicJam_ui\Catch\Catch.uproject
python -X utf8 Tools/check_all.py --ue
```

Editor (one editor at a time; an open editor of any project blocks the editor build):

```
Start-Process -FilePath 'F:\UE_5.9\Engine\Binaries\Win64\UnrealEditor.exe' -ArgumentList '"F:\UE\EpicJam_ui\Catch\Catch.uproject"'
python -X utf8 Tools/ue_mcp.py --timeout 5 status
python -X utf8 Tools/ue_mcp.py py Catch/SourceArt/SceneScripts/build_sound_bed.py
python -X utf8 Tools/ue_mcp.py call EditorToolset.EditorAppToolset ExitEditor "{}"
```

The link answers about 12 s after the editor window appears. If it does not answer within 90 s, read `CLAUDE.md` section 5 (auto start of the link in `Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini`). Save everything before the exit: `unreal.EditorLoadingAndSavingUtils.save_dirty_packages(True, True)` as the last line of an editor script. Read `Docs/research/ue59-editor-link-capabilities.md` before the editor work.

The orchestrator's scripted day (gate with the editor build, then a game run; prints the event count, the dump, the error count):

```
& "C:\Program Files\Git\bin\bash.exe" "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_check.sh" /f/UE/EpicJam_ui bed "catch.HudAutoPack 0," 2>$null
```

Expected: `10 ok, 0 fail, 1 skip`, `ev: 102`, `coins 8 c target today 10 pts`, `errors: 0`. Read that script to copy the game command line for your own run with the sound commands; write your own script into your scratch folder, do not edit the orchestrator's file.

## 7. Boundaries

- Work only in `F:\UE\EpicJam_ui`. Do not touch `F:\UE\EpicJam` (main) or `F:\UE\EpicJam_core`. No other executor is active now.
- Allowed files: `Catch/SourceArt/Sound/make_bed.py` (new), `Catch/SourceArt/Sound/README.md` (add a section, leave the old text), the three new wav files, `Catch/SourceArt/SceneScripts/build_sound_bed.py` (new), `Catch/Content/Audio/S_AmbSurface.uasset`, `S_AmbDeep.uasset`, `S_Music.uasset`, one new submix asset, `Catch/Content/Maps/Sea.umap`, `Catch/Source/Catch/CatchLookSound.h/.cpp`, `CatchTackleView.h/.cpp`, the file with the `catch.*` console commands, and the report.
- Not allowed: `CatchCore`; the HUD files (`CatchHud*`, `CatchUi*`) unless one line is unavoidable, and then say so; the 29 old wav files and their assets; `MPC_Sea`, `build_sea_materials.py`, `build_look_fish.py`, `build_look_camera_sound.py` (do not run them).
- No game rule, number, screen or player-facing text. This task is sound only.
- No downloads, no paid or free generators, no accounts, no network sources.
- Do not commit, do not package, do not push. Do not delete files. Do not kill Unreal processes you did not start. Do not edit `CLAUDE.md`, `AGENTS.md`, settings or permissions.

## 8. Report

Write `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-sound-bed.md` in English:

1. State and gate: what is done, the exact check results, what was not run.
2. The proof table of the three loops.
3. The music in words: key, tempo, bars, form, instruments and how each is synthesized.
4. Playback as built: the numbers of decision 6, where they live in the code, the log lines of a run.
5. Changed files.
6. Deviations from section 3, each with the reason.
7. Questions for the orchestrator.

## 9. Chain of command

You report to the orchestrator, not to the owner. Put open questions into the report; the orchestrator answers them. Do the parts that need no answer first. If something needs an owner decision (a rule, a number, a screen), do not invent it: write the question down.
