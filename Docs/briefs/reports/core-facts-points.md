# Core facts for the points conversion (worktree F:\UE\EpicJam_core, f9d442b)

Read-only analysis. Paths are relative to the repo root. "Not read yet": progression-proposal.md, modifiers-v2-proposal.md sections 1-8 except 3.5-3.6 and 8, builds detail, TestMovement18 body, CatchHud.cpp outside the lines cited.

## 1. Map of the core
- `Public/CatchCore/Tables.h` (213): ids (fish/bait/charm/upgrade/pack enums, :21-25), FishData, BaitData, CharmData (value/secondary/tertiary, no rarity), FeeData, PaceData (all rule numbers), ShopData, EdgeFlags, Tables.
- `Private/Tables.cpp` (113): MakeDefaultTables: zones, 15 fish, 12 baits, 4 upgrades, 2 packs, 20 charms, 3 fee rows, 5 day rules, x conversion.
- `Public/CatchCore/Game.h` (238): Result, Phase, RodState, MoneyReason, EventType, Event, FishEntity, HookData, RodData, LandedFish, ShelfItem, Offer, ShopState, State, FishDetails, PullEstimate, Mods, class Game.
- `Private/Game.cpp` (156): NewRun, RNG, ChangePhase, ChangeMoney, SetRodBait, SetGoalDepth, StartDay, SetFeeLevel, Advance/Step.
- `Private/Days.cpp` (209): day rules, day length, EndDay, shipping, Summary, SkipShipping, PayFee, NextDay, PrepareDay (Gold Sunfish entry).
- `Private/Fish.cpp` (329): zones, Low Tide shift, bands, wanted counts, refill, schools, swimming, BaitMatches.
- `Private/Rods.cpp` (580): rod timing, casting, bait loading, scan/bite, hang/escape, hunter check + EatFish, Cluster Hook, pull, LandRod.
- `Private/Prices.cpp` (214): FeeOf, PriceFish, LandFish, Big Haul, flights, ArriveFish, PullPreview, fish info card.
- `Private/Shop.cpp` (317): unlocks, CompileMods (charm -> Mods switch), upgrade prices, charm roll, shop refresh, all buy/sell/pack commands.
- `Private/CoreMath.h` (25): CoreMoney (floor), CoreRound (halves up), CoreMoneyAdd (saturating int64).
- `CoreVersion.h/.cpp`: ApiVersion() = 1 (tested core_tests.cpp:91; logged CatchGameMode.cpp:18).
- `Catch/CoreTests/core_tests.cpp` (941), `core_runs.cpp` (447), `Tools/core-test.ps1` (MSVC C++20 /W4 /WX, no exceptions/RTTI).

