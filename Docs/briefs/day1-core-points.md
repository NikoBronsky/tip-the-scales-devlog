# Brief: rules core v2 — points by weight, coins, the day's norm, the food ladder, chum

Executor: Opus (high). Worktree: `F:\UE\EpicJam_core` (branch `track/core`). Written by the orchestrator on 12.11.2026.
You report to the orchestrator, not to the owner. Write code, comments, the report and all player-facing text in English.

## 1. Goal

Convert the engine-free rules core (`CatchCore`) from "fish have a dollar price, money pays a fee" to the owner's new frame: the catch of a day is scored in points by weight against the day's norm, coins are the only money, hooked fish are eaten along a food ladder, and chum raises the count of a fish kind. Everything else in the game (HUD, sound, tutorial, balance) waits for this.

Owner decisions (`Docs/decisions/game-frame.answers.json`), quoted:
- `score-in-points-by-weight`: «у рыбы будет вес, который дает больше очков. То есть мы не напрямую цену увеличим, мы увеличиваем вес рыбы.» Confirmed reading: points of a fish = weight x the kind's points per kilogram.
- `scales-catch-against-norm`: «слева будет вес в очках, а справа будет вес, который нужно достичь, чтобы пройти уровень.»
- `food-ladder-next-size`: «рыбка 2 должна ловиться на рыбку 1, рыбка 3 должна ловиться на рыбку 2 … Рыбки не должны все ловиться на кильку. Это должен быть какой-то модификатор.»
- `money-like-balatro`, `no-day-locks-money-tight`, `chum-raises-fish-count`, `prepare-the-ground`: coins come as a payout for a passed day; money is tight; nothing is locked by day; chum raises how many fish of a kind swim.

## 2. What exists now

Read first: `Docs/briefs/reports/core-facts-points.md` (a verified fact sheet of the core as it is, with file:line references, a gap table and a block plan). It is the map for this task. Then `Docs/design/core-interface.md` (section 11 announces the v2 surface), `Docs/design/modifiers-v2.data.json` (version 2.1: `combineOrder`, `fishWeights`, `sizes`, `awards`, `ladder`, `dayRules`) and `Docs/design/economy-v2.data.json` (all economy numbers).

Facts that shape the work:
- One currency today: shipping adds each price to `money`; `PayFee` subtracts the fee (Days.cpp:108-178).
- The price is a fixed chain of multipliers in double maths with one rounding (Prices.cpp:34-81); Big Haul is added after landing in a 1 s window (:103-123).
- Eating needs `hunter`, a higher `tier`, the home band +-30, reach 504.9, chance 0.28 x factor, a check every 0.6 s (Rods.cpp:378-437).
- The 20 charms are hard-coded branches (`CompileMods`, Shop.cpp:20-46, and `mods_.active[]` checks).
- The Unreal module reads `FishData::tier` as the visual size of a fish (CatchSeaView.cpp:195-196).
- The test fixture `Quiet()` switches eating off with `hunter = false` (core_tests.cpp:16-23).

## 3. Decisions of the orchestrator (technical, revocable; say so in the report if you change one)

