# Brief: view 2.10 follow-up - the new blast price in the texts, core 2.10.3, playable build 17 (14.11.2026 17:18)

Executor: the same Opus executor that built view 2.10 (blocks 1 and 2), effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base `3ae83ea` (your blocks 1 and 2, accepted and committed by the orchestrator). You do not commit.

## 1. Goal

The director answered the price question after your build 16 was packaged: "Один вид за взрыв" (decision `blast-costs-one-kind`). One blast now takes one chum level from one kind: the kind the blast took the most fish of. The orchestrator also raised the blast radius from 80 to 160 (temporary number). The core executor builds this as core 2.10.3 (two default numbers, API 10 unchanged). The director wants to play it tonight; rules freeze at 24:00. Result: build 17 in `F:\UE\EpicJam_ui\Packaged\v17\Windows`. Build 16 (`Packaged\v16`) stays untouched.

## 2. What exists (verify before you rely on it)

- `track/ui` at `3ae83ea` holds core 2.10.1 (`d13053f`) content, merged from `0653f2d` plus the five files of `d13053f`.
- `track/core` will hold core 2.10.3 on top of `463a3e2` (2.10.2: `PaceData::blastCostKinds`; tool levers in `core_runs.cpp`). The orchestrator sends you the accepted commit id in the message that starts this task.
- The glossary term "Charge of dynamite" (`Catch/Source/Catch/CatchUiTerms.cpp`, Russian row in `Docs/design/text-ru.tsv`) ends with "Each kind it takes loses 1 chum level for good."
- The blast ring is drawn with `pace.dynamiteRadius`, so the larger radius needs no view number.
- Your open question 1 of block 2: the `ruleRu` texts in `Docs/design/modifiers-v2.data.json` still say "прикорма".

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. Bring the core commit in with `git merge --no-commit <commit>` (no commit by you). The core side did not touch `Docs/design/modifiers-v2.data.json` after `0653f2d`, so no conflict is expected there. If the merge conflicts, stop and tell the orchestrator.
2. Glossary text, English: `Charge of dynamite: it bounces off the walls and blasts after a random time. Fish near the blast land too, at 35% weight. The kind it took the most fish of loses 1 chum level for good.` Russian: `Заряд динамита: отскакивает от стенок и взрывается через случайное время. Рыба рядом со взрывом тоже попадает в улов, с весом 35%. Вид, чьей рыбы взрыв забрал больше всего, навсегда теряет 1 уровень прикормки.` Check that the tip still fits its box in both languages.
3. Russian layer only: in the `ruleRu` texts of `Docs/design/modifiers-v2.data.json` (the `track/ui` copy) replace the noun "прикорм" in its forms by "прикормка" in the right case; leave the adjective "прикормленный". Regenerate what depends on it. Do not touch English texts or any number in that file.
4. No other view change. The eight TEMP pictures stay.
5. Launch files of build 17: the same five `.bat` files as in build 16.
6. The director may be playing build 16 on this computer. Before every heavy step (editor build, packaging, a proof game) check for a running `Catch` game process that you did not start. If there is one, do not start the step: tell the orchestrator and wait. Proof games run with `-RenderOffScreen -unattended -nosplash -nosound -log`, one at a time, `-ExecCmds` quoted.

## 4. Work and proofs

Merge the core; edit the texts; `python -X utf8 Tools/check_all.py --ue`; package with `Tools\package-win64.ps1` into `Packaged\v17`; then on the packaged build with `-lang=ru -new -coins=300`:
- the shelf tip of Dynamite shows the new glossary text (frame);
- a day with Dynamite (and Lucky Fuse or Powder Keg): a blast that takes fish of two or more kinds shows exactly one "-1 прикормка" word, for the kind with the most fish (frame and the log lines of the `Blast` and `ChumChanged` events);
- the blast ring is visibly larger than in build 16 (frame);
- one run without arguments: English, no candidate (frame or log).

## 5. Done means

The gate with `--ue` has no fail (if `core-test` times out under load, run `Tools\core-test.ps1` alone and report both); build 17 starts with each launch line; the proofs are in the report with frame paths; no Cyrillic under `Catch/` beyond what the gate allows; `git status` shows only files you meant to change; never add `Catch/SourceArt/Fish/atlas.png` or `atlas.json`.

## 6. Boundaries

Write only inside `F:\UE\EpicJam_ui`. Always use absolute paths. Do not touch `F:\UE\EpicJam_core` or `F:\UE\EpicJam`. Invent no rule and no number. Never click, close, move or answer a system dialog. Do not stop any process you did not start. Delete nothing; do not touch `Packaged\v14`, `v15`, `v16`. No commits.

## 7. Report

Add a section "Build 17" to `Docs/briefs/reports/day3-view210-charge-and-links.md`: changes by file, the gate, the proofs with frame paths, decisions you changed, open questions, what is not verified. Final message to the orchestrator: the same in short. Time box: 35 minutes. You report to the orchestrator, not to the director.
