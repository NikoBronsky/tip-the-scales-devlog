# Brief: view part 10a - the game celebrates big moments; small look fixes; the first day of a new player

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. The base is the commit the orchestrator names in the launch message (view part 9c on core 2.7).

## 1. Goal

The game director's frame for the whole game: «Главное — ощущение и полировка. Каждое действие игрока отвечает красиво: движением, звуком, эффектом.» Small actions answer well now. The big moments of a day do not stand out from the small ones: a chain of three fish, a very heavy fish, a pull with many fish look and sound almost like one Sprat. Make the game answer in steps, so the player feels the difference. View only: no rule, no number of the game, no new screen.

## 2. What exists now (verify before you rely on it)

- Landing: every fish flies to the bucket on its own, the bucket rocks, water splashes, a sound plays (`CatchTackleView.cpp`, events `FishLanded`, `FishInBucket`; the points popups in `CatchHud.cpp`).
- The luck sign of part 9a (`CatchHud.cpp` `OnLuckyChance`): gold flash, rings, sparks, large words. The prize words of part 9b use the same style. `TARGET REACHED`: `CatchUiFlow.cpp` `PaintTargetReached`.
- The day-end count of part 9a rolls points and drops coins one by one (`CatchHudFlow.cpp`).
- Sound: three loops (sea at the surface, sea in the deep, music) and one-shot sounds made in code; find the sound helper before you add a call. No new sound files.
- Glows of part 9a: `CatchLit.h`, red for "would eat", green for "would bite". The part 9c executor noted: the red glow over bright Sunlight water reads pale pink-white (additive red over cyan), frame `text_feast_en_1920_red_hooked.png` in `F:\UE\EpicJam_ui\Catch\Saved\ue_mcp`.
- Hints: a plate at the bottom of the sea shows one line (for example "Fish on! Click the fish or its rod card to pull it in.").
- Reports of the last parts: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-timed-mode.md`, `day3-view-core25.md`, `day3-view-feel-highlights.md`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Three steps of answer for a landing, chosen from core numbers of that landing only (no new rule): normal; big; huge. Draft thresholds, as view constants in one place: big = the fish gives at least 15 % of the day's target, or its chain has 2 links, or the pull lands 3 fish; huge = at least 40 % of the target, or a chain of 3 or more links, or a pull of 5 or more fish, or a Whopper. Tune them on frames so that "huge" stays rare on days 1 to 3.
2. What grows with the step: the splash at the water, the rock of the bucket and the water thrown from it, the size and hold time of the points popup, the sound (the same sounds layered or pitched, louder and lower for heavy fish), a short push of the camera or of the sea block (a few pixels, under 0.3 s, never a long shake), and for "huge" a short gold flash with words that name the reason from core data: "CHAIN x3", "BIG CATCH", "x5 PULL". The visuals may hold for a few frames; the core clock never stops and input is never blocked.
3. Nothing covers a rod card, the day plate or the money. Several big moments close together queue or merge; they never stack into noise.
4. The red glow reads red on bright water and in the deep: change the look of the red glow only (for example a darker red rim under the additive core). The green glow stays.
5. The title screen: the rod on the raft crosses the letters of the title. Keep the letters clear (hide the tackle on the title screen or move it).
6. The extra-coins plate of the timed mode sits on the "SUNLIGHT" label of the sea (frame `text_rush_en_1920_rush_after.png`). Separate them.
7. The first day of a new player, calm mode, fresh start from the title: write down every hint the player sees, in order, with a frame each (title, preparation, first cast, first bite, first pull, target, day end, first shop). If a step has no hint and a new player cannot know what to do, add one line in the existing hint plate (at most three new lines in total; English text plus a Russian row). No new panel, no tutorial screen.
8. The new signs of parts 9b and 9c at 1280 x 720: run the proofs `core25`, `charms25` and `rush` at 1280 and fix overlaps.

## 4. Work by blocks

Order: decisions 1-3; decision 4; decisions 5, 6, 8; decision 7. Extend the report after each block.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`; `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes; proof runs in English and Russian with 0 errors and 0 missing keys.
- Frames from a running game, looked at by you: a normal, a big and a huge landing (a chain of 3, a Whopper, a pull of 5 fish with Trotline); the red glow on bright water and in the deep; the title screen; the timed day plate; the first-day frames of decision 7; the 1280 frames of decision 8.
- A short frame sequence (4 to 6 frames, about 0.1 s apart) of one huge landing, so the orchestrator can judge the motion.
- The report lists the thresholds, every new or changed text, every decision you changed, and what you did not verify.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
python -X utf8 Catch/SourceArt/Text/run_text_proof.py charms25 en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py rush ru 1920
```

Run game proofs one at a time, grouped near the end of a block. Never leave a proof game running. If the orchestrator writes "pause", stop every build, test and game at once until "resume". Do not kill a process you did not start; do not touch `Packaged/`. Use absolute paths in every command; never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_core`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, charm rule texts, `Catch/Source/Catch/Catch.Build.cs`, config files, the level, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings. A core problem goes into the report.
- Do not invent rules or numbers of the game. No downloads, no generated pictures, no fonts, no new sound files. No deleting or hiding of old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The theme rule of the jam entry: no director, cinema or "cut" wordplay in the game.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-big-moments.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. The game director is away until 10:00. Write questions into the report; do not wait for an answer if other work remains.
