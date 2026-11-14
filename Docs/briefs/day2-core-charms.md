# Brief: modifier rows in the core (block B6), strike chance as a stat, run flow changes

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 02:20 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_core`, branch `track/core`, base `5738853`. This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-core-charms.md`).

## 1. Goal

Put the designed modifier list into the rules core as data rows behind one rule evaluator, make the predator strike chance a stat the player can build on, and change three things in the run flow. All of it is engine-free C++ in `CatchCore` with tests.

Why it matters. The owner (game director) played the packaged build to day 15 on 13.11 and said the game is monotonous. His words (voice input, Russian, verbatim):

- «Сейчас мы получаем стабильную игру, нам стабильно выгодно делать глубину и ждать, пока рыбина будет одна другую есть, и все, и ждать. Вот весь геймплей.»
- «Просто я хотел бы видеть какое-то безумие. [...] чтобы чувак мог взять и сделать автоматический промышловый лов, где удочки сами все дергают. А другой взял бы и гигантских рыбин вытягивал. А третий брал бы и цепочки из рыб делал. [...] А следующий берет и сотнями рыб ловит. Ну, типа связками.»
- «Балатор я при этом на последних уровнях чувствую, как я сделал какую-то свою супер-пупер комбинацию, которая работает. А здесь у меня всегда одинаково.»
- «визуально выглядит практически всегда рыба хватается за цепочку. [...] на саму вероятность мы не влияем. [...] мне хочется, как в рыбалке, чтобы игрок на что-то влиял и понимал бы стратегию, то есть чтобы у него был какой-то азарт. [...] Можно добавлять аналоги джокеров [...] счастливая карта, там, которая там с каким-то шансом отрабатывает.»
- «некоторые дебафы было бы неплохо. [...] штуки, [...] которые давали плюсы и минусы.»

The cause, found by the orchestrator: the game still has the 20 old modifiers, hard-coded in a switch. The designed list (50 modifiers, 10 lines of play, each with a ground the player must prepare) was probed in a browser page on 12.11 and never reached the core. Recorded decisions behind this brief (`Docs/decisions/game-frame.answers.json` in the main checkout): `drive-and-build-variety`, `growth-must-meet-difficulty`, `chain-chance-is-a-stat`, `upgrades-with-tradeoffs-and-synergy`, `run-starts-with-shop`, `rule-days-random`, `rule-only-small-fish-count`, and the earlier one about ground and payoff («Модификатор срабатывает, когда игрок подготовил для него почву»).

The list goes into the branch as temporary data. The owner judges it by playing a build, so every number must stay a table value that one edit can change.

## 2. What exists now (verified by the orchestrator)

