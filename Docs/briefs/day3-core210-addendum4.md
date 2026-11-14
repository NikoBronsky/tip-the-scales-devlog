# Addendum 4 to brief day3-core210-link-charms: a charge that is still flying when the day ends explodes (14.11.2026 15:50)

Executor: the same Opus executor, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `0653f2d` (core 2.10.0, your work, accepted and committed by the orchestrator). The brief and addenda 1-3 hold except where this file says otherwise. You do not commit.

## 1. The director's word

Your open question 2 went to the director: "Заряд ещё летит, а день закончился: взорвать его или пусть пропадает?" He answered: "Взорвать". Recorded as decision `charge-explodes-at-day-end`.

## 2. What exists (verified by the orchestrator)

- `Catch/Source/CatchCore/Private/Days.cpp:147-162`, `Game::EndDay()`: pulls every rod, resets the Shockwave lift (`:159`), then `state_.charges.clear()` (`:160`), then `ChangePhase(Phase::Shipping)`.
- `Catch/Source/CatchCore/Private/Rods.cpp:840-861`, `Game::Explode(const Charge&)`: the `Blast` event at the point, `LandExtra` for every fish, `SeaCost`, `AfterBlast`.
- `Days.cpp:163-179`, `Game::TickDayEnd`: shipping starts only when every rod is idle and every catch item has arrived, so fish that land at the day end are waited for.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. At the day end every charge in `State::charges` explodes at the point where it is, in throw order, by the normal `Explode` path: `Blast` (with `id`, point, `count`, `otherFishIds`), the `FishLanded` events, `ChumChanged`, and the price as always. Then the list is empty. No charge vanishes any more.
2. The fish of these blasts are today's catch: they count in today's points before the sale, they fly to the bucket, and the shipping waits for them like for the fish of the last pulls. If they lift the total over the day's target, the core behaves exactly as it does when the last pulls of `EndDay` do that.
3. Your decision 5 stays: a source that succeeds at the day-end moment throws no new charge.
4. Shockwave: the lift still ends at 0 when the day ends. Explode first, then reset the lift (or skip the lift for these blasts); pick the simpler one and say which.
5. These fish roll no chance and fire no source (the loop guard of addendum 3 holds). The Bait Bomb does not fire.
6. Version: core 2.10.1, API stays 10 (no interface change). Update the comment in `Game.h` that says a flying charge "is removed with no event", and the version comment.

## 4. Tests to add

A charge flying at the day end explodes: one `Blast` with its id at its current point before `DayEnded`; its fish are in the catch list and in today's points; each kind taken loses a level; `State::charges` is empty after; two flying charges explode in throw order; a blast that takes nobody at the day end costs nothing; with Shockwave on the shelf `State::zoneLift` is 0 after the day end; the timed mode ends its day the same way; candidate switch off: the three fingerprints equal `b0d881d44f065645`, `7d632f58e4dd608e`, `4cef488c395857f0`.

## 5. Measure

Careful bot, default mode, candidates on, 300 runs, 12 days, alive after day 12, below-normal priority, at most three processes at once: control; Dynamite forced; Dynamite + Horseshoe forced. Report the three numbers next to the old ones (25.3 %, 8.0 %, 1.3 %), charges thrown against charges exploded, and kinds erased per run. Propose nothing new; do not tune.

## 6. Done means

`Tools\core-test.ps1` prints `CORE TESTS PASSED`; `python -X utf8 Tools/check_all.py` has no fail; `git status` shows only files you meant to change; the report `Docs/briefs/reports/day3-core210-link-charms.md` has a new section "Addendum 4" with the change by file, the tests, the three measures and anything not verified.

## 7. Boundaries

Write only inside `F:\UE\EpicJam_core`. The view executor works in `F:\UE\EpicJam_ui` and builds Unreal there; do not start Unreal builds. Invent no rule and no number. No commits. Time box: 20 minutes. Report to the orchestrator.
