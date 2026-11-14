# Brief: level 1 is easier at the start (core numbers)

Written by the orchestrator on 12.11.2026, about 23:00. Executor: Opus, effort high.
Worktree: `F:\UE\EpicJam_core`, branch `track/core` (HEAD `83c3d8b`). Nothing else.

## 1. Goal

The owner played the build on points and said (voice, 12.11, about 22:40):

> «стало ультра хардкорно, ультра тяжело. И я как бы не против, чтобы так становилось, но так должно становиться в будущем, а не в начале. В начале все должно быть чуть-чуть полегче. [...] по основе Балатру, там разные уровни сложности. То есть сначала у тебя нет модификаторов, все достаточно легко. Потом у тебя добавляются модификаторы»

Recorded as decision `easy-start-hard-later` in `Docs/decisions/game-frame.answers.json` (main checkout `F:\UE\EpicJam`).
Earlier decisions that still hold: `money-like-balatro` (coins are tight, prices are small), `no-day-locks-money-tight`,
`progression-goal` («сначала тесно и по одной вещи, потом сборка комбинации, потом большие деньги и много рыбы. Очень сложной игру не делаем»).

Your job: make the first difficulty level gentle at the start and harder later, by changing data numbers only, and prove it
with the balance program. You do not invent rules.

## 2. What exists now (verified by the orchestrator)

- Levels: `enum class FeeLevel { Soft, Normal, Hard }` (`Catch/Source/CatchCore/Public/CatchCore/Tables.h:26`).
- Norm tables, `Catch/Source/CatchCore/Private/Tables.cpp:101-105`:
  Soft `15, 40, 65, 100, 165, 260, 420, 670, 1070`, then x1.6; Normal `15, 50, 100, 200, 400, 800, 1600, 3200, 6400`, then x2;
  Hard `15, 60, 150, 380, 950, 2400, 6000, 15000, 37500`, then x2.5. `NormOf`: `Private/Prices.cpp:20-33`.
- Payout, `Tables.cpp:123` and `Prices.cpp:43-60`: base 4; over the norm 1 coin per 25 % of the norm, at most 4;
  interest 1 coin per 5 coins held, at most 3.
- Prices in coins, `Tables.cpp:92-99` and `:115-122`: charms 4 to 8, fish card 3 (+1), worm tin 1, reroll 2 (+1),
  chum 3 (+1 per level), packs 2 and 4, New Rod 5, Sharpen Rod 2, Deeper Line 6, Bigger Shelf 10. Start coins 0 (`Tables.cpp:142`).
- Rule days: every third day, fixed cycle (`Private/Days.cpp:8-14`, `Tables.cpp:108-114`). Day 3 is "Fish ignore Worm."
- Balance program: `Catch/CoreTests/core_runs.cpp`. Run:
  `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 12`
- Measurement today, policy person-like, 200 runs (alive % after the day / median points / norm):

  | Day | Soft | Normal |
  | --- | --- | --- |
  | 3 | 100 / 226 / 65 | 100 / 236 / 100 |
  | 4 | 100 / 255 / 100 | 83 / 255 / 200 |
  | 5 | 96 / 251 / 165 | 20 / 255 / 400 |
  | 6 | 62.5 / 290 / 260 | 0.5 / 556 / 800 |
  | 7 | 22.5 / 300 / 420 | 0 |
  | 8 | 1.5 / 240 / 670 | 0 |
  | 9 | 0 | 0 |

  Firsts (median day, share of runs): chain 2 (100 %), charm 4 (1 %), deeper line 2 (100 %), second rod 1 (100 %), chum 4 (37 %).
  Median coins after the payout: 6 to 12.
- The orchestrator's reading: the catch stops growing near 250 to 300 points from day 3 on every level. The policy almost
  never buys a charm. So the wall is not only the norm table: growth is not bought.
- The interface executor (another worktree) switches the default level of a new run to Soft. Soft is "level 1" from now on.
  Normal and Hard become levels 2 and 3. A level picker and extra conditions per level are a later layer, not your task.

