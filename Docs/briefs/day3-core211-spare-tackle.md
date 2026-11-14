# Brief: core 2.11 - a charm that lets the shop offer charms the player already holds (14.11.2026 21:52)

Executor: the same Opus executor that built core 2.10.0 - 2.10.3, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `694f1b3`. You do not commit.

## 1. Goal

The director played build 18 and decided (decision `charm-allows-duplicates`, 21:50, before the rule freeze at 24:00):

> "Ещё должна быть карточка которая позволяет делает модификаторы не уникальными (т.е. могут появляться уже купленные модификаторы)"

Reading: a new charm. While it is on the shelf, the shop may offer charms the player already holds, and the player can hold several copies of one charm. Without it nothing changes. The game needs it as a candidate (behind `Tables::candidates`, the `-new` switch) tonight or tomorrow morning; code freeze is 15.11 12:00. A small, safe change beats a complete one.

## 2. What exists (verify before you rely on it)

- `State::charms` is a `std::vector<CharmKind>` (`Catch/Source/CatchCore/Public/CatchCore/Game.h:82`), so the state can hold a kind twice. The orchestrator did NOT read how the shop filters held charms, how the rules of a held charm are evaluated (once per kind or once per shelf entry), how a charm is sold or moved, or which events and profile entries identify a charm by kind only. You know this code; write the facts into the report.
- Core 2.10.3 facts you built: `Game::ChanceOf(kind)`, `ChanceBonus` of Horseshoe, the charge sources, `Tables::candidates`, the fingerprints with the switch off: default `b0d881d44f065645`, timed `7d632f58e4dd608e`, profile `4cef488c395857f0`.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one; the names and numbers are temporary until the director speaks)

1. New candidate charm: id `spare_tackle`, English name `Spare Tackle`, Russian name `Запасная снасть`, class `sea`, rarity `rare`, the price that the existing price rule gives a rare charm (no new number), no `needs`. Rule text, English: `The shop can offer charms you already own. Each copy works on its own.` Russian: `Магазин может предлагать модификаторы, которые у вас уже есть. Каждая копия работает сама.`
2. While the player holds Spare Tackle, a charm offer of the shop (and of a reroll) may be a charm the player already holds, with the same weight as any other eligible charm. Rewards, gifts and unlock order stay as they are unless they share that code path cheaply; say which you covered.
3. Each copy takes its own shelf slot and is sold on its own. Selling Spare Tackle does not remove copies the player already holds; it only stops new duplicate offers.
4. Each copy works on its own, exactly as two different charms with the same rule would: added values add, factors multiply, each copy rolls its own chance, each copy fires its own trigger, Horseshoe gives +1 per copy.
5. Per charm a data flag `repeatable`. Default by this test: a second copy must change something for the player and must not break an invariant. A charm that is a pure switch (the second copy does nothing), a charm whose doubling you cannot make safe in the time, and Spare Tackle itself are not repeatable and are never offered twice. Start narrow: it is fine if only the charms with a plain number, factor or chance are repeatable in this version. List every charm with its flag and one reason in the report; the orchestrator shows the list to the director.
6. Anything that names a held charm by kind and becomes ambiguous with two copies (sell, move, an event that says which charm fired, profile rows) gets a shelf index or stays "first copy" where that is harmless. Bump the API to 11 only if a public signature or event changes; list every change for the view executor.
7. With the candidates switch off, and with it on but Spare Tackle never held, the game must be bit-identical to 2.10.3 (the three fingerprints above; say how you checked the second case).
8. Tests: holding Spare Tackle lets a held repeatable charm be offered and bought; a non-repeatable one is never offered twice; two copies of Dynamite roll twice; two Horseshoes give +2; selling one copy keeps the other; the random-command determinism test still passes with duplicates in play.
9. The director is playing build 18 right now. Reading and editing code is fine. Before you compile or run anything heavy (`Tools\core-test.ps1`, the balance tool, the gate) check for a running `Catch` game process. If one runs, do NOT start it and do not wait in a loop: finish the code, write the report with "code ready, not compiled, not tested" and hand back. The orchestrator resumes you when the game is closed.
10. No measure run is required. If the game is closed and time allows, one row: the careful bot, 300 runs of 12 days, Dynamite forced plus Spare Tackle forced, against Dynamite alone.

## 4. Done means

`Tools\core-test.ps1` passes and `python -X utf8 Tools/check_all.py` in `F:\UE\EpicJam_core` has no fail (or the report says plainly that nothing was run because the director's game was open); the fingerprints with the switch off are unchanged; the report holds the repeatable list, the API changes for the view, and what is not verified; `git status` shows only files you meant to change.

## 5. Boundaries

Write only inside `F:\UE\EpicJam_core`. Always use absolute paths (a relative path once read files of the main checkout). Do not touch `F:\UE\EpicJam_ui` or `F:\UE\EpicJam`. Do not change `Docs/design/core-interface.md` (the orchestrator owns it). Invent no other rule and no other number; where a rule question appears that this brief does not answer, write the question down and take the smaller reading. Do not stop any process you did not start. Delete nothing. No commits.

## 6. Report

Add a section "Core 2.11: Spare Tackle" to `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core210-link-charms.md`: how charms were held and evaluated before, what changed by file, the repeatable list, API changes, tests, fingerprints, open questions, what is not verified. Final message to the orchestrator: the same in short. Time box: 60 minutes of work (waiting for the director's game does not count). You report to the orchestrator, not to the director.