Paths are under `F:\UE\EpicJam_core\`.

- Core size: `Catch/Source/CatchCore/Private/*.cpp` and `Public/catchcore/*.h`, about 2750 lines. Tests: `Catch/CoreTests/core_tests.cpp` (1301 lines), runner `core_runs.cpp` (481 lines).
- `Private/Shop.cpp:29` `Game::CompileMods()`: a switch over 12 of the 20 charm kinds fills `Mods` (`Public/catchcore/Game.h:162-171`): `noEscape`, `biteRate`, `hunterRate`, `chainBonus`, `extraDaySeconds`, populations, `sizeMultiplier`, `zoneShift`, `clusterCount`, `packExtraChoices`, `extraShelfSlots`. All flags are global (not per rod, not per kind).
- `Public/catchcore/Tables.h:23` `namespace charm { enum ... }` (20 names); `:81-88` `CharmData {name, category, price, unlock, value, secondary, tertiary}`; `Game.h:103` `ShelfItem {kind, weightless, paid}` (no counter).
- Weight and points: `Game::WeighFish` in `Private/Prices.cpp` runs the v2.1 combine order (steps 1-9). The v2 report says where B6 hooks in: `Docs/briefs/reports/day1-core-points.md`, section 6 and the charm mapping in section 3.
- Eating: `Private/Rods.cpp`, `Game::TickHooked` (about lines 407-468). First check at a random 0.6-1.2 s (`hunterFirstSoonest/Latest`), then every `hunterCheckSeconds` 0.6 s. One roll per check for the hook: `if (RandomUnit() >= pace.strikeChance * factor) return;` with `strikeChance = 0.28` (`Tables.h:169`), `factor = mods_.hunterRate x baits[bait].hunterMultiplier`. A fish hangs `hangSeconds` 6.0 s (`Tables.h:41`) plus `hangPerRodLevel` 1.0 s per rod level (`Tables.h:103`). So with an eater in reach the strike is about 95 % certain before the escape. Only Fresh Scent changes the factor (`Shop.cpp:44`).
- Rule days: `Private/Days.cpp:8-14` `Game::RuleOf(day)` returns `tables_.dayRules[(day / period - 1) % size]`, `specialPeriod = 3`: a fixed cycle. `Tables.h:27` `enum class DayRule { None, IgnoreWorm, FastEscape, NoSunlightBites, CheapFishWorthless, ShortDay }`. The rng is `rng_` (xorshift, `Private/Game.cpp`, `RandomUnit`), seeded in `Game::NewRun`.
- Run start: `Private/Game.cpp:12-50` `Game::NewRun` sets `money = pace.initialMoney` (0, `Tables.cpp:145`), one unopened Small Pack (`state_.unopenedPacks.push_back(pack::Small)`), and emits `PhaseChanged` to `Phase::Prep` of day 1. `Game::NextDay` (`Days.cpp:185`) needs `Phase::Shop` and no open pack, increments the day and goes to `Prep`.
- Shop numbers now (`Tables.cpp`): charm cards cost 4-8 coins by an interim map, three offers, all 20 offered from shop 1; reroll 2 (+1); packs 2 and 4; chum 3 (+1 per level); upgrades New Rod 5/8/12/17, Sharpen Rod 2/4/6/9 per rod, Deeper Line 6/10/15/21/28, Bigger Shelf 10/18; payout base 4, over at most 4, interest at most 3.
- The design, single source for the rows: `Docs/design/modifiers-v2.data.json`, version 2.1. Keys: `combineOrder`, `rarities` (weight 60/30/10, price 4/6/8, weightless 6/9/12), `weightless`, `shop`, `primitives` (8 triggers, 15 conditions, 15 effects, 17 counts, 7 rule flags, each with its meaning), `primitiveChanges`, `charms` (50 rows: `id`, `nameEn`, `class`, `rarity`, `price`, `ruleEn` with `{n}` for a live value, `kind`, `rows` = trigger + conditions + effect, optional counter), `builds` (10 lines of play). The reasoning is in `Docs/design/modifiers-v2-proposal.md` (916 lines; read sections 3, 4, 12 and 13).
- The orchestrator's fact sheet for B6 (main checkout): `F:\UE\EpicJam\Docs\briefs\reports\core-facts-points.md`, line 100 (block B6), section 5 (open questions) and section 6 (what the core lacks). The open questions that touch B6 are answered in section 3 below.
- View consumers of the old names (module `Catch`, not your files; the newest view is on `track/ui`): charm enum names in `CatchHud.cpp:411,418`, `CatchLookSound.cpp:112-113`, `CatchTackleView.cpp:783`; `Mods().noEscape` in `CatchHud.cpp:390`, `CatchTackleView.cpp:333,935`; charm texts keyed by the core charm name in `CatchUiText.cpp:7-33` and `CatchUiPanels.cpp:219-227`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Rows and evaluator

1. The JSON file is the single source of the rows. Recommended way: a small generator script (`Catch/CoreTests/gen_charm_rows.py`, Python 3, standard library only) reads the JSON and writes a C++ table file (for example `Catch/Source/CatchCore/Private/CharmRows.inc`) that is committed; `--check` exits 1 when the file on disk differs from a fresh generation. The tests must not need Python. Hand-written rows are acceptable only if you show a check that they match the JSON.
2. Only English goes into game source: `nameEn`, `ruleEn`. Never emit `nameRu` or `ruleRu` into C++ (the check gate rejects non-English text in the game).
3. One evaluator executes every row: trigger, conditions (all must hold), effect, optional `per` count, `perStep`, `minus`, `cap`. No per-charm code paths except the rule flags that already exist in the core (`auto_reel`, `no_escape`, `cluster`, `bait_knife`, `giant_size`), which the evaluator switches on per focus.
4. `cap` limits the resulting value (kg, percent, coins, slots), as the rule texts say (Slow Hands "up to +300%", Same Again "up to +20 kg").
5. Follow the data rows where the v2 core kept a v1 reading: Food Chain needs a chain of 2 or more; Big Haul is +10 % per other fish of the same pull (remove the 1 s window); Auto-Reel needs 3 or more rods; Barbed Hooks needs rod level 3 or more (per rod); Cluster Hook calls fish only for a shoal fish of a chummed kind; Low Tide lifts chummed kinds only (per kind); Thrifty Hook leaves the game. Chum Bucket and Dusk Frenzy do exactly what their rows say and nothing else.
6. Counters live on the shelf item (`ShelfItem.counter`) and survive days. Selling the charm loses the counter.
7. Day end order: payout as now (interest reads the coins held before the payout); then `on_day_end` rows that add coins, in shelf order; then growth rows (they read the coins after all coins are added); then `ground_add`.
8. Storm Glass: the extra coins for the Gold Sunfish are read by the gold code as a rule value. The Gold Sunfish still skips combine steps 3-9.
9. `seconds_hooked` is the time the landed fish itself hung (the timer restarts when a fish eats the fish on the hook, as now).
10. Dynamic shelf slots (Rod Stand, Sea Chest) can fall below the number of charms held. Then the charms stay and a purchase is refused until a slot is free. Weightless charms take no slot, as now.
11. Shop: three charm offers drawn by rarity weight 60/30/10 without repeats and without charms already on the shelf; prices 4/6/8 and weightless 6/9/12; every charm is offered from the first shop (no unlock days). Unlocks between runs are a later task: keep `Unlock` in the data, unused.
12. Keep source compatibility where it is cheap: keep the `charm::` constants of the kinds that stay (append new kinds after them, keep `Count` last), keep `Mods` fields the view reads (`noEscape` may become "true if any rod has it" plus a per-rod query). List every break in the report; the view is adapted later on `track/ui`.
13. Texts for the view come from the core: add a query that returns a charm's English name, class, rarity and its rule text with `{n}` replaced by the live value for the current state (shelf item or shop offer). This replaces the name-keyed text maps of the view later.
14. Keep the existing rounding scheme of v2 (integer tenths of kg, halves up). No floating-point drift between platforms: the same seed and commands must give the same events.

Supplement 2.2: three new rows and two new primitives (owner's words: chance-based "joker" analogues, a multiplier at a chain link, purchases with a plus and a minus). Add them to the JSON as version 2.2 in the same format, with a `versionNote`.

15. New condition `chance {value}`: after all other conditions of the row hold, one roll of the core rng; the row fires with probability `value`.
16. New trigger `on_lucky`: fires once each time a row with a `chance` condition fires.
17. `timer` gets `what = hang`: seconds added to the time a fish hangs before it escapes (may be negative; the result never goes under 1 s).
18. `lucky_bite` — Lucky Bite, class fish, uncommon, 6. Rule text: "When a fish eats the fish on your hook: 1 chance in 4 that it weighs x3." Row: `on_chain_link`, condition `chance 0.25`, effect: a x3 weight multiplier stored on that hook; it applies at combine step 6 when the fish on the hook lands, stacks with later lucky links of the same hook, and is lost if the fish escapes. Emit an event for the view when it fires (rod, hook, charm, value).
19. `fish_tale` — Fish Tale, class fish, uncommon, 6. Rule text: "Grows +15% each time a lucky chance comes true. All fish +{n}% weight." Rows: `on_lucky` -> `counter_add` (step 15, no cap); `on_land` -> `percent` value 1 per `counter`.
20. `thin_line` — Thin Line, class rod, uncommon, 6. Rule text: "Larger fish strike twice as often. Hooked fish escape 2 s sooner." Rows: passive `rate eat x2`; passive `timer hang -2`.

Strike chance as a stat

21. Lower `pace.strikeChance` so that, with no modifier, a level 1 rod and an eater in reach, about 40 % of hooked fish that hang until their escape get struck (now about 95 %). Find the value by simulation and report the measured share; my estimate is about 0.055 per check. Do not change the check timing, the reach or the ladder. The chance grows with what the player builds: hang time (rod level, Barbed Hooks), bait `hunterMultiplier`, Fresh Scent, Thin Line.
22. Add a query for the view: the chance (0..1) that a larger fish strikes the fish on a given hook before it escapes, from the current factors and the checks left; 0 when no fish kind in the open zones may eat it. It is an estimate for display only.

Run flow

23. The run opens in the shop. `NewRun` ends in `Phase::Shop` before day 1 with `pace.initialMoney = 5`; the shop there has the normal offers (charms, packs, upgrades, chum, fish cards, worm tin); chum bought there is active on day 1; no payout and no interest happen before day 1; `NextDay()` leads to `Prep` of day 1. The free starting Small Pack stays (the player may open it in that shop or in preparation, as the core already allows). Put the switch into the tables (for example `pace.startInShop`, default true) so old test fixtures can turn it off. You choose how `state.day` reads in that shop (0, or 1 with a flag); document it.
24. Rule days are random per run. At `NewRun`, draw the order of rule days from the run seed and store it; `RuleOf(day)` stays a pure lookup, so the forecast two days ahead keeps working. Guard: the first and second rule day draw only from the light rules (IgnoreWorm, FastEscape, ShortDay); from the third rule day on, every rule may come; no rule repeats until all rules of its pool were used.
25. New rule `DayRule::OnlySmallFish` (append at the end of the enum): "Fish of 10 kg or more give no points." Exact: a final weight (combine step 6) of 10 kg or more gives 0 points; the Gold Sunfish is exempt; threshold in the `dayRules` table. It is a heavy rule (decision 24).
26. `Storm Glass` (`no_day_rule`) switches the rule of the day off, whichever rule it is.

Measurement, no tuning

27. Do not tune norms, prices or payouts. After the work, extend `core_runs.cpp` with one scripted policy per line of play from `builds` (buy its charms and its ground when offered and affordable, otherwise play like the person-like policy) and print, for the Soft level, days 1-15, 200 runs: alive % and median points / norm per policy, plus the share of hooked fish that get struck. The orchestrator uses the table to retune the norm row.

## 4. Work by blocks (keep the tests green after each block)

- A. Data model and generator: charm rows, rarity, counters, texts. The 20 old kinds map to their 2.1 rows; Thrifty Hook goes.
- B. Evaluator for `passive` and `on_land` rows: conditions, counts, `flat`, `percent`, `mult` (weight and award), `chain_step`, `eat_reach`, `count`, `rate`, `timer`, `rule`, `shop`, `slots`. `CompileMods` becomes row-driven; flags per rod and per kind.
- C. Event triggers and counters: `on_pull`, `on_escape`, `on_chain_link`, `on_day_end`, `on_shop_enter`, `on_purchase`; `coins`, `ground_add`, `counter_add`, `counter_reset`.
- D. Shop: rarity draw, prices, weightless prices, dynamic slots, the text query.
- E. Supplement 2.2 (decisions 15-20) and the strike chance (21-22).
- F. Run flow (23-26).
- G. Tests, the fuzz run, the build policies and the measurement table (27).
- H. Report.

If you run short of session, stop at a green state after a finished block and report exactly what is left. A finished, tested A-D is worth more than a half-wired A-G.

## 5. Done means

1. `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1` passes. New tests: at least one case per trigger, per condition, per effect kind and per count; at least one case per charm that checks the example of its rule text (53 charms); day end order; dynamic slots; rarity draw shares over many draws; the three supplement rows; the strike share (decision 21) within a stated tolerance; the rule order (guard, no repeats, same seed gives the same order, forecast matches the day); the new rule (9.9 kg counts, 10.0 kg gives 0, Gold Sunfish exempt); the shop before day 1 (5 coins, purchase works, chum active on day 1, `NextDay` leads to day 1 preparation).
2. The random-command fuzz (10,000 commands) runs with charms on the shelf and holds the old invariants plus: counters within caps, shelf never over its slots after a purchase, no negative coins.
3. Determinism: the same seed and the same commands give the same event list twice.
4. `python -X utf8 Catch/CoreTests/gen_charm_rows.py --check` exits 0 (if you took the generator).
5. `powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 15` prints the table of decision 27.
6. `python -X utf8 Tools/check_all.py` (without `--ue`) shows 0 fail apart from the git-clean check.
7. No file outside the list in section 7 is changed.

## 6. Verification commands (run from `F:\UE\EpicJam_core`)

```
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 15
python -X utf8 Tools/check_all.py
git status --short
```

Do not run UnrealBuildTool, `Tools\build-editor.ps1`, the editor or the packager: another executor builds the Unreal project in `F:\UE\EpicJam_ui` at the same time, and two builds collide. Your proof is the engine-free test build.

## 7. Boundaries

- You may change: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/modifiers-v2.data.json` (only to add version 2.2: the three rows, the two primitives, `timer.what = hang`), and your report. Nothing else.
- Do not touch `Catch/Source/Catch/**` (the view), `Catch/Content/**`, `Tools/**`, `Docs/design/core-interface.md` (the orchestrator edits it in fixed windows: write the interface changes into your report instead), `CLAUDE.md`, `AGENTS.md`, the journal, the decisions.
- Do not invent rules, numbers or modifiers beyond the JSON and this brief. If a row cannot be executed as written, do not guess: implement the nearest reading only if the rule text settles it, mark it in the report, and otherwise leave the charm out of the shop pool with a note.
- Nothing from the browser prototype goes into the game: no code, no text copied from its source. The JSON and the proposal are design documents and are the allowed source.
- No downloads, no new dependencies, no generators. No commits, no pushes. Do not delete files.
- Parasites, squids as a line of play, treble hooks and unlock tasks between runs are owner ideas without a decision. They are out of scope.

## 8. Report

Write `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core-charms.md` in English, and hand back a short summary. Contents:

1. What is done per block, what is not.
2. Test and fuzz results (counts), the determinism check, the generator check.
3. The measured strike share before and after, and the value of `strikeChance`.
4. The measurement table of decision 27, as printed.
5. Every reading you took where the data was unclear, each with the table value or file that holds it.
6. The interface changes for `core-interface.md`: new and changed commands, state fields, queries and events, exact names.
7. Every break for the view module: old name -> new name or query, with the view file and line from section 2 where you know it.
8. Charms left out of the shop pool, if any, and why.
9. Questions for the orchestrator.

## 9. Chain of command

You report to the orchestrator, not to the owner. Write open questions into the report; technical ones you answer yourself and list as revocable readings. Director-level questions (a new rule, a number the owner must feel) go into section 9 of the report and you continue with the parts that do not depend on them.
