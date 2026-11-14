# Brief: core 2.8 - charms open between runs, by tasks

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `6ee58d5` (core 2.7, API 7).

## 1. Goal

Decision of the game director `unlock-by-tasks` (13.11, Russian, dictated by voice): «первый раз он проходит с простыми наживками и простыми улучшениями, а потом будут постепенно открываться новые. ... по принципу Балатру. ... Наловил много кильки, получил вот эту штуку на стаю ... И то есть вот в таком ключе надо делать, чтобы это было само по себе задание.» On 14.11 he asked whether this system exists. It does not: every one of the 59 charms can be offered in the first run, and the game keeps nothing between runs. Build the core side: a profile that lives across runs, a task per locked charm, offers only from open charms. The task list below is a draft; the director approves it later, so every row must be data.

Scope: charms only. Baits, shop goods and zones keep their in-run opening as it is.

## 2. What exists now (verify before you rely on it)

- Charm rows are generated: `Catch/CoreTests/gen_charm_rows.py` writes `Catch/Source/CatchCore/Private/CharmRows.inc` from `Docs/design/modifiers-v2.data.json` (`--check` fails when the file is stale). 59 charms.
- Offers: `Game::RollCharms`, `Game::RollCharmOffers` (`Shop.cpp`), the prize and reward roll, the fit filter in `Offers.cpp` (a charm is offered only when it can work with the present tackle, bait, fish and zones). Rarity weights `t.shop.charmRarityWeights = {60, 30, 10, 2}`.
- In-run opening: `struct Unlock { int shopDay; Zone zone; }` (`Tables.h:51`), `Game::Unlocked`, `BaitUnlocked`, `NotifyUnlocks`, event `ContentUnlocked` (`Shop.cpp:33`).
- Run counters that some charms already keep (Logbook, Steady Hand, Collector, Fish Tale, chains for Apex): find them before you add new ones.
- Fingerprints of core 2.7: default mode `b0d881d44f065645`, timed mode `7d632f58e4dd608e`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Profile. A plain core struct that outlives a run: the set of open charms, the progress of every task, a version. It has `Serialize()` to a short text (one `key=value` per line) and a tolerant parse: a missing, empty, cut or foreign text gives a fresh profile, unknown lines are skipped, numbers are clamped; a parse never throws and never crashes. The core does no file work; the view saves and loads the text.
2. The game takes a profile (`SetProfile` before or after `NewRun`) and gives it back (`Profile()`), with a cheap way for the view to learn that it changed (a counter or an event) so the view can save. A game that never got a profile behaves as today: every charm is open, no task is counted, no new event is sent. All old tests and both fingerprints must stay as they are.
3. Tasks are data in `Docs/design/modifiers-v2.data.json` next to their charm (a start flag or one task: kind, target, optional fish kind or zone, the English text), generated into the tables by the existing generator. A charm without a task and without the start flag counts as open.
4. Task kinds needed by the draft (add a kind only when a row needs it): total counters across runs (fish of a kind landed, chains landed, escapes, fish landed on a bare hook, on a bait of a rarity, on one bait kind, below Sunlight, bait packs opened, fish cards bought, chum bought, lucky chances that came true); feats inside one run, day or pull (a chain of N links, N fish in one pull, a fish of N kg, a fish that hung N seconds, N fish of one kind in one day, N chains in one day, rods owned, rod level reached, the level limit reached, bait kinds held, Sea charms on the shelf, every shelf slot full, coins held, chum level of one kind, card level of one kind, a zone opened, a day passed, a day passed with only Sunlight open, the Gold Sunfish landed).
5. When a task completes: the charm opens at once in the profile, the core sends `ContentUnlocked` for that charm (reuse the event; add a field only if the view cannot tell a charm from other content), and the charm can be offered from the next roll of offers (shop, reroll, prize). The current offers do not change.
6. Offers: with a profile, every roll of charms (shop, reroll, reward, prize) takes only open charms that pass the fit filter. A rarity with no open charm gives its weight to the open rarities. A roll with fewer open fitting charms than places shows fewer cards, as the fit filter already allows; say in the report what the player sees on day 1.
7. A query for the view: the list of tasks with charm, text, progress, target and done, in a stable order, and the count of open charms.
8. The timed mode counts tasks like the calm mode. Tide Pool keeps its rule (never offered in the timed mode).
9. API number 8. Additions only; nothing the view of API 7 uses may change.

## 4. Draft task list (data, not approved: keep every number in the data file)

Open from the start (20): rod_rack, whetstone, steady_hand, big_haul, chum_bucket, lucky_pack, bait_trader, same_bait, sprat_fever, same_again, big_shoals, collector, small_fry, well_fed, long_day, warm_current, coral_garden, cold_spring, rising_tide, tide_pool.

