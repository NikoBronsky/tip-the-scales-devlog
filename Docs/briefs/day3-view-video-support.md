# Brief: view - two staging aids for the video (no rule changes)

Written by the orchestrator on 14.11.2026, 14:58. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base `722f4d9` (build 15). You do not commit.

## 1. Goal

The director rejected video draft 2. Two of his points need the game's help: the video must show Dynamite and Harpoon inside rich late builds (he wants harpoons and blasts as their own mad builds), and the calm first part must not be covered by teaching tips ("ролик ... рассказывает что итак есть в подсказка" about draft 1). The video is recorded from a packaged build that is driven by console commands. Give that build two aids and package it. The rules of the game, the default game and everything the player sees without these commands stay exactly as in build 15.

## 2. What exists now (found by the video executor and the orchestrator; verify before you rely on a line number)

- `catch.CheatRun` and `catch.FrameRun` build their tables without the candidate switch of the launch argument `-new`: `Catch/Source/Catch/CatchSession.cpp:599` and `:728`. A helper `SessionTables()` exists and honours `-new`. With plain tables the shop never offers a candidate charm (`Profile.cpp:130-139`), so Dynamite and Harpoon cannot be bought in a staged late build.
- Console commands are registered in `CatchSession.cpp:1126-1345`; HUD commands are the `catch.Hud*` family.
- Teaching tips: `UCatchHud::ShowTip` (`CatchHud.cpp:4448`), fired at `CatchHud.cpp:2773-2777`, `3623`, `3639`. Besides them the HUD shows a day hint bar on days 1-2 and a preparation hint every day ("Pick a bait and a depth. Press START DAY."). No command hides any of them.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. `catch.CheatRun` and `catch.FrameRun` (and `catch.BiteRun` if it has the same gap) use the session tables, so with `-new` on the command line the candidate charms can be offered and bought in these runs. Without `-new` these commands behave exactly as before.
2. New console command `catch.Tips 0|1` (default 1). With 0 the HUD shows no teaching tip, no day hint bar and no preparation hint. It hides only hint texts: the day rule plate, the forecast, charm cards, numbers and every other element stay. It is a session flag, not saved, not reachable from any menu, off in the default game. Log one line `CATCH_TIPS 0|1` when it changes.
3. Nothing else changes. No new rule, number, screen or player-facing text. No Cyrillic under `Catch/`.

## 4. Work

A. The two changes of section 3.
B. Build: `python -X utf8 Tools/check_all.py --ue` in your worktree must give no fail.
C. Package: `powershell -NoProfile -ExecutionPolicy Bypass -File Tools\package-win64.ps1 -Project F:\UE\EpicJam_ui\Catch\Catch.uproject -OutDir F:\UE\EpicJam_ui\Packaged\v15` (add `-EditorOpen` only if an editor is open). Do not touch `Packaged\v14`.
D. Proof on the packaged build, one game at a time, on the second display, started and stopped by you only (launch line as in `F:\UE\EpicJam\Tools\video\record_take.ps1`; read that script first, and never click or answer a system dialog):
   - with `-new`: a `catch.CheatRun` run in which the shop offers Dynamite or Harpoon and `catch.Buy` puts it on the shelf (a `catch.Shot` frame and the log lines);
   - without `-new`: the same commands never offer a candidate;
   - `catch.Tips 0` on day 1: a frame of the preparation and a frame of the day without any hint text; `catch.Tips 1`: the hints are back.
   Look at the frames yourself.

## 5. Done means

- The gate with `--ue` has no fail. The package exists at `F:\UE\EpicJam_ui\Packaged\v15\Windows` and starts.
- The proofs of block D are in the report with the frame paths.
- `git status` shows only the files you meant to change. Never add `Catch/SourceArt/Fish/atlas.png` or `atlas.json`.

## 6. Boundaries

- Write only inside `F:\UE\EpicJam_ui`. Another executor works in `F:\UE\EpicJam_core`; do not touch it. Do not touch `F:\UE\EpicJam` except to read.
- Do not stop any process you did not start. If the build is blocked by an open editor or a locked file, report it; do not close anything.
- Delete nothing. No commits. Invent no rule.

## 7. Report

`Docs/briefs/reports/day3-view-video-support.md` in your worktree, in English: what changed by file and line, the build and package results, the proofs, what is not verified. Final message to the orchestrator: the same in short. Time box: about 40 minutes. You report to the orchestrator, not to the director.
