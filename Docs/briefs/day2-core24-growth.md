# Brief: core 2.4 - fast growth (a free charm after each day), offers that fit the player, a fair rule day, balance pass 2

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 13:00 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_core`, branch `track/core`, base `d6a4ceb` (core 2.3, clean). This brief lives only in the main checkout.

## 1. Goal

Make the player grow fast and the numbers grow big, without making the game easy. After each passed day the player takes one charm out of three for free. Offers only hold charms that can work for the player. The targets rise much more steeply, fitted by the bots. One rule day stops being a trap.

Why it matters. The owner (game director) played build 7 on 13.11 and said, by voice:

- Pace: «это не безумный темп, это вдумчивый, медленный такой геймплей потихоньку, полигоньку развивать».
- Cards: «дело причем не столько в недостатке очков, сколько, наверное, в недостатке нужных карточек […] я хотел бы […] хищников увеличивать […] А вместо этого мне попадаются […] карточки, которые […] увеличивают вес, если […] зависимость высокой глубины. А у меня вообще нет очков» (he had no depth open).
- Big points: «Когда я говорю тонны рыбы я не имею в виду буквально, я скорее про большое количество очков».
- The trap: «я проиграл на событии, которое говорит, что рыба, допустим, свыше 10 килограммов не дает очков».
- Coins: «он должен получать много монет, но не бесконечное количество».
- The frame of the game: the game must not be easy; the fee presses on the player every day; a failed fee ends the run.
- The go (13.11, about 12:55), to the orchestrator's question "Start the core block: a free charm per day, steep targets, a fair trap?": «В принципе, запускай следующий блок.» This also lifts his earlier «пока баланс не меняй».

Decisions on record: `fast-growth-approved`, `shop-offers-must-fit-the-player`, `tons-of-fish-means-big-points`, `day-ends-at-target`, `start-gift-instead-of-first-shop` in `Docs/decisions/game-frame.answers.json` (main checkout).

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_core\`.

- Core 2.3 report with the interface delta and the measurements: `Docs/briefs/reports/day2-core23.md`. Read it first. Balance pass 1: brief `F:\UE\EpicJam\Docs\briefs\day2-balance-pass1.md`, its report under `Docs/briefs/reports/` in this worktree.
- Start gift (model for the reward): `State::giftPending`, `GiftOffers()`, `GiftReady()`, `TakeGift(index)`, events `GiftOffered`, `GiftTaken`; `StartDay` returns `Locked` while the gift is pending.
- The day ends when the points reach the target (`Game::TargetMetNow`, `ReachTarget` in `Catch/Source/CatchCore/Private/Days.cpp`); the time bonus is 1 coin per 20 % of the day saved, at most 4 (`Prices.cpp`).
- Shop charm offers: `Game::RollCharms` in `Catch/Source/CatchCore/Private/Shop.cpp` (about line 56) rolls three charms by rarity (60/30/10) and does not read the `needs` field. 47 of the 53 charms carry a non-empty `needs` in `Docs/design/modifiers-v2.data.json`.
- Charms can be sold from the shelf for half the price. The shelf has five slots; some charms change the slot count; some are weightless.
- Day rules: `OnlySmallFish` ("Fish of 10 kg or more give no points.") and `CheapFishWorthless` ("Fish lighter than 10 kg give no points.") both read the FINAL weight of the fish (table weight plus every bonus). Nearly every charm adds weight, so a grown build scores zero on `OnlySmallFish`.
- Measured on core 2.3 (200 runs x 15 days, person-like bot): a passed day lasts 7-16 s on days 1-9 (the day has 50 s); the bot owns 1, 2, 4 charms after days 4, 7, 10; survival after days 3/6/9/12/15 is about 99/95/62/13/1 %, with the big drops on the rule days 9 and 12; targets are 20 on day 1, 280 on day 9, 2400 on day 15; the Gold Sunfish is landed on 50 % of the rule days; the gift Deeper Line is weak (68 % alive after day 6 against 95 %).
- Data and generators: `Docs/design/economy-v2.data.json`, `Docs/design/modifiers-v2.data.json` (it also holds the Russian text of every charm), `Catch/CoreTests/gen_charm_rows.py`, tests `Catch/CoreTests/core_tests.cpp`, bot runs `Catch/CoreTests/core_runs.cpp`, `BuildTargets.inc`.

## 3. Decisions (director decisions are fixed; technical ones are revocable - say so in the report if you change one)

Director decisions (fixed)

D1. After each passed day the player takes ONE charm out of THREE for free, or skips. The start gift before day 1 stays as it is.
D2. The reward and the shop offer only charms that can work for the player.
D3. The rule "10 kg or more" reads the fish KIND (its table weight), not the final weight. The rule "lighter than 10 kg" stays on the final weight (growth passes it).
D4. The targets rise much more steeply than now; the player sees big numbers. Coins per day stay bounded.
D5. The game stays hard: the fee presses every day.

Technical decisions (revocable)

1. Reward flow. Mirror the gift: after the settlement of a passed day the core sets `State::rewardPending`, emits `RewardOffered`, and gives `RewardOffers()` (three charm ids with name, text, rarity), `TakeReward(index)`, `SkipReward()`, event `RewardTaken` (index or skip). `StartDay` returns `Locked` while a reward is pending. Shop purchases and selling work while it is pending (the player may have to sell a charm to make room). `TakeReward` with a full shelf returns the same result code that a shop purchase gives for a full shelf. No reward after a failed day (the run is over). The Gold Sunfish prize and the day rules do not change the reward.
2. Reward roll. Three different charms, not owned, from the same rarity roll as the shop, through the same fit filter. The reward does not consume or change the shop offers of that shop; the shop must not show a charm that sits in the pending reward.
3. Fit filter. Write one predicate per `needs` tag: "this charm has an effect with what the player owns now" (rods, hooks, open zones, baits in the box, fish kinds reachable with those baits and zones, charms on the shelf). An offer list is filled in this order: charms that fit now; then charms that fit after ONE purchase that this shop sells now (a rod, a depth level, a bait pack that can hold the needed bait); then anything. The third group must be rare; measure how often it is used. Fix wrong `needs` tags while you are there (Tide Pool carries `margin` and now counts saved time).
4. The rule day. `OnlySmallFish` reads the table weight of the kind. Give it a text that says so in at most 60 characters (proposal: "Only small kinds score. Kinds of 10 kg or more give 0."). Add `DayRuleScores(kind)` or an equal query so the view can mark the kinds that score today. Check with the bots that every build has a kind that scores on that day within its open zones and a bait for it in the shop before the day; if not, the rule day offers that bait.
5. Gift card text: Sharp Rod reads "Rod 1 starts at level 3 with two hooks."
6. Balance pass 2, in this order. Every number you move is listed in the report with before and after.
   - a. Growth: with the reward on, the person-like bot should own about 1 charm after day 1, 3 after day 3, and a full shelf by day 5-6. If the reward alone does not give that, look at the bot's shop policy before you touch prices.
   - b. Targets: refit the Soft target row so that a PASSED ordinary day of the person-like bot has a median length of 30-40 s of the 50 s on every day from 1 to 15. This replaces the old soft target "points against the target on days 1-3".
   - c. Big numbers: the target should multiply by about 1.5-1.8 per day on average, so day 9 lands in the thousands and day 15 in the tens or hundreds of thousands. The curve must come from what the builds can really score. If the fitted curve is flatter, raise the multiplying charms (at most 15 charm values, as in pass 1) and refit; do not pad the targets beyond what the builds reach.
   - d. Survival of the person-like bot: keep the targets of balance pass 1 as the data and its report state them, and remove the cliffs: no single day may cost more than about 15 points of survival. Rule days stay harder than ordinary days, not lethal by themselves.
   - e. All ten bot builds stay inside the strength band of pass 1; no build may pass day 15 in more than about half of its runs.
   - f. The gift Deeper Line comes within 10 points of the other two gifts after day 6 (for example the card also gives a bait that catches in the new zone; if you change what the card gives, change its text).
   - g. The Gold Sunfish is landed on at least 80 % of the PASSED rule days of the person-like bot (it must arrive early enough in a day that ends at the target).
   - h. Coins: the payout per day stays bounded (base 4, time at most 4, interest at most 3, plus charm rows). Report the highest coins per day any build reaches.
7. Bots: add a reward policy per build (take the charm that fits the build best; with a full shelf, sell the weakest charm when the new one is better; else skip). Keep the "before" mode: a table switch that turns the reward and the filter off, so the report shows before and after from the same code.
8. Version: ApiVersion 5, data version 2.5.

## 4. Work by blocks (in this order; extend the report after each block)

1. Reward flow, events, state, tests (decisions 1-2).
2. Fit filter for the reward and the shop, the `needs` audit, tests (decision 3).
3. The rule day by kind, the query for the view, text, tests; the Sharp Rod text (decisions 4-5).
4. Bot policies and the "before" switch (decision 7). Measure: with the reward on and the old targets.
5. Balance pass 2 (decision 6), then the final tables.
6. Russian text in `modifiers-v2.data.json` for every charm text you changed; regenerate; the interface delta for the view in the report.

Blocks 1-3 are rules and are needed by the view first: finish and report them before the long bot loops of blocks 4-5.

## 5. Done means

- `python -X utf8 Tools/check_all.py` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/CoreTests/gen_charm_rows.py --check` passes.
- New tests pass: reward flow (offer, take, skip, full shelf, locked start), the filter never offers a charm from the third group while three fitting ones exist, the rule day by kind, determinism with the reward on.
- The report holds, for the person-like bot and for each of the ten builds, 200 runs x 15 days, before and after: survival by day, median length of a passed day by day, charms owned by day, the target row, the median points of a full day against the target, coins per day (median and highest), the Gold Sunfish share, the three gifts, how often the third filter group was used, survival on each rule day by rule.
- The report lists every changed number, every changed charm text (English and Russian), every `needs` tag changed, and the interface delta for the view (section "For the view").

