# Brief: balance pass 1 — the norm row in triples, a fair measure of build strength, weak and dominant builds into a band

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 03:25 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_core`, branch `track/core`. Base: the commit the orchestrator names in the launch message (core 2.2 is in it). This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-balance-pass1.md`).

## 1. Goal

Core 2.2 made the strike of a larger fish a real chance (about 40 % instead of about 96 %) and put 53 modifiers into the shop. The numbers around them were tuned for the old game, so the test player now dies far too early and no build pays off in the measurement. Re-fit the level 1 norm row in the owner's three-day rhythm, build a fair measure of how strong each of the 10 builds is, and move weak or dominant builds into a band by changing row values only.

Why it matters. The owner (game director) lost a run on day 15 and asked for drive and for several working build paths. His decisions of 12.11 and 13.11 (voice input, Russian, verbatim):

- `easy-start-hard-later`: «стало ультра хардкорно [...] так должно становиться в будущем, а не в начале. В начале все должно быть чуть-чуть полегче.» Recorded goal of the measure: a skilled player almost always passes day 6, more often than not passes day 9, rarely passes day 12.
- `growth-must-meet-difficulty`: «Нам сделать надо какой-то баланс, чтобы игрок чувствовал, что он уже в какой-то момент, чтобы он мог ощутить сложность. [...] Так-то, в смысле, нормально, 15-й день игры по сложности.» Growth comes from modifier combinations; coins stay tight.
- `day-rhythm-easy-medium-boss`: «в Балатру это же не случайно у него идет легкая игра, средняя игра и босс, потому что ты после босса будешь опять на легкой. Ты чуть-чуть расслабишься, да, там будет больше очков, но босс дает ограничения, а следующая штука нет.» Recorded reading: the norm row repeats a triple: norm, norm x 1.5, norm x 2.
- `drive-and-build-variety`: «Сейчас мы получаем стабильную игру, нам стабильно выгодно делать глубину и ждать, пока рыбина будет одна другую есть [...] чтобы чувак мог взять и сделать автоматический промышловый лов, где удочки сами все дергают. А другой взял бы и гигантских рыбин вытягивал. А третий брал бы и цепочки из рыб делал. [...] А следующий берет и сотнями рыб ловит.»
- `big-fish-path-is-one-scenario`: one boring path is fine as one of several; it must not be the only working path.

## 2. What exists now (verify before you rely on it)

