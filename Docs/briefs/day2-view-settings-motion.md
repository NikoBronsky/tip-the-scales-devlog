# Brief: view, part 6 - the fish meets the hook; the interface comes alive (cursor light, depth on hover, idle life, the settings panel); a response audit

Extended on 13.11.2026 at about 19:00 after the director's playtest notes on build 9: block 0 (the hook) and decisions 13-17 are new.

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 18:45 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message (view part 5, the item pictures, is committed in it). This brief lives only in the main checkout.

## 1. Goal

The settings panel must feel as alive as the rest of the game. It arrives, answers every action and leaves with motion, sound and a small effect. After that, every other control and window of the interface gets the same check, so that no screen feels flat.

Why it matters. The owner (game director), decision `settings-need-effects`, 13.11, after he opened the settings in build 9: «В настройках нет. эффектов, а мы договорились что всё в игре включая интерфейс будет сделано круто и с эффектами. я просто хз как называются эффекты для анимации интерфейса.» The frame of the game says: feeling and polish are the main thing; every action of the player answers beautifully, with motion, sound and an effect. He judges every frame against a browser prototype he liked.

Two more notes of the same playtest belong to this block:

- Decision `fish-mouth-misses-hook-b9`: «Первое, что бросается в глаза, что рыбы опять не попадают по крючку. В смысле, она клюет, но немножко офсет есть.» The fish bites, but its mouth is a little off the hook. This is the first thing he sees, so it is block 0.
- Decision `living-interface`: «подумай, что можем улучшить в интерфейсе, что мы можем сделать приятнее, красивее, чтобы оно жило своей жизнью, чтобы там картинки как-то двигались, выглядели объемно, когда курсором вводим, чтобы, не знаю, вокруг курсора фонарь, свечение было, когда мы под наводим и так далее. То есть что-то, что относительно легко для тебя сделать, реализовать, и что даст огромный эффект. Вот как вот эти эффекты сейчас очков собирающихся выглядят супер.» He wants cheap things with a large effect: pictures that move, pictures that look like they have depth under the cursor, a light around the cursor. The points that fly together at the day end are his example of "super".

## 2. What exists now (verify before you rely on it)

- The hook and the mouth: `CatchFishArt::MouthLocal` (`CatchFishArt.cpp:75`, from `MouthX`, `MouthY` of the art table), used by `CatchLookFish.cpp` (lines 592-604: the hang point and the bite bend with its `Peck` gap; `GetMouth` at line 845) and by `CatchTackleView.cpp` (lines 1431-1442: the hook is drawn at `Mouth + Scr(5, 17)`; line 1539: a measure of the distance between the mouth and the bend of the hook). The fish picture bends in the pixel shader while it swims; check whether the mouth of the bent picture is where the code thinks it is.
- The item pictures of part 5: `CatchItemArt.*` and the report `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-item-art.md`.

