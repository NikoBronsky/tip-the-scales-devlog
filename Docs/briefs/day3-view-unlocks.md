# Brief: view part 10b - the profile file, the sign of a newly opened charm, the goals on existing screens

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. The base is the commit the orchestrator names in the launch message (view part 10a plus the merge of core 2.8, API 8).

## 1. Goal

Decision of the game director `unlock-by-tasks` (13.11, Russian, dictated by voice): «первый раз он проходит с простыми наживками и простыми улучшениями, а потом будут постепенно открываться новые. ... по принципу Балатру. ... Наловил много кильки, получил вот эту штуку на стаю ... чтобы это было само по себе задание.» Core 2.8 keeps a profile across runs: 20 charms are open from the start, 39 open by tasks. The view must keep the profile on disk, celebrate a charm that opens, and show the player what to aim for. No new screen: the full collection screen waits for the director's decision.

## 2. What exists now (verify before you rely on it)

- Core report: `Docs/briefs/reports/day3-core28-unlocks.md` (in the worktree after the merge). Interface: `PlayerProfile::Parse(text)` (tolerant, never fails) and `Serialize()`; `Game::SetProfile(profile)` once at startup, before the first `NewRun` (it stays over `NewRun`); `Game::Profile()`; `Game::ProfileChanges()` (+1 at every change); `Game::CharmOpen(kind)`; `Game::OpenCharmCount()`; `Game::Tasks()` -> `TaskInfo{charm, id, name, text, kind, tier, progress, target, done}` in a stable order; event `ContentUnlocked` with `charm != None` = a task opened that charm (`slot` = its row in `Tasks()`, `count` = open charms after it). Without `SetProfile` the game is as before: everything open, nothing counted.
- The session creates the game in `CatchSession.cpp` (`MakeUnique<catchcore::Game>(MakeDefaultTables(), Seed, FeeLevel::Soft)`, line 58 before part 10a) and starts runs in `UCatchSession::NewRun`. `ContentUnlocked` already reaches `CatchHud.cpp` and `CatchTackleView.cpp` (`OnContentUnlocked` is empty).
- The run-over screen and the title screen: `CatchUiFlow.cpp`, `CatchUiMenu.cpp`, `CatchHudMenu.cpp`. The luck sign and the prize words (gold popup style): `CatchHud.cpp`. Item pictures: the item atlas, `CatchItemArt.inc`. Rarity frames: part 8.
- Russian test rows with patterns: `Docs/design/text-ru.tsv`, generator `Catch/SourceArt/Text/gen_text_ru.py`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. The profile file: one text file under the project's saved folder (`FPaths::ProjectSavedDir()`, sub-folder `Profile`, file `catch-profile.txt`); in a packaged game this is inside the game folder, and the submission archive leaves the saved folder out. Load at session start and give the result to `SetProfile` before the first run. A missing or unreadable file is a fresh profile. Save when `ProfileChanges()` moved: at most once per second during play, and always at the day end, at the run end and when the game closes. Write to a temporary file and replace the old one. A failed read or write never stops the game: one log line `CATCH_PROFILE <what happened>`. Log at start: `CATCH_PROFILE loaded: <open> of <all> charms open, <done> of <tasks> tasks done` (or `fresh`).
2. Launch arguments, for testing only (not player-facing): `-unlockall` = no profile at all (everything open, nothing counted, nothing saved); `-freshprofile` = start from a fresh profile and do not read the file (it is saved as usual afterwards). Agent commands: show the profile, make it fresh, finish a task by its id.
3. The sign when a charm opens (during a day or in the shop): a card-like popup with the picture of the charm in its rarity frame, the words "NEW CHARM", its name, and the task that was done. Gold popup style, a sound from the existing sounds, about 2.5 s; it never covers a rod card, the day plate or the money; several signs queue. The charm's rule text is not needed on the sign.
4. Goals on the run-over screen: a block "CHARMS OPENED THIS RUN" with the pictures and names of the charms opened in this run (nothing when none), and a block "NEXT GOALS" with up to three unfinished tasks: the ones closest to done by share of progress (ties: the `Tasks()` order), each with its text, a progress bar and "progress / target", and the charm it opens shown as a locked picture (dimmed, with a lock) plus its name.
5. Goals on the title screen: one line "CHARMS <open> / <all>" and the same "NEXT GOALS" block, placed where it covers neither the title, the mode cards nor the buttons. With `-unlockall` neither the line nor the block is shown.
6. The run-over screen also shows the run in numbers, from what the view already knows: the day reached, fish landed, the heaviest fish (name and kg), the longest chain. Short rows in the existing style; they count up like the day-end rows if that is cheap, else they appear at once.
7. Task texts come from the core in English. Add Russian rows (patterns with the number) for the 39 task texts and for every new text.
8. The shop needs no change: locked charms are simply not offered.
9. One proof mode `unlock`: a fresh profile; a day in which a task is done (by play or by the agent command); frames of the sign, of the run-over screen with both blocks, of the title screen with the goals; then the game is started again and the log shows the loaded profile with the same numbers; then the file is replaced by garbage and the game starts with a fresh profile and no error. Other proof modes run with everything open (as before), so their frames do not change.

## 4. Work by blocks

Order: compile on core 2.8 with unchanged behaviour; decisions 1-2; decision 3; decisions 4-6; decision 7; decision 9. Extend the report after each block.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`; `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes; proof runs in English and Russian with 0 errors and 0 missing keys.
- Frames from a running game, looked at by you, in English and Russian: the sign (one, and two queued); the run-over screen with opened charms, goals and run numbers; the title screen with goals; the title screen with `-unlockall` (unchanged).
- The log lines of the load, of a save, of the second start with the same numbers, and of the garbage-file start.
- The report lists the file path in the editor and in a packaged game, every new text with its Russian row, every decision you changed, and what you did not verify.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
python -X utf8 Catch/SourceArt/Text/run_text_proof.py rush en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py unlock en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py unlock ru 1920
```

Run game proofs one at a time, grouped near the end of a block. Never leave a proof game running. If the orchestrator writes "pause", stop every build, test and game at once until "resume". Do not kill a process you did not start; do not touch `Packaged/`. Use absolute paths in every command; never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_core`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/modifiers-v2.data.json`, charm rule texts, task texts or targets, `Catch/Source/Catch/Catch.Build.cs`, config files, the level, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings. A core problem goes into the report.
- Do not invent rules or numbers of the game. No new screen, no settings entry. No downloads, no generated pictures, no fonts, no new sound files. No deleting or hiding of old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The theme rule of the jam entry: no director, cinema or "cut" wordplay in the game.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-unlocks.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. The game director is away until 10:00. Write questions into the report; do not wait for an answer if other work remains.
