# Addendum 5 to brief day3-core210-link-charms: measure the balance levers of the dynamite (tool only) (14.11.2026 16:45)

Executor: the same Opus executor, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `d13053f` (core 2.10.1, accepted and committed by the orchestrator). You do not commit.

## 1. Goal

The director plays the new charms tonight (build 16, about 19:00) and then picks a balance lever before the rule freeze at 24:00. The measures of 2.10.1 say Dynamite kills the run (alive after day 12: 6.0 % against a control of 25.0 %, band 27.0-52.0 %). The orchestrator wants a table of levers ready before he plays, so that his choice can be applied in minutes. This task changes no game number: it adds levers to the measuring tool and runs a grid.

## 2. What exists (verified by the orchestrator)

- `Catch/Source/CatchCore/Public/CatchCore/Tables.h`: `dynamiteRadius = 80.0f` (`:394`), `dynamiteWeight = 0.35` (`:396`), `blastChumCost = 1` (`:417`), `ShopData::chumMinLevel = -2` (`:484`). The sea is 1000 wide and 3400 deep, so a radius of 80 is small; your report says 44 % of the blasts take no fish and a blast takes 1.9 fish.
- `Catch/CoreTests/core_runs.cpp:2178-2282`: the argument parser with "this run of the tool only" levers (`--set id:rN=value`, `--charm-offers`, `--force`, `--candidates`). There is no lever for the radius, the chum floor or the chance of a source.

## 3. Technical decisions (orchestrator; each revocable)

1. Add tool-only levers to `core_runs.cpp`, in the style of the existing ones: `--radius <units>` (`pace.dynamiteRadius`), `--chum-min <level>` (`shop.chumMinLevel`), `--chance <charm id>=<N>` (the first `Chance` condition of that charm becomes 1 in N), `--blast-weight <factor>` (`pace.dynamiteWeight`). The game tables, the data file and the core sources stay untouched.
2. Grid, each row: careful bot, default mode, Soft, candidates on, Dynamite forced, 300 runs of 12 days, below-normal priority, at most three processes at once.

   | Row | Radius | Chance | Chum floor | Weight |
   | --- | --- | --- | --- | --- |
   | base | 80 | 1 in 5 | -2 | 0.35 |
   | A | 160 | 1 in 5 | -2 | 0.35 |
   | B | 240 | 1 in 5 | -2 | 0.35 |
   | C | 80 | 1 in 10 | -2 | 0.35 |
   | D | 80 | 1 in 5 | -1 | 0.35 |
   | E | 240 | 1 in 5 | -1 | 0.35 |
   | F | 240 | 1 in 10 | -2 | 0.35 |
   | G | 160 | 1 in 5 | -1 | 0.35 |
   | H | 240 | 1 in 5 | -2 | 0.6 |

3. For each row report: alive after day 12, charges per played day, share of blasts that take no fish, fish per blast, points landed by blasts per run if the tool can count it cheaply, chum levels lost per run, kinds erased per run. Next to it the control (no forced charm), measured once.
4. One more pair with the best two rows by survival: Dynamite + Horseshoe forced, to see whether the pair stays playable.

## 4. Done means

`Tools\core-test.ps1` still prints `CORE TESTS PASSED` (you touched only the tool, so run it once at the end); the base row repeats 6.0 % (or you explain the difference); the report `Docs/briefs/reports/day3-core210-link-charms.md` has a new section "Addendum 5" with the table, the exact command lines, and three sentences of reading: which lever moves survival most, which makes the blast richest (fish per blast), and what each costs. Propose at most two combinations; do not tune the game.

## 5. Boundaries

Write only `Catch/CoreTests/core_runs.cpp` and your report inside `F:\UE\EpicJam_core`. No change to `Catch/Source`, the data file or the tests. No Unreal builds (the view executor builds and packages in `F:\UE\EpicJam_ui`; the machine is busy, keep the priority low). No commits. Time box: 25 minutes. Report to the orchestrator.