Read first: `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core-charms.md` (the report of the executor who built core 2.2: readings, interface, the measurement table, the build policies). Paths below are under `F:\UE\EpicJam_core\`.

- Norm rows: `Catch/Source/CatchCore/Private/Tables.cpp`, `t.fees`. Soft (level 1, the level the game starts): `10, 35, 55, 90, 140, 220, 400, 500, 600, 1300, 1900, 2800, 4000, 5800, 8400`, then x 1.4 per day, two significant digits. Normal and Hard follow; the owner does not play them yet.
- Payout: base 4 coins, +1 per 25 % over the norm (at most 4), interest 1 per 5 coins held (at most 3). The run opens in the shop with 5 coins. Charm prices 4 / 6 / 8 by rarity, 3 offers per shop, reroll 2 +1.
- The runner: `Catch/CoreTests/core_runs.cpp`. `main` takes `runs days`. It prints three plain policies at three levels, then the build table (`PrintBuildTable`). Policies: `PersonBuys` (person-like, with `CharmGain` as its guess of a charm's worth), `BuildBuys` / `BuildStep` / `SetRigs` / `MakeBuilds` (10 build policies read from the free text `builds[].ground` of the data). `Tools/core-test.ps1` builds `core_tests.exe` and `core_runs.exe` into `Catch/Saved/CoreTest/` and runs them; `-RunBalance -Runs 200 -Days 15` takes about 11 minutes.
- The accepted measure of 12.11 (person-like, Soft, 200 runs), before core 2.2: alive after days 3 / 6 / 9 / 12 / 15 = 100 / 99 / 68 / 22 / 4.5 %. The owner played that build and called the difficulty up to day 15 normal.
- The same measure on core 2.2 now: 97.5 / 72.5 / 19 / 1.0 / 0 %. Median points of the person-like policy by day: about 52, 88, 154, 186, 249, 392, 460, 675, 780, 1130 (days 1-10). They grow by about x 1.35 a day; the norm jumps x 1.8 on day 7 and x 2.2 on day 10.
- The build table of the report: no build policy outlives person-like, and the build policies hold 0.6-1.4 charms at the end of a run. Cause (orchestrator's reading): a build policy buys only its own 5 or so charms out of 53, and a shop shows 3 offers, so a build charm is on offer in about one shop of four. The table measures the luck of the offers, not the strength of the build.
- Data: `Docs/design/modifiers-v2.data.json` (version 2.2): 53 charms with `rows`, `ruleEn`, `ruleRu`; 10 `builds` with `charms` and a free text `ground`. Generator `Catch/CoreTests/gen_charm_rows.py` writes `Catch/Source/CatchCore/Private/CharmRows.inc`; `--check` must exit 0. `core_tests.cpp` has one case per charm with the example of its rule text.

Answers to the open questions of the core 2.2 report (section 9):

1. Shallows: keep `zones_closed` = 6 - zones open. Deeper Line has five levels, so the seabed is a zone the player can open.
2. Groundbait and Logbook: a tie goes to the first kind in table order. Keep.
3. The 40 % strike share is meant for a fish that hangs until it escapes. Keep 0.047.
4. Machine-readable ground targets: yes, see decision 6.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Block A, the norm row (must be finished first):

1. Shape. For every triple k = 1, 2, 3, ...: norm of day 3k-2 = B_k, day 3k-1 = B_k x 1.5, day 3k = B_k x 2, each rounded to two significant digits as the table does now. The table holds days 1-15 (B_1 to B_5). After day 15 the shape continues: B_k+1 = B_k x g with one constant g. If the row model has only a growth per day, extend it by the smallest change (for example a triple growth field) and keep Normal and Hard as they are.
2. Targets, person-like policy, level Soft, 200 runs, 15 days:

   | Measure | Target |
   | --- | --- |
   | Alive after day 3 | at least 98 % |
   | Alive after day 6 | at least 92 % |
   | Alive after day 9 | 55-70 % |
   | Alive after day 12 | 10-25 % |
   | Alive after day 15 | at most 5 % |
   | Median points / norm, days 1-3 | between 2 and 5 |
   | Median points / norm on the easy day of a triple | higher than on its rule day, for every triple |

   B must grow from triple to triple. Prefer the smoothest B row that meets the table. Do not touch the payout, the prices, the start coins, the strike chance or any other number in this block.
3. Speed. Add optional arguments to `core_runs.exe` (for example a policy name, a level, "tables only") so that one person-like run at Soft takes under a minute. Without the new arguments the output keeps its present sections and format; you may append sections. Run the exe directly from `Catch/Saved/CoreTest/` for your loops (after `Tools/core-test.ps1` built it); do not edit `Tools/**`.
4. A test asserts the triple shape of the Soft row for days 1-30.

Block B, the measure of build strength:

5. A new table "BUILD STRENGTH" at level Soft: every build policy gets a guaranteed offer of its next missing charm in every shop, in the order of `builds[].charms`, and buys it at its price as soon as it has the coins, before its ground purchases. Everything else stays as the build policy plays now. Print per build: alive % per day; median points / norm per day; median charms held after days 3, 6, 9, 12. Add two reference rows: person-like, and person-like that never buys a charm. The guaranteed offer is a test seam: it must not change the game a player gets (off by default, or code that lives only in the tests). Name the seam in the report.
6. Put the ground targets of each build into the data as a machine-readable field `groundTargets` (rods, rod level, zones, chum and card levels by fish kind, bait, bare hooks, depth, coin reserve): the current readings of the runner (report 2.2, section 5, item 20), fixed where you find a reading that fights the build's own charms. The runner keeps its own table; say in the report how you keep the two equal (a generated include or an assert, your choice).
7. Economy pace, measure only: for person-like at Soft print per day the median coins after the payout, the median charms held, the share of runs with a full shelf, the median rods and open zones. Compare with these marks: 2 charms held after day 4, 3 after day 7, 4 after day 10. If the pace is under the marks, propose the smallest lever with numbers (charm prices, reroll price, offers per shop) and its effect on the survival table. Propose only; do not apply.

Block C, the band (only after A and B are green and written into the report):

8. Strength of a build = its median points in the BUILD STRENGTH table on days 6, 9 and 12 (runs alive on that day), divided by the median of the 10 builds on the same day. Weak: under 0.6 on two of the three days. Dominant: over 1.7 on two of the three days.
9. For a weak build raise, and for a dominant build lower, the values of the rows of its own charms (charms that belong to that build alone or mostly). Limits: per charm at most x 1.5 up or x 0.67 down in total over this pass; friendly round numbers; no change of a trigger, a condition, a rarity, a price, a rule shape or a name. Change the numbers in `ruleEn` and `ruleRu` to match, regenerate, and update the charm's test case.
10. List every change: charm, row, before, after, the build, its strength before and after. A build that stays weak at the limit stays as it is: report it with your reading of the cause (it may need a rule, and rules are the owner's).
11. After block C run block A again: the person-like policy buys the changed charms too. The final numbers of the report come from this last fit.

Not in this brief: new charms, new rules, the charm prices and rarities, the payout, the shop, Normal and Hard, anything in the view module.

## 4. Work order and stops

A (norm fit; write the report section the moment it holds) -> B -> C -> A again -> the full `-RunBalance -Runs 200 -Days 15` run and the report. If you run short of session, stop after a finished block with green tests and report exactly what is left. Block A alone is already a useful result.

## 5. Done means

1. `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1`: CORE TESTS PASSED, zero warnings.
2. `python -X utf8 Catch/CoreTests/gen_charm_rows.py --check`: exit 0.
3. `python -X utf8 Tools/check_all.py`: `10 ok, 0 fail, 1 skip`.
4. The person-like row at Soft, 200 runs, meets the table of decision 2. Print it in the report with the final norm row for days 1-18.
5. The BUILD STRENGTH table and the economy pace table are in the output of `-RunBalance -Runs 200 -Days 15` and in the report.
6. After block C: no build is dominant; every weak build is either inside the band or listed with the cause.
7. `git diff --stat` shows only the files of section 7.

## 6. Verification commands (from `F:\UE\EpicJam_core`)

```
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 15
python -X utf8 Catch/CoreTests/gen_charm_rows.py --check
python -X utf8 Tools/check_all.py
git diff --stat
```

`Tools/core-test.ps1` builds into one folder of the worktree: never run two copies of it at the same time. Long runs go to the background or get a long timeout. Other executors build and run the Unreal game on this machine now, so runs may take longer than the times above.

## 7. Boundaries

- You may change: `Catch/Source/CatchCore/Private/Tables.cpp` (the Soft norm row); the norm row model in `Catch/Source/CatchCore/**` only as far as decision 1 needs; `Docs/design/modifiers-v2.data.json` (values of rows and the numbers in their texts, `groundTargets`, a version note 2.3); the regenerated `CharmRows.inc`; `Catch/CoreTests/**`; your report. Nothing else.
- No change to what a rule means. No new public command or query of `Game` except the smallest test seam for decision 5 (list it for `core-interface.md`).
- Do not run UnrealBuildTool, the editor or the packager. Do not touch `F:\UE\EpicJam`, `F:\UE\EpicJam_ui`, `F:\UE\EpicJam_look`, `Catch/Source/Catch/**`, `Tools/**`, `CLAUDE.md`, `AGENTS.md`, `Docs/design/core-interface.md`.
- English only in every file under `Catch/`. The Russian `ruleRu` texts live in the JSON under `Docs/`.
- No commits, no pushes.

## 8. Report

Write `F:\UE\EpicJam_core\Docs\briefs\reports\day2-balance-pass1.md` in English and hand back a short summary: the norm row before and after with the survival table; how you fitted it; the BUILD STRENGTH table before and after block C; the economy pace table and your proposal; every changed charm value; the test seam; decisions you changed; what is left; questions.

## 9. Chain of command

You report to the orchestrator, not to the owner. Answer technical questions yourself and list them as revocable readings. A question about what the game should be goes into the report, and you continue with the parts that do not depend on it.
