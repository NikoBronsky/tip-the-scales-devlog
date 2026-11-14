# Addendum 7 to brief day3-core210-link-charms: one kind pays per blast becomes the rule; blast radius 160 (14.11.2026 17:16)

Executor: the same Opus executor, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `463a3e2` (core 2.10.2, accepted and committed by the orchestrator). You do not commit.

## 1. The director's word

The orchestrator showed him your two grids and asked: who pays a chum level for one blast? He answered: "Один вид за взрыв". Recorded as decision `blast-costs-one-kind`. It replaces the reading "every kind taken loses a level".

## 2. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. `PaceData::blastCostKinds` default becomes 1 (your lever of 2.10.2: the kind with the most fish in this blast pays, among the kinds that can still lose a level; a tie goes to the lower kind id). Update the comments that call "every kind pays" the rule.
2. `PaceData::dynamiteRadius` default becomes 160 (a temporary number of the orchestrator: at 80 a blast takes about two fish and a third of the blasts are empty). The Harpoon, the Chain Shot and every other number stay.
3. Core 2.10.3, API stays 10. No data file change (`Docs/design/modifiers-v2.data.json` stays untouched: the view branch edits it too, and the price text lives in the view glossary).
4. Fix the tests that relied on the old defaults, keeping what each one proves. Keep the test of 2.10.2 that checks both values of the lever.
5. The three switch-off fingerprints must stay `b0d881d44f065645`, `7d632f58e4dd608e`, `4cef488c395857f0`.
6. Measure once at the new defaults (no levers), careful bot, default mode, Soft, candidates on, 300 runs of 12 days, low priority, at most three processes at once: control; Dynamite forced (expect your row P2: 10.0 %); Dynamite + Horseshoe; Lucky Bite + Lucky Fuse; Dynamite + Shockwave; Dynamite + Bait Bomb. Report alive after day 10 and day 12, fish per blast, chum levels lost per run, kinds erased per run. Propose nothing; do not tune.

## 3. Done means

`Tools\core-test.ps1` prints `CORE TESTS PASSED`; `python -X utf8 Tools/check_all.py` has no fail; `git status` shows only files you meant to change (`Tables.h`, `CoreVersion.cpp`, `core_tests.cpp`, the report, and `Rods.cpp` only if a comment needs it); the report has a section "Addendum 7" with the change, the tests you had to touch and why, the measures, and anything not verified.

## 4. Boundaries

Write only inside `F:\UE\EpicJam_core`. Always use absolute paths (your last task read files from the main checkout by a relative path). No Unreal builds. The director may start playing build 16 on this computer at any time: run the tests and the measures at below-normal priority. No commits. Time box: 15 minutes. Report to the orchestrator.
