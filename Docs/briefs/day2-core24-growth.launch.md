# Launch note for the brief `day2-core24-growth.md` (executor: Codex, blocks 1-3)

Written by the orchestrator on 13.11.2026, about 13:05 (owner's clock, UTC+1). This note and the brief together are the whole task.
Work autonomously to the end. Do not ask the user questions. You report to the orchestrator, not to the owner.

## Read first, in this order

1. The brief: `F:\UE\EpicJam\Docs\briefs\day2-core24-growth.md`. It lives only in the main checkout. Do not edit it.
2. The project contract: `F:\UE\EpicJam_core\CLAUDE.md` (Russian). Sections 5, 7, 8 and 9 matter: build rules, no regression, no commits by executors, what never goes to git.
3. `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core23.md`: the state of the core you build on, section 3 is the interface delta of the start gift, which is the model for the reward.

## Your part of the brief

Do blocks 1, 2 and 3 of brief section 4, plus the Russian text and the "For the view" section of block 6 for what you changed:

1. The reward flow after each passed day (brief decisions 1-2).
2. The fit filter for the reward and the shop, and the `needs` audit (decision 3).
3. The rule day by fish kind with the query for the view, its text, and the Sharp Rod gift text (decisions 4-5).

Do NOT do blocks 4 and 5 (bot policies and balance pass 2): another executor does them later. Do not move any target, price or charm value. One exception: the bots must not hang on the new pending reward. Give the bots the simplest policy (take the first offered charm that fits the shelf, else skip) behind the existing table switches, so the existing balance program still runs to the end. Keep the reward and the filter behind table switches (brief decision 7) with the switches ON by default in the game data.

Version: ApiVersion 5, data version 2.5 (brief decision 8).

## Where you work

- Worktree `F:\UE\EpicJam_core`, branch `track/core`, base commit `d6a4ceb` (clean). You are the only writer in this worktree.
- Never touch `F:\UE\EpicJam` (main), `F:\UE\EpicJam_ui` (another executor works there now) or `F:\UE\EpicJam_look`.
- No commits, no pushes, no packaging, no file deletions, no downloads.

## Checks you run

From `F:\UE\EpicJam_core`:

- `python -X utf8 Tools/check_all.py` must end with `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/CoreTests/gen_charm_rows.py --check`
- `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1` (the engine-free tests; add your new tests there).
- One short balance run to prove the bots still finish: `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 20 -Days 15`. Never run two copies at once.

The gate fails on any Cyrillic letter under `Catch/`. Russian charm text lives only in the fields of `Docs/design/modifiers-v2.data.json` that already hold Russian.

## If your sandbox blocks a command

Your session can write only inside the folder it was started from (`F:\UE\EpicJam_core`). An UnrealBuildTool build writes outside it; do not run it and do not ask for wider access. If the sandbox also blocks the engine-free tests or the gate:

1. Finish the code, the tests and the data of your blocks.
2. Read your C++ changes once more for compile errors: every changed call site, every new enum value in every switch, the generated rows against the generator.
3. List in the report every command you could not run, with the exact error text. The orchestrator runs them.

## Report

Write `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core24.md` in English (brief section 8). Create it right after reading, with one status line per block, and extend it after each block. Sections for your part: what changed by block (file, function), the `needs` audit table (tag, predicate, charms), changed texts (English and Russian), "For the view" (new state, commands, events, result codes, the rule-day query), checks with their output lines, commands you could not run, decisions changed, open questions. If you run out of your usage limit, stop after a finished block with green tests and say exactly what is left.
