# Brief: view part 9c - the choice of the run mode, the timed mode on screen, core queries instead of view mirrors

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. The base is the commit the orchestrator names in the launch message: view part 9b plus the merge of core 2.6 (`track/core` `e776b8d`, API 7).

## 1. Goal

The owner wants two ways to play. Decision `timed-mode-approved` (14.11, Russian, dictated by voice): «хотелось бы все-таки добавить режим отдельный для игры именно на время, чтобы вот как до этого было. ... этот вариант, где тебе надо обычное количество набрать, медленный, размеренный ... А тот вариант был именно вот на скорость, на раш, на азарт, и ты не замечал, как ты там большое количество рыбы возил.» Then: «Да, делай отдельный режим.» The calm mode (the day ends at the target) stays the default.

## 2. What exists now (verify before you rely on it)

- Core report: `Docs/briefs/reports/day3-core26-timed-mode.md` (in the worktree after the merge). In short: `NewRun(seed, level, RunMode mode = RunMode::Target)`; the mode is in `State::mode`; in the timed mode the day runs 50 s, `TargetReached` fires once with 0 s saved and the day goes on, a missed target gives `RunOver`, `DaySettled.payoutOver` pays up to 4 coins and the time payout is 0; the timed mode has its own target table; Tide Pool is never offered there.
- The view starts a run without a mode (`UCatchSession::NewRun`, the title screen and the run-over screen call it). The title screen and the settings: `CatchUiMenu.cpp`.
- The day HUD and the day-end flow assume the calm mode: the day plate shows the stake of the day clock (`CatchHudFlow.cpp`, `TimeBonusNow`), the words TARGET REACHED end the day (`CatchUiFlow.cpp` `PaintTargetReached`), the flow rows are PAYOUT, TIME SAVED, INTEREST, charm rows (`CatchHudFlow.cpp` `BuildFlowRows`; PAYOUT already adds `payoutOver`).
- New core queries: `Details().eatenBy` follows the reach rules of charms; public `WouldBite(rod, hook, kind, bait = AnyBait, depth = -1) const`. The view still mirrors the bite rule in `UCatchHud::HookAcceptsView` (part 9a, green glow).

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. The mode is chosen when a run starts: on the title screen, next to the start of a new run, two clear choices with one line each of what the mode is. Working names (English, player-facing): "CALM: the day ends when you reach the target" and "RUSH: the day runs its full time, extra catch pays coins". The calm mode is preselected. The run-over screen starts the next run in the same mode and lets the player go back to the title to change it. The choice answers the cursor like every other control (hover, press, sound).
2. The mode is visible during the run: a small mark on the day plate. In the timed mode the day plate shows the clock without the time-bonus stake; after the target is reached the plate and the catch counter show that every further point counts toward the extra coins (from the core numbers only).
3. TARGET REACHED in the timed mode is a short celebration that does not stop the day and does not cover the sea.
4. The day-end flow in the timed mode: no TIME SAVED row; a row for the coins over the target (label "OVER TARGET") that counts like the other rows.
5. Russian rows for every new text (`Docs/design/text-ru.tsv`): calm = «Спокойный», rush = «На время».
6. The green glow uses the core query `WouldBite` instead of the view's own mirror; delete nothing that other code still needs, remove the mirror if nothing uses it. The red glow reads `eatenBy` as before (it now follows charms): check Big Mouth or Free Feast in one frame.
7. Proof modes and cheats start runs in the calm mode unless they name the timed mode; add one proof mode for the timed mode.

## 4. Work by blocks

Order: compile on core 2.6 with unchanged behaviour; decision 6; decisions 1 and 5; decisions 2-4; decision 7. Extend the report after each block.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`; `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes; proof runs in English and Russian with 0 errors and 0 missing keys.
- Frames from a running game, looked at by you: the title screen with the mode choice (calm preselected; rush hovered and chosen), in English and Russian; a timed day before and after the target; the timed day end with the OVER TARGET row; a timed day that misses the target and ends the run; a calm day unchanged; the green glow from `WouldBite`.
- The log of a timed run shows the mode at run start.
- The report lists every changed text before and after and every decision you changed.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py classes en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms ru 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time, grouped near the end of a block. Never leave a proof game running. If the orchestrator writes "pause", stop every build, test and game at once until "resume". Do not kill a process you did not start; do not touch `Packaged/`. Use absolute paths in every command; never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_core`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, charm rule texts, `Catch/Source/Catch/Catch.Build.cs`, config files, the level, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings. A core problem goes into the report.
- Do not invent rules or numbers of the game. No downloads, no generated pictures, no fonts, no new sound files. No deleting or hiding of old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The theme rule of the jam entry: no director, cinema or "cut" wordplay in the game.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-timed-mode.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