## 2. As-is facts
### a. Price of a caught fish (Prices.cpp:34-81, called from LandFish :83)
1. Gold Sunfish: CoreRound(FeeOf(day) x pace.specialReward 0.5) (:39-43). Nothing else.
2. Sprat + Bait Knife: price 0, liveSprat, streak 0 (:44-50).
3. value (double) = FishData::price x fishLevels[kind] (:52); x hook priceMultiplier (Golden Grub 2, set at bite Rods.cpp:245) (:53).
4. Multipliers in fixed order, each from charm `value`: Sprat Fever x4 (Sprat), Giants x3, Deep Pressure x2 if catchDepth >= 300, Old Faithful x3 if 1 rod, Dusk Frenzy x2 if dusk (:60-64).
5. Chain: x(1 + chain x mods.chainBonus) (0.5; Food Chain sets 1.0, Shop.cpp:38) (:65-70).
6. Same Again: + streak x 3 after multipliers (:71-76); streak state lastLandedKind_/sameKindCount_ (Game.h:188).
7. CoreRound once (:77); rule 4: price < 10 -> 0 (:78).
8. After landing: Big Haul +CoreRound(price x 0.5) to every fish landed within 1.0 s game clock once 2+ landed, across rods (:103-123).
- Per-charm amounts = rounded deltas (charmAmounts). Types: double math, int64 results, saturating adds. No float weight anywhere.
### b. Money (State::money int64, Game.h:108)
- Start: pace.initialMoney 0 (Game.cpp:25). Every change goes through ChangeMoney (Game.cpp:69-73, emits MoneyChanged old/new/reason).
- +: ShipNext +batch price, reason Shipping (Days.cpp:111,126); summaryBonus (never set) (Days.cpp:132); SellCharm refund floor(paid x 0.5) (Shop.cpp:219-220).
- -: PayFee (Days.cpp:161); BuyUpgrade :151, BuyFishCard :194, BuyCharm :208, Reroll :229, BuyPack :242, BuyWormTin :254.
- Fee: FeeOf (Prices.cpp:17-31): byDay rows, then last x growth^(d-rows) rounded to 2 significant digits (CoreSignificant :8-16); Soft closed formula 15 x 1.5^(d-1). Table Tables.cpp:81-85: Soft 15,23,34..384; Normal 15,30,70,150,320,700,1500,3200,7000 x2.15; Hard 15,38,95..22900 x2.5.
- Check: PayFee in Summary: money < fee -> RunOver event (day, price=fee, money) and phase RunOver; else money -= fee, FeePaid, Victory once on day 9, tomorrow rule, Shop (Days.cpp:151-178). Catch total and money are the same currency today.
### c. Eating the hooked fish (Rods.cpp:378-437, EatFish :272-288)
- Check every 0.6 s per hook holding a fish (first 0.6-1.2 s, AttachFish :242); none if a hunter is coming or the prey is Gold Sunfish (:398-408).
- Candidate: FishData::hunter, Swim, not fleeing, tier > prey tier (:417); effective zone open (:418); edge flags :419-420; hook depth inside home band +-30 (:421); |dx| <= hunterReach 504.9 (:423). Nearest by |dx|.
- Chance: strikeChance 0.28 x factor; factor = mods.hunterRate (Fresh Scent x2, global) x bait hunterMultiplier (Siren Pearl 2) (:427-429). Strike speed 2.6 x eater speed x factor (:431). Eat within 16 units (:405).
- Eat: all fish of the hook doomed, chain += count eaten, eater attached as new bite (new hang, same catchDepth, Grub from bait), ChainLink with price of eater now (:272-288).
- Wide band for hunter kinds when zone open (Fish.cpp:55-59). Hunter tiers in data: Bass 3, Tuna 4, Angler 4, Gulper 5, Squid 6.
### d. Fish table (Tables.h:33-49, Tables.cpp:15-31)
- Fields: name, zone, price (int64), tier, shoal, hunter, floor, unique, hangSeconds, lowerBandFraction, unlock, count, speed, length, swimsFromDay.
- Target replaces price -> fishWeights kg (Sprat 2, Mackerel 5, Bass 12, Lantern 4, Cod 10, Tuna 30, Hatchet 7, Angler 45, Oarfish 80, Gulper 60, Snail 15, Squid 160, Flounder 40, Coela 400) and tier -> size (Lantern 1->2, Hatchet 2->3, Gold 0). WARNING: CatchSeaView.cpp:195-196 uses `tier` as the visual size of the fish; add a new `size` field, do not reuse tier.
### e. Baits (Tables.cpp:34-47)
- 12 baits; catch list, any, anyFromZone/Through, reusable, survivesEscape, bite/price/hunter multipliers. Live Sprat: Rare, 3 portions, accepts Bass, Tuna, Angler, Gulper, Squid, Oarfish (:42); made by Bait Knife on arrival (Prices.cpp:142-149). Matching: BaitMatches Fish.cpp:318-328, HookAccepts Rods.cpp:152-158, pack eligibility BaitUnlocked Shop.cpp:10-19 uses the catch list.
### f. Charms today
- 20 kinds, enum Tables.h:23. Effects are hard-coded branches: CompileMods switch (Shop.cpp:20-46) into global Mods (Game.h:133-142), plus `mods_.active[...]` checks in Prices.cpp, Rods.cpp, Days.cpp, Fish.cpp. Not data rows. No rarity, no counters.
- Shelf: slots = 5 + Bigger Shelf level + extraShelfSlots(0) (Shop.cpp:45); Weightless takes no slot (Shop.cpp:205-206).
### g. Shop (Shop.cpp:63-316, ShopData Tables.h:177-187)
- Charms: 2 offers day 1, then 3; uniform random among unlocked, not owned; Weightless 10% at CoreRound(x1.5) (:71-87). Prices 15-35 (Tables.cpp:59-78), unlock days 1/2/4.
- Reroll 3, +2. Cards: from day 2, 2 offers, price 6 + 4(L-1), effect fishLevels+1 (base x level). Packs Small 6 (3 show/1 pick), Big 14 (day 3, 4/2), open at once. Worm Tin 3 (+12). Upgrades: New Rod 20/45/100/220, Sharpen 12/30/70/160, Deeper 25/150/700/3000/12000 (Soft 25/70/160/380/900), Bigger Shelf 150/400 (day 4).
### h. Day rules and special fish
- Every 3rd day, fixed cycle (Days.cpp:8-14; rows Tables.cpp:87-93): 1 IgnoreWorm (Rods.cpp:155,163), 2 FastEscape drain x2 (Days.cpp:28-33), 3 no bites in Sunlight zone (Rods.cpp:302), 4 price < 10 -> 0 (Prices.cpp:78), 5 day -15 s (Days.cpp:24).
- Gold Sunfish enters at PrepareDay (Days.cpp:207, Fish.cpp:101-105), leaves at Summary (Days.cpp:141-142), price half fee. No coins.
### i. Money/price carriers and Unreal readers
- Events: FishLanded.price/charms/charmAmounts; ChainLink.price; ShipItem.price/money/count/charms; MoneyChanged old/new/reason; FeePaid price=fee, money; RunOver price=fee, money; DayEnded price=catchTotal; Purchase.price; CharmSold.price; CharmFired.price (amount); FishInBucket.price.
- State: money, catchTotal, bucketTotal, catchList[].price, shop offers .price, rerollPrice, fishLevels. Queries: FeeOf, PullPreview.money, FishInfo/FishInfoOf.price.
- Unreal readers: CatchHud.cpp:409 (rod value Money(Pv.money)), :390 Mods().noEscape, :411/418 charm::AutoReel on shelf, :412 HUNTER NEAR; :1463-1506 OnShipItem (TargetMoney = E.money, ShipSum += E.price, "+$"); :1512-1527 FishInBucket popup; :1531-1535 Gold popup; :1556-1565 ChainLink "CHAIN x"; :1570-1573 FeePaid "-$fee"; :1452 sale; :864-885 summary (best by price); :978-1112 shop prices vs Cash; :1696-1726 fish card (price, liveSprat, "Hunter. Eats a smaller fish"); :1977-2019 MoneyFacts (money, FeeOf day/day+1, catchTotal, bucketTotal, catchList price); :1079/:2104 Mods().packExtraChoices; :2703 catch.HudMoney.
- CatchTackleView.cpp:394 Scales.Step(bucketTotal, FeeOf(day)) (already points-vs-norm meaning); :714 F.Price = E.price; :333, :935 Mods().noEscape; :783 charm::ClusterHook.
- CatchSession.cpp:76/352 PullPreview; :137 Pay -> PayFee; :140-145 buys; :298-326 event logs with "$"; :340-399 dump; console catch.Pay :477, catch.Buy :481, catch.Sell, catch.Reroll, catch.NewRun level. No console command adds money.
- CatchLookSound.cpp:93 chainLength, :112-113 charm::ClusterHook / charm::BigHaul. Charm texts/tags keyed by core name: CatchUiText.cpp:5, CatchUiPanels.cpp:216.
### j. Tests
- core_tests.cpp: 20 test functions, ~240 CORE_REQUIRE. Areas: start state/units/bands (1), population (2), rods (1 + TimeRod x3), bite + 12x15 bait matrix (1), escape (1), worked price examples 4 cases (1), price order/Big Haul/Deep Pressure/Dusk (1), day end + shipping (1), fees 3 levels x 15 days (1), shop ~40 asserts (1), bait change (1), gold + rules (2), charms 8 cases (1), edge flags (1), movement 18.1 (1), progression gates 3 cases (1), 10,000 random commands determinism + invariants incl. money >= 0 and sum(price) == catchTotal (1), 1000-fish capacity (1).
- Price/money-dependent: WorkedExamples, PriceOrder, Fees, Shop, GoldAndRules, GoldReachAndLowTideHunter (Tuna 105), Charms, Bite (Chum, Live Sprat row), DayEndAndShipping (money == catchTotal), StartState (ApiVersion 1).
- RISK: fixture Quiet() (core_tests.cpp:16-23) silences eating by `hunter = false`. Under "every kind eats" that no longer works; tests need a new switch (strikeChance 0 or an eater flag).
- core_runs.cpp: 3 policies (saves-everything, buys-cheapest-useful, person-like) x Soft/Normal/Hard, variants reference, earned-depth, steady-build, tight-start, round2 fee-led/bait-gated/both. Prints per day: alive %, median money before fee, fee; diagnosis: first chain/charm/deeper/second rod day, mean income, fish count, income share by zone, chain %, charm %, purchases. Uses money, catchTotal, FeeOf, FishInfo price, hunter/tier (HunterNear :128-145).

