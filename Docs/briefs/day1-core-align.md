# Brief: day one, step 4 — align the rules core with the rules reference

Status: written and handed out on 12.11.2026 at 09:35 (owner's clock). Executor: Opus, effort high (Codex reached its usage limit at 09:10).

## 1. Goal

Make the rules core play exactly the game the owner approved. The owner approved the rules by playing a prototype; his words: «Выглядит практически готово ... можно начать переносить». The core was written from the content list, which misses many numbers and small rules. A written reference of the approved rules now exists. Align the core to it and prove it with tests.

## 2. What exists now (verified by the orchestrator on 12.11)

- **Your work folder is the git worktree `F:\UE\EpicJam_core` (branch `track/core`).** Write only there. Never write into `F:\UE\EpicJam`: another executor has the Unreal Editor open there and builds the first playable version against the current core.
- The core: `Catch/Source/CatchCore/` (about 1,500 lines, plain C++20, namespace `catchcore`). Tests and the balance runner: `Catch/CoreTests/`. Build and run them: `Tools/core-test.ps1` (8 seconds); `-RunBalance -Runs N -Days D` runs the runner.
- The frozen interface: `Docs/design/core-interface.md`. Sections 1-9 are frozen names; section 10 is the C++ surface and the readings the first executor took.
- **The rules reference: `Docs/design/rules-reference.md` (742 lines).** It states every rule and number of the approved play. Its section 17 lists 16 places where the core differs. Its section 16 lists 21 differences from the content list. Its section 15 lists what is unknown.
- The first brief and its code rules still hold: `Docs/briefs/day1-core.md`, section 3 (no exceptions, no RTTI, no iostream in the core, banned identifiers `check`, `verify`, `ensure`, `min`, `max`, `std::int64_t` for money, tests outside the module folder, standalone compiler flags).
- You never see the prototype and you do not need it. Do not look for it. The reference is the only source.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. Order of authority for rules and numbers: the rules reference first, then the content list, then the current core. Where the reference and the content list disagree (reference section 16), the reference wins. Reason: the owner approved the play, not the list.
2. The five edge behaviours of reference section 15, item 1: implement them as the reference describes the prototype, each behind one named flag in the data, so that the owner can switch one off later without new code.
3. Units: the core keeps x from 0 to 1000 and depth as in the reference. Scale every x quantity of the reference (positions, speeds along x, reach along x) by 1000/1030 once, in the tables. A distance that mixes x and depth uses the scaled x and the unscaled depth. Put the factor in one named place.
4. Time: keep the fixed step of 1/60 s. Do not imitate the frame step of the prototype.
5. The interface names stay: every command, phase, event and state field of core-interface sections 1-10 keeps its name and meaning. You may add fields and events. Where the reference changes a flow (a bought pack opens at once; a rod without fish can be pulled; the sea persists across days instead of a rebuild), keep the old command valid, make the new flow work through the existing names, and write the change into core-interface section 10. The Unreal layer must keep compiling against `Game.h` without edits; if one edit there is unavoidable, list it at the top of your report.
6. All numbers stay data in `Tables`. No rule number in code.

## 4. Work

1. Go through reference section 17 (16 items), then sections 1-14 in order, and align the tables and the rules. Do not skip a section because the core "already has it": compare each number.
2. Tests: for every aligned rule add or change a test that states the number of the reference. At least: fish count per species on day 1 and on day 5; day end lands every hooked fish; price order with the four worked examples of reference section 7; rounding to nearest; Big Haul window and order; rod timings at levels 1, 3 and 5; fee of days 1-12 on the three levels; offers per shop on days 1-4; pack rules; the Gold Sunfish schedule and which day rules touch it; bait change in the water; the persistent sea and refill; determinism (same seed, same events) still holds.
3. Runner: keep the two simple policies. Add a third policy that plays more like a person: it waits for a chain when a hunter is near and the escape is not close, buys by best money per cost among rod, sharpen, depth and packs, and keeps the fee of tomorrow in reserve. Print the same table. It measures; it does not tune numbers.
4. Update `Docs/design/core-interface.md` section 10 so that it equals the code. Remove readings that the reference replaced.
5. Unreal-side compile check from your worktree, at the end: `& "F:\UE_5.9\Engine\Build\BatchFiles\Build.bat" Catch Win64 Development -Project="F:\UE\EpicJam_core\Catch\Catch.uproject" -WaitMutex`. The editor target is locked while an editor is open; do not run `Tools/build-editor.ps1`.

## 5. Done means

1. `Tools/core-test.ps1` passes, with the new tests, in under 30 seconds.
2. Every item of reference section 17 is aligned and has a test. The report has a table: item, done or not, test name.
3. The game target builds from your worktree.
4. `python -X utf8 Tools/check_all.py` is green in your worktree (without `--ue`).
5. The runner prints its table for three policies and three fee levels.

## 6. Boundaries

- You do not invent the game. If the reference says UNKNOWN or is silent, keep the current behaviour, put the value in data, and write the question into the report.
- Files you may create or change, all inside `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Tools/core-test.ps1`, `Docs/design/core-interface.md` (section 10 only), `Docs/briefs/reports/day1-core-align.md`. Nothing else. Do not change the rules reference or the content list: report an error you find in them.
- Do not commit, do not push, do not delete files, do not start the Unreal Editor.
- Player-facing names are English, exactly as in the reference.

## 7. Report

English, in `Docs/briefs/reports/day1-core-align.md`, at most 45 lines: the table of the 16 items; other rules you aligned; the tail of the test output; the runner table of the third policy on Normal; interface additions; flows that changed for the Unreal layer; questions; errors found in the reference.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Game questions: write them down; the orchestrator asks the owner.
