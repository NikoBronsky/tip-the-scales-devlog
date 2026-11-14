# Launch note for the brief `day2-text-language.md` (executor: Codex)

Written by the orchestrator on 13.11.2026, 05:25 (owner's clock, UTC+1). This note and the brief together are the whole task.
Work autonomously to the end. Do not ask the user questions: the owner is asleep. You report to the orchestrator, not to the owner.

## Read first, in this order

1. The brief: `F:\UE\EpicJam\Docs\briefs\day2-text-language.md`. It lives only in the main checkout. Do not edit it.
2. The project contract: `F:\UE\EpicJam_ui\CLAUDE.md` (Russian). Sections 5, 7, 8 and 9 matter: build rules, no regression, no commits by executors, what never goes to git.
3. `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-core22.md`: the state of the view you build on. Section 6 lists the newest player texts.

## Where you work

- Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base commit `63746b6` (clean). You are the only writer in this worktree.
- Never touch `F:\UE\EpicJam` (main), `F:\UE\EpicJam_core` (another executor works there now) or `F:\UE\EpicJam_look`.
- No commits, no pushes, no packaging, no file deletions (files are deleted only by the owner), no downloads, no fonts, no generators.

## Updates to the brief

The brief was written before the view moved to core 2.2. These lines win over the brief where they differ.

1. Brief section 2 is partly out of date. Modifier (charm) names, class, rarity and rule texts now come from the core: `UCatchSession::CharmText` calls `Game::CharmTextOf` (see `Catch/Source/Catch/CatchSession.h`, `CatchSession.cpp` and `Catch/Source/CatchCore/Public/CatchCore/Game.h`, struct `CharmDetails`). `CatchUiText.h` and `CatchUiText.cpp` hold only a comment now. Keep the files. Read how `CharmDetails.rule` is built: a template with a live value, or a finished English sentence. Make your layer translate it correctly, live numbers included.
2. `F:\UE\EpicJam_ui\Docs\design\modifiers-v2.data.json` (version 2.2) has `nameRu` and `ruleRu` for all 53 modifiers next to the English fields. Your generator must take the modifier name pairs and rule pairs straight from this JSON, in addition to `Docs/design/text-ru.tsv`. Reason: a balance pass is changing some of those values now in another worktree. After its merge a regeneration must be enough; no hand edit.
3. Proof scripts. Do not use the two scratchpad scripts named in brief section 6: they are for the old core. Use the current ones in the worktree:
   - `bash Catch/Saved/core22_run_check.sh /f/UE/EpicJam_ui <prefix> "catch.HudAutoPack 0,"`
   - `bash Catch/Saved/core22_run_feedback.sh /f/UE/EpicJam_ui <prefix>`
   Read them before you copy them for your own runs. Do not edit them in place. Other useful scripts in the same folder: `core22_lucky.sh` (a cheat run with five modifiers, a lucky hit, counters), `core22_rules.sh` (rule cards), `core22_mouse.sh`.
4. Reference numbers that must stay equal after your work (English start, no argument):
   - plain day: `ev: 199`; `DaySettled day 1, 37 pts, target 10 pts, payout 4 c + over 4 c + interest 1 c, modifiers 0 c, coins 14 c`; `errors: 0`;
   - flow run: `ev: 206`; the same `DaySettled` line with 25 pts; `errors: 0`.
5. The run now opens in the first shop ("before day 1", button "TO DAY 1", "TARGET DAY 1:"). The starting bait pack opens by itself over it. `catch.NewRun 42,catch.NextDay` leads to the preparation of day 1.
6. Developer commands you can use for frames: `catch.CheatRun <seed> <charm|~charm>...`, `catch.HudLookRule <1..6>`, `catch.HudRuleCard`, `catch.HudLookRuleOff 1`, `catch.Hover <fish id|name>`, `catch.HudPoint shelf <n>`, `catch.Shot <name>` (frames land in `Catch/Saved/ue_mcp/`), `catch.After <seconds> <command>`.
7. Run one game at a time. A `-game` run binds port 8000 (an editor-link plugin). Two overlapping runs make the later one log two errors about `127.0.0.1:8000`. That is not a game bug: rerun it. UnrealBuildTool builds from several worktrees wait for each other (`-WaitMutex`), so a build can take longer than usual.
8. Build and gate: `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui` must end with `10 ok, 0 fail, 1 skip`. The gate fails on any Cyrillic letter under `Catch/`. That is why the Russian table goes into the game only as `\uXXXX` escapes written by your generator. Do not change the gate.
9. Frames. Brief section 5 item 6 asks you to look at frames in both languages. If you can open PNG files as images, do it and fix what you see (clipped or overlapping Russian text). If you cannot view images, say so plainly in the report, finish blocks A, B and D with log proof (`catch.TextMissing` reporting 0 missing keys, the gate, both build flag values, the equal reference numbers), and take the frames anyway. The orchestrator then does the layout pass.
10. Keep every existing pop, count-up, tilt and timing. The English layout must not change.

## If your sandbox blocks a command

Your session can write only inside the folder it was started from. It is started from `F:\UE\EpicJam_ui`, so the worktree is writable. An UnrealBuildTool build and a game run also write outside the worktree (the engine folder, the user's application data). If the sandbox blocks such a command, do not try to get around it and do not ask for wider access. Do this instead:

1. Finish the code of blocks A and B, the Russian table and the generator. Run `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check`.
2. Read your C++ changes once more for compile errors: every changed call site, every format string and its arguments, both values of `CATCH_WITH_RUSSIAN`.
3. List in the report every command you could not run, with the exact error text. The orchestrator runs them and does the layout pass.

## Done means

Brief section 5, with the updates above.

## Report

Write `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-text-language.md` in English (brief section 8). Give a short summary at the end of your run: what is done per block and where (file, function); counts of keys and translated keys; terms you chose beyond the glossary; texts left English and why; the check results with log lines; the list of frames; every decision you changed; what is left; questions. If you run out of your usage limit, stop after a finished block with a green build and say exactly what is left.