## 3. Gap table (target = data 2.1 + ECON of the money probe)
| Target | State | Files |
| --- | --- | --- |
| Coins only money; catch never pays | missing (one currency) | Days.cpp, Game.h, Game.cpp |
| Points = weight x award, 9-step combine | missing (price pipeline) | Prices.cpp, Tables.h/.cpp, Game.h |
| fishWeights kg, size 1-6 | missing | Tables.h/.cpp |
| Award = card level, Blue Ribbon on award | partly (fishLevels x base) | Prices.cpp |
| Norm curves (ECON) | partly (fee code fits byDay+growth; numbers differ, Soft formula) | Tables.cpp |
| Settle: points >= norm -> payout 4 + over (25% steps, cap 4) + interest (1/5 coins, cap 3); else RunOver | missing | Days.cpp, Game.h |
| Gold: half norm points + 3 coins at landing | partly (half fee only) | Prices.cpp |
| Ladder: any kind but Gold eats size+1..+reach; eat factor | partly (hunter + tier >) | Rods.cpp:408-436 |
| eat_reach (Big Mouth 2, Free Feast 5) | missing | Rods.cpp |
| Live Sprat as size-1 lure | missing (accept list) | Fish.cpp, Rods.cpp, Shop.cpp |
| Chum level per kind, +50%/level from next day, cap 5, 2 shop offers, 3+level coins | missing | Game.h, Fish.cpp:62-75, Shop.cpp |
| Ground row: chum x2, card x2 (3+(L-1)), worm tin 1 | partly | Shop.cpp |
| 50 charms as rows (8 triggers, 15 conditions, 15 effects, 17 counts) | missing (20 hard-coded) | new Charms.cpp, Tables.h/.cpp, all |
| Charm rarity draw 60/30/10, prices 4/6/8, Weightless 6/9/12 (ceil), 3 offers from shop 1, no unlock days | missing | Shop.cpp, Tables |
| Reroll 2 +1; packs 2/4; upgrades in coins | partly (data only) | Tables.cpp |
| Counters per shelf charm | missing | Game.h ShelfItem |
| Per-rod/per-kind rule flags (auto_reel 3+ rods, no_escape rod lvl 3+, cluster same chummed kind, low_tide per kind, no_day_rule) | partly (global) | Shop.cpp, Rods.cpp, Fish.cpp |
| Big Haul +10%/other fish of the pull (no 1 s window) | different | Prices.cpp |
| Rule 3 hook above depth 100; rule 4 weight < 10 kg | different | Rods.cpp:302, Prices.cpp:78 |
| Thrifty Hook gone | exists, to remove | Shop.cpp, Rods.cpp |
| Hunters from day 2, bare hook Sunlight only (ECON hunterDay 2, bareZone 0) | partly (data gates exist) | Tables.cpp |