## 6. Verification commands (run from `F:\UE\EpicJam_core`)

```
python -X utf8 Tools/check_all.py
python -X utf8 Catch/CoreTests/gen_charm_rows.py --check
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 200 -Days 15
powershell -NoProfile -Command "& 'F:\UE_5.9\Engine\Build\BatchFiles\Build.bat' CatchEditor Win64 Development '-Project=F:\UE\EpicJam_core\Catch\Catch.uproject' -Module=CatchCore -WaitMutex"
```

Never run two copies of `core-test.ps1` at once in this worktree. A 200 x 15 balance run takes about 11 minutes; use fewer runs while you search and the full run for the tables. Another executor builds Unreal in `F:\UE\EpicJam_ui` at the same time: builds wait for each other through `-WaitMutex`; do not kill a process you did not start.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/economy-v2.data.json`, `Docs/design/modifiers-v2.data.json`, and your report file.
- Do not change `Catch/Source/Catch/**` (the view), `Docs/design/core-interface.md`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not add a rule, a charm, a fish, a bait or a screen that this brief does not name. A question for the game director goes into the report; go on with what needs no decision.
- Do not delete or hide old work; old modes stay as table switches. No commits, no pushes.
- All player text is English; Russian only inside the data file fields that already hold Russian. No Cyrillic in source files.

## 8. Report

File: `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core24.md`, in English. Create it right after reading, with one status line per block, and extend it after each block (the desktop app stopped twice today; the report is the state that survives). Sections: what changed by block, tables (section 5), changed numbers, changed texts, `needs` audit, "For the view" (interface delta), checks with their output lines, not checked, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
