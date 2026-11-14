# Addendum 6 to brief day3-core210-link-charms: a lever for "one kind pays per blast", and its measure (14.11.2026 16:57)

Executor: the same Opus executor, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `9bf6fa7` (your addendum 5 tool levers, accepted and committed by the orchestrator). You do not commit.

## 1. Goal

Your addendum 5 grid shows that the price model holds survival down: every kind a blast takes loses a level, so a larger blast is worse for the player. The director's words for the price were: "Он должнен глушить и собирать рыбу но уменьшать ее количество в море на (1 даунгрейд)". The orchestrator read that as "each kind taken loses one level". The other reading is "one blast costs one downgrade". The orchestrator is asking the director which one he meant. This task prepares the second reading as a data lever with the default unchanged, and measures it, so his answer can be applied by changing numbers.

## 2. What exists (verified by the orchestrator)

- `Catch/Source/CatchCore/Private/Rods.cpp:888`, `Game::SeaCost(rod, source, kinds)`: every kind in `kinds` (unique, in kind order) loses `blastChumCost` owned and active levels, down to `ShopData::chumMinLevel`; never the Gold Sunfish, a prize or a unique kind; one `ChumChanged` per kind.
- `Game::Explode` (`Rods.cpp:840`) passes the kinds of the blasted fish to `SeaCost`.
- `Catch/CoreTests/core_runs.cpp`: the levers `--radius`, `--chum-min`, `--chance`, `--blast-weight` of addendum 5.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. New `PaceData` field `blastCostKinds` (int, default 0). 0 = today's rule: every kind taken pays. N above 0 = at most N kinds pay per blast: among the kinds of this blast that can still lose a level (not protected, above the floor), the ones with the most fish taken in this blast; a tie goes to the lower kind id. One `ChumChanged` per kind that paid, as today.
2. The default stays 0, so the game, the tests and all fingerprints are unchanged. No data file change. Core 2.10.2, API stays 10.
3. Tool lever `--cost-kinds <N>` in `core_runs.cpp`.
4. Tests: with `blastCostKinds` 1 a blast that takes fish of two kinds costs only the kind with more fish; a tie costs the lower kind id; a kind at the floor is skipped and the next one pays; with 0 the behaviour equals 2.10.1; the three switch-off fingerprints are unchanged.
5. Measure, Dynamite forced, careful bot, default mode, Soft, candidates on, 300 runs of 12 days, low priority, at most three processes at once, all with `--cost-kinds 1` and floor -2, weight 0.35:

   | Row | Radius | Chance |
   | --- | --- | --- |
   | P1 | 80 | 1 in 5 |
   | P2 | 160 | 1 in 5 |
   | P3 | 240 | 1 in 5 |
   | P4 | 160 | 1 in 10 |
   | P5 | 240 | 1 in 10 |

   Then on the best row by survival after day 12: Dynamite + Horseshoe, and Dynamite + Bait Bomb. Report the same columns as in addendum 5, next to the control (25.0 %) and the base row (6.0 %).

## 4. Done means

`Tools\core-test.ps1` prints `CORE TESTS PASSED`; `python -X utf8 Tools/check_all.py` has no fail; `git status` shows only `Rods.cpp`, `Tables.h`, `CoreVersion.cpp` (comment), `core_tests.cpp`, `core_runs.cpp` and your report; the report has a section "Addendum 6" with the change, the tests, the table, the command lines and a short reading: does "one kind pays" bring any row into the band 27.0-52.0 %, and which row is the richest blast that stays near the control.

## 5. Boundaries

Write only inside `F:\UE\EpicJam_core`. No data file change, no default change. No Unreal builds (the view executor builds and packages in `F:\UE\EpicJam_ui`). No commits. Time box: 20 minutes. Report to the orchestrator.