## 4. Block plan (dependency order)
1. **B1 Data model, additive.** Tables.h/.cpp: FishData `weight` (double kg), `size`; CharmData `rarity`; ShopData charm rarity weights/prices, chum, card 3/1, reroll 2/1, payout table, specialCoins; ECON fees and coin prices. Keep `price`/`tier`. Compiles. Tests: TestShop/TestFees numbers. Shim-safe.
2. **B2 Economy split.** Coins = State::money; shipping adds points to a day total, not money; PayFee settles (points >= norm -> payout coins, else RunOver); NormOf with FeeOf alias; MoneyReason +Payout, +Special, +Charm. Days.cpp:108-178, Game.h. Tests: payout parts and caps, RunOver with coins held, interest uses coins before payout. BREAKS meaning (compiles): CatchHud.cpp:1466 (TargetMoney = E.money), :1573, :1977-2019, CatchSession.cpp:298-326 logs.
3. **B3 Weight/points pipeline.** Accumulator {flat, percent, mult, awardMult, chainStep, coins}; steps 1-9; LandedFish/FishLanded/ChainLink/PullPreview/FishDetails carry points (+ weight). Remove Big Haul window (Prices.cpp:103-123, haul_, bigHaulWindow). Tests: v2.1 worked examples (Sprat eaten by Mackerel 7.5 kg -> 8; Bass 24; Tuna 75; 5 Sprats by Mackerel 17.5 -> 18; Bass card 3 chain 1: 54). Shim: fill `price` = points so Catch compiles.
4. **B4 Food ladder.** Size test, every kind except Gold eats, eat factor (Fresh Scent only on chummed prey), eat_reach hook (default 1), Live Sprat lure, eats/eatenBy in FishDetails. Rods.cpp:408-436, Fish.cpp:318-328, Shop.cpp:10-19. Tests: ladder per zone table of proposal 12.2; Quiet() fixture fix. Compiles; hover text CatchHud.cpp:1726 stale.
5. **B5 Ground.** chumLevels + activeChum (applied at PrepareDay, as probe enterPrep), BuyChum command, shop.chum offers, ContentType::Chum, WantedCount x (1 + 0.5 x active). Fish.cpp:62-75, Shop.cpp. Additive; Catch needs UI later.
6. **B6 Charm rows.** Evaluator + 50 rows; counters; triggers on_land/on_pull/on_escape/on_chain_link/on_day_end/on_shop_enter/on_purchase; per-rod/kind flags; dynamic slots; coins effects; rarity draw. Replaces CompileMods switch. BREAKS consumers: charm enum names (CatchHud.cpp:411,418; CatchLookSound.cpp:112-113; CatchTackleView.cpp:783) -> keep constants for auto_reel, cluster_hook, big_haul as shims; Mods().noEscape (CatchHud.cpp:390, CatchTackleView.cpp:333,935) -> per-rod field; name-keyed UI text tables. Tests: one case per trigger, effect kind and count; the builds as smoke runs.
7. **B7 Day rules + Gold.** Rule 3 depth 100, rule 4 weight, Storm Glass no_day_rule, Gold +3 coins. Small.
8. **B8 Runner and docs.** core_runs.cpp policies on coins/points; core-interface.md.
- 19:00 window of core-interface.md: FeeOf -> NormOf (alias); PayFee meaning (settle) or new SettleDay; State::money = coins; catchTotal/bucketTotal/LandedFish.price/PullEstimate.money/FishDetails.price/FishLanded.price/ChainLink.price/ShipItem.price/DayEnded.price = points (add `points`, `weight`); ShipItem.money meaning; FeePaid -> payout event {norm, points, base, over, interest, coins}; RunOver + points; MoneyReason values; new BuyChum, shop.chum, chumLevels, ShelfItem.counter, FishData.size/weight, FishDetails size/award/eats; per-rod noEscape/autoReel; ApiVersion 2.