## 3. Technical decisions (revocable: change one if it breaks something, and say so in the report)

1. Only data numbers change. No new rule, command, event or state field. Exception: if a number must differ per level
   (for example the payout), you may make that one value per level; say so.
2. Knobs you may turn: the Soft norm table and its growth; the payout numbers; shop prices; start coins.
3. Knobs you must not turn without the orchestrator: fish weights, points per kg, charm effects, bait tables, day rules,
   zone depths, day length, escape timers, the food ladder. If the goal cannot be met without them, stop tuning, show the
   evidence and propose the smallest change.
4. Goal for level 1 (Soft), policy person-like, 200 runs, 15 days:

   | Measure | Goal |
   | --- | --- |
   | Alive after day 3 | 100 % |
   | Alive after day 6 | at least 95 % |
   | Alive after day 9 | 55 to 70 % |
   | Alive after day 12 | 10 to 25 % |
   | Alive after day 15 | at most 5 % |
   | Median points / norm, days 4 to 9 | 1.15 to 2.0 (pressure stays) |
   | Runs with a charm bought by day 5 | at least 60 % |
   | Median coins after the payout, days 1 to 6 | at most 14 (coins stay tight) |
   | Policy saves-everything alive after day 6 | at most 10 % (a player who buys nothing must fall) |

   The goal is a target, not a law. Get as close as the numbers allow and report the distance.
5. Audit the policy first. If person-like buys no charm because its value estimate is wrong, fix the policy (test code)
   and say so. If it buys no charm because coins are too few, that is a numbers finding.
6. You may shape the Soft norm table in steps of three days (two plain days, then the Gold Sunfish day as the peak),
   like antes in Balatro. Your choice, by measurement.
7. Normal and Hard keep their norm tables. Payout and prices are shared, so report the same tables for Normal and Hard
   after your change. Normal must stay harder than Soft on every day.

## 4. Work

- Block A: audit the person-like policy and the coin flow (where do coins go on days 1 to 5; what does growth cost).
- Block B: tune the allowed knobs toward the goal. Keep a log of each try (numbers, result) in the report.
- Block C: update tests that pin a changed number (no test deleted), run everything, write the report.

Files you may touch: `Catch/Source/CatchCore/Private/Tables.cpp`, `Catch/Source/CatchCore/Public/CatchCore/Tables.h`
(only for decision 1's exception), `Catch/Source/CatchCore/Private/Prices.cpp` (only for the same exception),
`Catch/CoreTests/core_runs.cpp`, `Catch/CoreTests/core_tests.cpp`, `Docs/briefs/reports/day1-core-level1.md`.

## 5. Done means

- Core tests pass.
- The balance run of section 6 prints the tables; the report shows before and after for three policies on three levels.
- Every changed number is listed: file, before, after, why.
- The distance to each goal row is stated.

## 6. Verification (run exactly)

```
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 15
python -X utf8 Tools/check_all.py
```

Do not run `check_all.py --ue` and do not start UnrealBuildTool: another executor builds Unreal in `F:\UE\EpicJam_ui`.

## 7. Boundaries

- Work only in `F:\UE\EpicJam_core`. Do not touch `Catch/Source/Catch` (the view) or anything in `F:\UE\EpicJam_ui` and `F:\UE\EpicJam`.
- No Unreal editor, no Unreal build, no assets.
- Do not edit `Docs/design/core-interface.md`, `CLAUDE.md`, `AGENTS.md`, the journal or `DEVLOG.md`.
- No commits. No deletions. No new gameplay.
- A missing owner decision: write the question in the report and continue with what needs no answer.

## 8. Report

`F:\UE\EpicJam_core\Docs\briefs\reports\day1-core-level1.md`, in English: state and gate; the audit finding; the table of
changed numbers; the balance tables before and after; the distance to the goal; decisions of section 3 you changed; questions.

## 9. Chain of command

You report to the orchestrator, not to the owner. Questions go into the report. The orchestrator answers technical
questions and takes director-level ones to the owner.