1. Scope. This task is blocks B1-B5 and B7 of the fact sheet, plus tests, the runner and the interface note. The 50 charm rows (B6) are NOT in this task: the owner has not judged the charm list yet. Build so that B6 can plug in: one accumulator, one eat test, one place for coin effects.
2. The 20 charms of v1 keep working with the least change: a price multiplier becomes a weight multiplier (step 6), a price addition becomes kilograms (step 4), Big Haul's +50% becomes +50 percent (step 5, same 1 s window), Food Chain sets the chain step to 1.0. Fresh Scent and Siren Pearl keep their eat factor. List each mapping in the report. Their shop prices come from `shop.interimCharmPrices`.
3. Scoring order (data 2.1 `combineOrder`): (1) Gold Sunfish = half of the day's norm, halves up, no weight; (2) a Bait Knife Sprat = 0 points; (3) weight = table weight in kg; (4) + flat sum in kg; (5) x (1 + percent sum / 100); (6) x multipliers x bait multiplier (Golden Grub 2) x chain factor (1 + chain x chain step, step 0.5) = final weight; (7) day rule 4: final weight under 10 kg gives 0 points; (8) award = card level of the kind x award multipliers; (9) points = final weight x award, halves up.
4. Rounding. The final weight is rounded half up to 0.1 kg after step 6 and stored as integer tenths. Rule 4 and the points use that rounded weight, so the player is paid for the weight shown. Avoid x.4999 errors: use integers or a documented epsilon.
5. Coins. `State::money` is the coin count. Fish never add to it. The day total is in points (`catchTotal`, `bucketTotal`). Shipping only counts the catch (it changes no money).
6. Settlement. `PayFee` keeps its name and gets the new meaning (add the alias `SettleDay`): points >= norm -> coins += base + over + interest (numbers and formulas in `economy-v2.data.json` `payout`; interest uses the coins held before the payout), then Shop; else RunOver. New event `DaySettled {day, norm, points, base, over, interest, coins after}`. `RunOver` carries norm, points and coins.
7. Norm. `NormOf(day)`; `FeeOf` stays as an alias. Curves from `economy-v2.data.json` `norm` (three levels, table then growth, the existing two-significant-digits rule).
8. Gold Sunfish gives its points and `goldSunfish.coinsAtLanding` coins when it lands.
9. Shop prices, offers and gates come from `economy-v2.data.json` (`shop`, `gates`). Everything is offered from the first shop. Fish card: award + 1.
10. Fish data: add `weight` (kg) and `size` (1-6, Gold 0) from data 2.1 `fishWeights` and `sizes`. Do NOT reuse or change `tier`: the Unreal module draws by it. `price` stays in the struct for now.
11. Food ladder. A hooked fish of size N is eaten only by a free fish of size N+1 up to N+reach; reach is 1. Every kind except the Gold Sunfish may eat; the Gold Sunfish is never eaten. All other conditions stay (open zone, hook depth inside the eater's band +-30, reach along x, check every 0.6 s, chance 0.28 x eat factor, nothing eats before `gates.eatFromDay`). The `hunter` flag keeps only the wide band, the arrival day and the hover note. Put the reach behind one function (`EatReach(eater, hook)`, returns 1 now) so charm rows can widen it later.
12. Live Sprat bait is a size-1 lure. A free fish takes a Live Sprat hook as a normal bite when it passes the ladder test against size 1 (so size 2 with reach 1). Its accept list goes away. It may be offered in packs when a kind that takes it swims today or tomorrow in an open zone (the existing rule for list baits). `FishInfo` lists Live Sprat for the kinds that take it.
13. Chum. `chumLevels` per kind, 0 to `chum.maxLevel`. Command `BuyChum(offer)`; the shop shows `chum.offers` kinds drawn from the kinds whose effective zone is open (never the Gold Sunfish, never a kind at the cap). The wanted count of a kind is multiplied by (1 + growth x level), from the next day: copy the levels to an active set when the day is prepared.
14. Day rules: rule 4 reads the weight (text from data 2.1 `dayRules`, the `new` field of that rule). Rule 3 stays as it is in v1: the owner has not decided it.
15. Compatibility. Do NOT edit `Catch/Source/Catch/**`. The Unreal module must compile unchanged against the new core: keep every name it uses (the fact sheet lists them in 2.i): `FeeOf`, `PayFee`, `Mods()`, the charm constants, `PullEstimate::money`, every `price` field. The `price` fields carry points now; add `points` and `weightTenths` (or a name you prefer) beside them. Old events that no longer happen (`FeePaid`) stay in the enum and are not emitted. `ApiVersion()` returns 2.
16. Tests. `Quiet()` gets a real switch (strike chance 0 or a table flag). Rewrite the price, fee, shop and gold tests to the new numbers; do not delete a test area.

## 4. Work, in this order

- B1 Data: `Tables.h/.cpp`: weight, size, norm tables, payout, coin prices, chum data, gates.
- B2 Economy split: points total, shipping without money, `PayFee`/`SettleDay`, `DaySettled`, `RunOver`, `NormOf`, Gold coins. Files: Days.cpp, Game.h, Game.cpp, Prices.cpp.
- B3 Weight and points: the accumulator and steps 1-9; `LandedFish`, `FishLanded`, `ChainLink`, `PullPreview`, `FishDetails` carry points and weight. Files: Prices.cpp, Game.h.
- B4 Food ladder and Live Sprat. Files: Rods.cpp, Fish.cpp, Shop.cpp; `FishDetails` gains size, award, the kinds it eats on a hook and the kinds that eat it (reach 1, open zones).
- B5 Chum. Files: Game.h, Fish.cpp, Shop.cpp.
- B7 Day rule 4 by weight.
- Runner: `core_runs.cpp` compiles and reports per day: alive %, median points against the norm, median coins. The three policies buy with coins.
- Interface note: fill section 11 of `Docs/design/core-interface.md` with the exact final names (this one file outside the code is yours to edit).

## 5. Done means

1. `Tools/core-test.ps1` passes with zero warnings (it builds with /W4 /WX).
2. Worked examples are tests: Mackerel that ate a Sprat = 7.5 kg, 8 points; Sea Bass with chain 2 = 24; Tuna with chain 3 = 75; a Mackerel that ate five Sprats = 17.5 kg, 18 points; Sea Bass card level 3, chain 1 = 54; Gold Sunfish on day 1 Normal = 8 points and 3 coins; rule 4: Mackerel chain 1 (7.5 kg) = 0, Golden Grub Mackerel (10.0 kg) = 10.
3. Payout tests: 55 points against 15 pays 4 + 4; 60 against 50 pays 4 + 0; interest 0 at 4 coins, 1 at 5, 3 at 15 and at 40; a day under the norm ends the run and keeps the coins.
4. Ladder tests: Sprat on the hook: Mackerel eats, Sea Bass and Tuna do not; Mackerel on the hook: Sea Bass eats, Mackerel does not; Lanternfish (size 2) eats a Sprat; nothing eats the Gold Sunfish and it eats nothing; nothing eats on day 1; each chain of the zone table in `modifiers-v2-proposal.md` 12.2 is possible and no other pair is.
5. Live Sprat tests: Mackerel takes it, Sea Bass and Giant Squid do not.
6. Chum tests: level 2 of a kind doubles its wanted count from the next day, not the same day; the cap holds; the price follows the level.
7. The 10,000 random commands test passes with the invariants restated (coins never negative; the sum of points of the catch list equals `catchTotal`).
8. `Catch/Source/Catch` is unchanged and compiles against the core. Try the game target (`Catch Win64 Development`) in this worktree; if the build tool refuses because an editor is open elsewhere, say so and the orchestrator compiles at the merge.
9. `-RunBalance -Runs 200 -Days 9` runs and its table is in the report.

## 6. Verification commands

```powershell
powershell -ExecutionPolicy Bypass -File Tools/core-test.ps1
powershell -ExecutionPolicy Bypass -File Tools/core-test.ps1 -RunBalance -Runs 200 -Days 9
python -X utf8 Tools/check_all.py
```

Shell note: the Bash tool may fail in this session with "unexpected EOF while looking for matching quote". Use the PowerShell tool. To run a `.sh` file: `& "C:\Program Files\Git\bin\bash.exe" <file>`.

## 7. Boundaries

- Work only in `F:\UE\EpicJam_core`. Never touch `F:\UE\EpicJam` (the sea-look executor owns the editor seat there) or `F:\UE\EpicJam_ui`.
- Files you may change: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/core-interface.md` (section 11 only), your report. Nothing else.
- No commits, no branch switches, no pushes. Delete no file.
- Do not open the Unreal editor.
- Invent no rule and no number. Rules come from this brief and data 2.1; numbers from the two data files. Where they are silent or disagree with `rules-reference.md` / `content-list.md`, take the nearest reading, keep it in `Tables`, and list it in the report. The data files win over the two older documents.
- Nothing from the browser probes goes into the game: no code. The numbers are restated in `economy-v2.data.json`; use that file.
- Player-facing text stays English, age 13+, no logos.

## 8. Report

File: `Docs/briefs/reports/day1-core-points.md`, English, at most 120 lines:
1. What changed per block, with files.
2. The final surface: commands, events, state and query fields that are new or changed meaning (this is what the HUD executor will read).
3. The mapping of the 20 v1 charms.
4. Test counts by area and the balance table.
5. Readings you had to take, each with the place in `Tables` where it lives.
6. What is not done, and what B6 (charm rows) will need from this code.
Hand back after B1-B3 pass their tests if more than 100 minutes have gone by; otherwise hand back when all is done.
