# Brief: core 2.7 - two bugs of the new charms, one rule check, one comment

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `e776b8d` (core 2.6, API 7).

## 1. Goal

The view executor of part 9b found two core bugs while it showed the six new charms in the world. Both make a charm lie to the player, so they block the next build for the game director. Fix them with tests, check one rule reading, and correct one comment. No new rules, no new numbers.

## 2. What exists now (verify before you rely on it)

- Whopper (`CharmRows.inc`: "Once a day a fish ten times larger enters the sea. It weighs x10."). `Fish.cpp:131` gives the entity `sizeFactor = mods_.whopperFactor`. `Prices.cpp:244` reads the factor through `FindFish(id)`; `Prices.cpp:185` adds the Whopper effect when `context.sizeFactor != 1`. The fish has already left the sea when it lands, so the factor falls back to 1. Case from the view (`CheatRun 42 !few Whopper`): the fish enters as "Mackerel #9 x10", its rod card shows 50 pts while it hangs on the hook, then the log says `FishLanded rod 0 Mackerel #2 5 pts 5.0 kg`. Expected: 50 kg and 50 pts, also in the bucket, at the weigh-in and in the day totals.
- Upside Down (`state_.zonesMirrored`, set in `Days.cpp:310`; `Fish.cpp:15-21` `EffectiveZone`; `Fish.cpp:241` sets `entity.zone` in the population refresh). Case from the view (`FrameRun 42 UpsideDown`, bought, next day): Mackerels stay at depths 55, 78, 266 in Sunlight and bite Worm hooks there, while `EffectiveZone` puts Mackerel in the last zone on that day. Fish that were in the sea before the mirrored day keep their depth.
- Bottom Rig ("The lowest hook of each rod can go one zone deeper than your line.") with Trotline ("Every rod has 2 more hooks."): `Charms.cpp:315-353` puts the Trotline hooks at `hooks.begin()` and sets `maxGoal`. At goal depth 700 with two zones open, all three hooks of the rod hang inside the closed zone.
- `Game.h:125-126` says the Trotline hooks are "the last extraHooks entries of `hooks`"; the code inserts them at the front.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Whopper: the weight factor belongs to the fish from the moment it enters the sea until the day is settled. Every number the core gives for that fish uses it: the hooked value, `FishLanded`, the catch list, the weigh-in, the day totals, `Details`. Sweep every reader of `sizeFactor` and of `FindFish` on a fish that may have left the sea. A Whopper that a predator eats: follow the rule the code already has for an eaten fish; report what it is.
2. Upside Down: on a day where the zones are mirrored, no fish is in a zone where it does not live that day. When the mirror turns on or off between days, the fish already in the sea move to their zone of the new day (a depth inside that zone, by the same placement the refresh uses for a new fish of that kind) or leave the sea and come back through the normal refresh - pick the one that keeps the population counts right and the event log deterministic. Kinds whose zone of the day is closed follow the rule the code already has for closed zones.
3. Bottom Rig with Trotline: the reading of the rule text is literal - only the lowest hook of a rod may fish in the zone below the open ones. A higher hook that hangs inside a closed zone catches nothing there. Find out what the core does now; make it match this reading; add a test. Give the view a way to ask it if none exists (for example through `WouldBite` with the hook index), and say in the report what the view must call.
4. Correct the `Game.h` comment about the Trotline hooks.
5. The public interface stays compatible with API 7: the view executor of part 9c builds against it at the same time. Additions are allowed; list each in the report. Raise the API number only if you add something.
6. Balance: the Whopper now pays what its text says. Rerun the survival table of the default mode (300 runs, careful bot, days 4 to 15) and the timed table (days 3 / 6 / 9 / 12 / 15) and put both next to the 2.5 / 2.6 numbers in the report. Do not change targets or prices; the orchestrator decides after the numbers.

## 4. Work by blocks

Order: decision 1 with tests; decision 2 with tests; decisions 3 and 4; decision 6. Extend the report after each block.

## 5. Done means

- New tests: a landed Whopper weighs and scores x10 in `FishLanded` and in the day totals; with Upside Down, after the day starts, every fish in the sea is in `EffectiveZone(kind)`, and after the charm leaves the shelf the next day puts them back; Bottom Rig with Trotline follows decision 3.
- All old tests pass. The fingerprint of the default mode stays `b0d881d44f065645`; if it moves, the report says which event moved and why.
- `python -X utf8 Tools/check_all.py --ue` in the core worktree prints `11 ok, 0 fail, 0 skip`.
- The report lists every changed file, every interface addition, the two survival tables, and anything you could not verify.

## 6. Verification commands (run from `F:\UE\EpicJam_core`)

```
powershell -NoProfile -ExecutionPolicy Bypass -File F:\UE\EpicJam_core\Tools\core-test.ps1 *> <a log file in your scratch folder>
python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue
```

PowerShell 5.1 marks a native program as failed when you use `2>&1`; write the output to a file with `*>` and read the file. The editor build waits for the build of the other worktree (`-WaitMutex`); that is normal. If the orchestrator writes "pause", stop every build and test at once until "resume". Do not kill a process you did not start. Use absolute paths in every command; never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_ui`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, and your report file.
- Do not change `Catch/Source/Catch/**` (the view), charm rule texts, prices, targets, config files, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, `Docs/design/core-interface.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game. A point that needs a decision goes into the report as a question; go on with the rest.
- No deleting or hiding of old work. No commits, no pushes.

## 8. Report

File: `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core27-fixes.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