## 5. Risks and open questions
- Norm numbers conflict: data units.norm "curves keep their numbers"; proposal 12.1 "Normal 25, 50, 100, 200"; brief day1-economy-probe.md:35 25,...; ECON played: Soft 15,40,65,100,165,260,420,670,1070 x1.6; Normal 15,50,100,...,6400 x2; Hard 15,60,150,380,950,2400,6000,15000,37500 x2.5 (journal.md:404 moved day 1 to 15); core/rules-reference 1.6: 15,30,70.. x2.15. Which one?
- Payout numbers exist only in ECON / brief (data says "do not change"). Gold +3 coins only in ECON (Storm Glass "+3 more" implies it).
- Unlock days: decision `no-day-locks-money-tight` comment keeps gradual unlock (big pack, card, shelf); brief item 7 opens everything from shop 1.
- Rule 3 (depth 100) is in data 2.1 but proposal section 11 says it goes to the owner; no owner decision found.
- Live Sprat bite: scan by hook zone, or eat conditions (band +30, 520 x)? Pack eligibility of Live Sprat without a list?
- Chum Bucket: Cluster timer x2 kept? Dusk Frenzy bite x3 dropped? Data rows have neither.
- `cap` in scaled effects: data note says count cap, texts mean value cap (Slow Hands 300%, Same Again 20 kg).
- on_day_end order: Rising Tide reads coins after payout; Tide Pool/Bait Trader also add coins on_day_end; order unsettled though shelfOrderMatters = false.
- on_land for Gold: steps 3-9 skip it, Storm Glass coins row needs it to fire.
- seconds_hooked: heldSeconds resets when an eater attaches (Rods.cpp:241). Intended?
- Shelf slots from Sea Chest can drop below occupied slots after spending coins.
- Rounding: weight double with (100+P)/100 gives x.4999999 cases; compute in integer hundredths or add epsilon. Weight shown 0.1 kg but points from unrounded weight; rule 4 on unrounded weight (9.96 shows 10.0, gives 0). Flat now multiplied by award (by design 12.1). Payout over = floor(4 x (pts - norm) / norm) in integers. Weightless ceil vs core nearest.
- rules-reference 1.6 (money pays fee), 5.6 (hunters, tiers), 7 (order, Big Haul window), 9.1 (Live Sprat list), 10.1/10.2/12 (unlock days, card price) and content-list.md (:51-52 hunter, :193 "$6", :248, :344-348, :421) contradict 2.1.
- Chum pool "kinds of open zones": table zone or effective zone?

## 6. New concepts the core lacks
- Row evaluator (triggers, conditions, effects, counts): large.
- Chum levels + active copy + shop row + command: medium.
- Ladder eat check with size and eat_reach: medium; Live Sprat as lure: small-medium.
- Per-charm counters: small. Trigger hooks on_pull/on_escape/on_day_end/on_shop_enter/on_purchase: medium.
- Coins effects mid-day and at day end: small. Payout settle: small.
- Per-kind low_tide (EffectiveZone per kind): small-medium. no_day_rule: small.
- Per-rod flags (auto_reel 3+ rods, no_escape level 3+): small. Cluster same chummed kind: small.
- Dynamic shelf slots (rods_level4, coins_held): small. Charm rarity draw: small.
- Counts today_top, bait_kinds, stock_portions, species_card3, margin_quarters, shelf_sea_others, pull_others: small each.
- Fractional weight + award: small.