Paths are under `F:\UE\EpicJam_ui\Catch\Source\Catch\`.

- `CatchUiMenu.cpp`: the settings store (lines 16-110: `MusicPercent` and `SoundPercent` in steps of 10, fullscreen); the layout functions (`SettingsPos`, `SettingsStepPos`, `SettingsTogglePos`, `SettingsBackPos`, lines 119-140); `StepButton` (line 167: hover lift of 8 %, click squash); `VolumeRow` (line 178: the name, `-` and `+`, ten segments, the last lit segment grows by 8 px for 0.35 s after a change, the percent text); the panel itself (lines 381-391: a fade and a 24 px rise in 0.25 s, the word `SETTINGS`, the rows `MUSIC` and `SOUND`, `FULLSCREEN` with an `ON` / `OFF` button, `BACK`).
- `CatchUiMenu.h`, `FMenuView` (lines 87-110): `SettingsAge`, `SetHover[6]`, `SetPress[6]`, `RowPop[2]`.
- `CatchHudMenu.cpp`: `MenuRowPop` (line 132), `MenuSettingsAge` (lines 149, 205, 212), the fill of the view (lines 328-338), the hit table `MenuHits[MhMusicDown + i]`.
- The title screen: the letters of the name fall one after another (`LetterLandTime`), the menu buttons have a hover spring and a click squash.
- The effect vocabulary the game already has: coin and spark bursts, the card sheen (`Ui::PaintPoseSheen`), popups (`Ui::PaintPopup`), the wobble of shelf chips, flights of cards and coins, one sound family per action. Sound reports: `F:\UE\EpicJam_ui\Docs\briefs\reports\day1-sound.md`, `day2-sound-bed.md`.
- Proofs: `Catch/SourceArt/Text/run_text_proof.py` (modes plain, flow, fail, big, depth, screens, title, reward), cheats `catch.After`, `catch.On`, `catch.Shot`.

So the panel has a small rise, a small hover lift and a click squash. The owner reads this as "no effects". The bar is: as lively as the shop and the day summary.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Entrance. The panel lands with a clear overshoot (scale and rise together), the dim behind it deepens, then the title and the rows arrive one after another (about 50 ms apart), each with its own small overshoot. All of it ends within 0.45 s. Input works from the first frame.
2. Exit (`BACK` or Esc). A short reverse, at most 0.2 s, then the menu buttons return one after another. No window of the menu appears or disappears in one frame.
3. Row icons, drawn by the painter: a note for `MUSIC`, a speaker with waves for `SOUND`, a window frame for `FULLSCREEN`. An icon answers a change of its row (the note hops, the waves pulse, the frame grows). At 0 % the icon is crossed out and shakes once.
4. The volume bar. After a change the lit segments answer as a wave that runs from the changed end; the newly lit segment gets a small spark burst, a segment that goes dark gets a small puff; the percent number pops. Reaching 100 % gets one bigger burst. A click on a segment sets that value. Holding `-` or `+` repeats the step.
5. Sound. Every step plays a sample at the new level: the `SOUND` row plays an action sound the game has; the `MUSIC` row changes the bed at once and plays a soft tick. The tick rises in pitch with the value. A control under the cursor gives a soft hover tick if the game has a fitting sound. No new sound files: reuse the existing ones with pitch and volume.
6. The fullscreen control becomes a switch: a knob that slides with a spring between `OFF` and `ON`, the track changes colour, `ON` gets a small burst. The window mode itself changes at once, as today.
7. Buttons of the panel and of the menus. Hover: lift, a sheen that sweeps once, a lit outline. Press: squash and a ring that spreads from the click point. Release: a bounce. Put this into shared helpers so that the title menu, the pause menu and the settings use the same answer.
8. While the panel is open it bobs slightly, like a thing that floats on water (at most 3 px, a period of about 3 s), and a few slow bubbles rise behind it. Text must stay easy to read; nothing rotates text at rest.
9. Limits of taste: no animation is longer than 0.45 s, nothing blocks input, a value applies at once, nothing loops loudly.
10. The response audit. Go through every control and window of the interface (the title, the pause, the settings, the day panel, the rod cards, the bait box and bait menu, the shop tiles, the pack window, the fish table, the shelf, the reward window, the summary, the run-over screen, the hover cards). Fill a table in the report: element, entrance, hover, press, change of value, exit, sound; write "yes", "no" or "weak". Fix the gaps that the shared helpers of point 7 close cheaply. The target: no control without a hover answer and a press answer; no window that appears or disappears in one frame. List what you did not fix, with the reason.
11. No new player text is expected. If you add one, add its Russian row and regenerate.
12. Proof. Add a proof mode `settings` (or a cheat sequence): the entrance in flight, the panel at rest, a volume step (the wave and the sparks), 0 % (the crossed icon), the switch in mid-slide, the exit in flight. Add one strip of five frames of the entrance, 50 ms apart, so that the motion can be judged from stills.

13. The hook (block 0). Measure first: for each of the 15 fish kinds take frames of the bite and of the fish on the hook (several swim phases) and measure the distance on screen between the drawn mouth and the bend of the hook. Then fix the cause (the mouth table, the bend of the picture, the peck gap, the hook offset) until the mouth sits on the hook within 3 px at 1920x1080 for every kind, also for a hunter that ate the hooked fish and for the squid. Put the table of distances before and after into the report. Do not change when or how a fish bites: this is drawing only.
14. The cursor light. A soft warm light follows the cursor over the interface. Cards, tiles, chips and buttons near it get a highlight that moves with the cursor (a sheen or a rim on the side that faces the cursor). It must be gentle: it never hides text and it is not a second cursor. In the sea view it may be a faint glow in the water; keep it off if it costs frame time or hides fish.
15. Depth under the cursor. A card, tile or chip under the cursor tilts toward the cursor (a few degrees, with a spring), its picture shifts a little against the plate (parallax), its shadow moves the other way, and it rises. The picture must look like a thing that lies on the card, not like print. Use it on shop tiles, pack cards, reward cards, fish cards, shelf chips and gift cards through one shared helper.
16. Idle life. Pictures are never frozen: an item picture breathes or bobs slowly, each with its own phase; a few have their own small motion if it is cheap (the worm wriggles, the float bobs, a glint runs over metal and glass, the coin on the wave turns). Do the cheap general motion for all 72 pictures and the special motion only where a general helper gives it (a glint sweep for a list of ids, a wriggle for a list of ids). Motion at rest stays small; text never moves at rest.
18. A layout bug found by the executor of part 5: in the shop with two rods the ROD 2 card of the top bar sits under the rule banner. Fix it, and check the top bar with one, two, three and four rods at 1920x1080 and 1280x720.
17. You have latitude here. If you see another cheap effect with a large result in the spirit of points 14-16, do it and list it in the report as your addition. Stop at what one executor can finish and verify in this block; the order of blocks below is the order of priority.

## 4. Work by blocks (in this order; extend the report after each block)

0. The hook: measure, fix, measure again (decision 13).
1. Shared helpers: the button answer (hover sheen, press ring, release bounce), the burst and the puff, the stagger, the cursor light, the tilt with parallax, the idle motion.
1a. The cursor light, depth under the cursor and idle life on the shop, the pack window, the reward window, the shelf and the fish cards (decisions 14-16).
2. The settings panel: entrance, exit, icons, the volume bar, the switch, the sounds, the bob and the bubbles.
3. The title menu and the pause menu on the shared helpers.
4. The response audit and the cheap fixes.
5. Proof frames and checks.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- The proofs pass with 0 errors and 0 missing keys: `run_text_proof.py plain en 1920`, `title en 1920`, `flow en 1920`, `reward en 1920`, `screens ru 1280`, and the new `settings en 1920`.
- The hook: the table of mouth-to-hook distances for 15 kinds, before and after; close frames of three kinds on the hook.
- The living interface: a frame pair of one shop tile with the cursor on its left and on its right side (the tilt, the highlight and the parallax differ); a strip of five frames of an idle picture; the frame time of the shop screen with and without the new effects (it must not get noticeably worse; state the numbers).
- Frames you looked at yourself, 1920x1080: the six states of decision 12, the five-frame strip, the title menu with a button under the cursor and in a press, the pause menu. One frame of the settings at 1280x720. One frame of the settings in Russian.
- The settings still work: the volumes and the window mode apply and survive a restart (the `CATCH_SETTINGS` log line shows the saved values).
- The report holds the audit table.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py title en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py reward en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py settings en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`. Do not kill a process you did not start. The owner may be playing a packaged build on this machine: do not close any game window you did not open, do not touch `Packaged/`. If the gate reports a time-out of the core tests while the machine is busy, run `Tools\core-test.ps1` alone and report both results.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, a Russian row in `Docs/design/text-ru.tsv` if decision 11 applies, and your report file. Use absolute paths.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Source/Catch/Catch.Build.cs`, `Catch/Config/**`, the level, the assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not add a setting, a rule or a number of the game. The panel keeps its three settings.
- No downloads, no generated pictures, no fonts, no new sound files.
- Do not delete or hide old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/`. No director, cinema or "cut" wordplay.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-settings-motion.md`, in English. Create it right after reading, with one status line per block, and extend it after each block. Sections: what changed by block, the audit table, the sounds reused and where, frames with paths, checks with their output lines, not checked, placeholders that remain, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