Tier 1 - most first runs complete it:
- fresh_scent: "Land a chain."
- bait_knife: "Land 30 Sprats."
- deep_pressure: "Open the Twilight zone."
- mixed_bag: "Hold 3 bait kinds at once."
- auto_reel: "Own 3 rods."
- giants: "Land a fish of X kg or more." (X: calibrate)
- storm_glass: "Land the Gold Sunfish."
- fine_bait: "Land a fish on Rare bait."
- dusk_frenzy: "Hold 2 Sea charms at once."
- fishmonger: "Buy 5 fish cards."
- barbed_hooks: "Let 10 fish escape."
- lucky_bite: "Open 10 bait packs."

Tier 2 - two or three runs, or the middle of a good run:
- cluster_hook: "Land 120 Sprats." (the director's own example)
- food_chain: "Land 10 chains."
- apex: "Land a chain of 3."
- old_faithful: "Sharpen a rod to level 4."
- long_leader: "Land 3 fish in one pull."
- slow_hands: "Land a fish that hung for 6 s."
- bare_steel: "Land 30 fish on bare hooks."
- thin_line: "Land 5 chains in one day."
- blue_ribbon: "Raise a fish card to level 4."
- logbook: "Land 25 fish of one kind in one day."
- deep_reel: "Open the Midnight zone."
- shallows: "Pass day 5 with only Sunlight open."
- sea_chest: "Hold 25 coins."
- groundbait: "Buy chum 15 times."
- pure_waters: "Raise the chum of one kind to level 3."
- live_well: "Land 10 fish on Live Sprat."
- free_feast: "Land 20 fish on Rare or Legendary bait."
- rod_stand: "Fill every shelf slot."
- fish_tale: "See 3 lucky chances come true."
- big_mouth: "Land 25 chains."
- bottom_rig: "Land 30 fish below Sunlight."

Tier 3 - a strong run:
- extra_edge: "Sharpen a rod to its limit."
- low_tide: "Open the Abyss zone."
- whopper: "Land a fish of Y kg or more." (Y: calibrate)
- trotline: "Pass day 9."
- tiny_terror: "Land a chain of 4."
- upside_down: "Open the Trench zone."

Use the words the game already uses in rule texts and on screen (chain, pull, bait pack, fish card, chum, shelf slot, zone names). If a draft text names a thing by another word than the game, correct the text and list it.

Calibration (block 4): simulate profiles that live across runs - the careful bot, 30 profiles x 6 runs in the default mode. For every task report in which run it completes (share of profiles after runs 1, 2, 3, 6). Adjust targets (numbers only, not the kind of the task) so each task lands in its tier: tier 1 at least 60 % of profiles after run 1, tier 2 at least 60 % after run 3, tier 3 between 10 % and 60 % after run 6. If the bot cannot do a task because of how it plays (for example it never keeps a bare hook), say so and keep the draft number. Pick X and Y from the fish table.

## 5. Work by blocks

1. Profile, serialization, tolerant parse, tests (garbage, empty, cut text, a profile of a newer version).
2. Tasks as data, counting, `ContentUnlocked`, the query; tests per task kind.
3. Offers from open charms only; tests (a locked charm is never offered; an opened charm can be offered from the next roll; rarity with no open charm; old fingerprints unchanged without a profile).
4. Calibration runs and the survival table of the default mode for a fresh profile, run 1 (300 runs, careful bot, days 4 to 12) next to the 2.7 table. Do not change targets of days or prices.

Extend the report after each block.

## 6. Done means

- New tests pass, all old tests pass, both fingerprints are unchanged without a profile.
- `python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue` prints `11 ok, 0 fail, 0 skip`.
- The report holds: every interface addition with one line on how the view uses it; the serialized text of a sample profile; the final task table (charm, text, target, tier, calibration shares); the survival table; what the player sees in the shop of day 1 with a fresh profile; anything you could not verify.

## 7. Verification commands

```
powershell -NoProfile -ExecutionPolicy Bypass -File F:\UE\EpicJam_core\Tools\core-test.ps1 *> <a log file in your scratch folder>
python -X utf8 F:\UE\EpicJam_core\Catch\CoreTests\gen_charm_rows.py --check
python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue
```

PowerShell 5.1 marks a native program as failed when you use `2>&1`; write the output to a file with `*>` and read the file. The editor build waits for the build of the other worktree (`-WaitMutex`); that is normal. If the orchestrator writes "pause", stop every build and test at once until "resume". Do not kill a process you did not start. Use absolute paths in every command; never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_ui`.

## 8. Boundaries

- You may change only files under `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/modifiers-v2.data.json` (add the task fields only; charm rules, prices and rarities stay), and your report file.
- Do not change `Catch/Source/Catch/**` (the view) beyond what a compile needs; if the view in this worktree stops compiling because of an addition, the addition is wrong - additions must not break API 7 callers.
- Do not change charm rule texts, prices, day targets, config files, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, `Docs/design/core-interface.md`, the check gate, permission settings.
- Do not invent rules beyond this brief. A point that needs a decision goes into the report as a question; go on with the rest.
- No deleting or hiding of old work. No commits, no pushes.

## 9. Report

File: `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core28-unlocks.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 25 lines.

## 10. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
