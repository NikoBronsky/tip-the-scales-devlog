# Day 1 step 4: core aligned with rules-reference.md

No edit to the Unreal layer. `Game.h` changes are additive; the game target builds (`Build.bat Catch Win64 Development`: Succeeded, 0 warnings).

| # | Gap of reference 17 | Done | Test |
|---|---|---|---|
| 1 | Per-species counts and bands | yes | TestStartState |
| 2 | Growth 1 + 0.08 (day-1), cap x2 | yes | TestPopulationGrowth |
| 3 | Day end pulls every rod, lands all | yes | TestDayEndAndShipping |
| 4 | Money rounds to nearest | yes | TestWorkedExamples, TestPriceOrder |
| 5 | Big Haul last, 1 s window | yes | TestPriceOrder |
| 6 | Rod speed 0.8^(level-1) | yes | TestRodTimings |
| 7 | Fee: Soft formula, 2 significant digits | yes | TestFees |
| 8 | Fixed rule cycle | yes | TestGoldAndRules |
| 9 | Rules 1/3/4 skip gold, 2 applies | yes | TestGoldAndRules, TestEscape |
| 10 | Gold Sunfish reach (band 40-2470) | yes | TestGoldReachAndLowTideHunter |
| 11 | Giant Squid band to 2950 | yes | TestStartState |
| 12 | Low Tide open-zone check | yes | TestGoldReachAndLowTideHunter |
| 13 | Bait change in the water at once | yes | TestBaitChange |
| 14 | Empty pull allowed | yes | TestRodTimings |
| 15 | Bought pack opens at once | yes | TestShop |
| 16 | Persistent sea | yes | TestPersistentSea |

Other rules aligned: units (`PaceData::xScale`), rod x, cast stagger, zone-speed travel, min goal and clamp, scans, bite rates (bare /4, Chum, Dusk), hang 7 s and escape, flee, hunters and chains, Cluster Hook, Auto-Reel, Thrifty, Bait Knife, Same Again, Deep Pressure, Giants, shipping 0.35/0.5x0.88^i/0.8 s and batches above 12, Gold Sunfish, Victory on day 9, shop offers, rerolls, Deeper Line Soft row, the five 15.1 edge flags (`Tables::edges`).

Test tail: `PASS determinism: 10,000 random commands ... / PASS capacity: 1,000+ fish at the fixed step / CORE TESTS PASSED / core-test: exit 0; 12,11 seconds`. `check_all.py`: 9 ok, 0 fail, 2 skip.

Runner, person-like, Normal, 20 runs (day / runs reaching / alive % / median money before fee / fee):
`1 20 100 136 15 | 2 20 100 910 30 | 3 20 100 2312 70 | 4 20 100 2860 150 | 5 20 100 3092 320 | 6 20 100 4409 700 | 7 20 100 5067 1500 | 8 20 100 5633 3200 | 9 20 65 7459 7000`.
The other two policies die on day 4-5 (Normal). The gap comes from early Deeper Line: deeper fish pay far more and bare hooks catch them. Measurement only; no number changed.

Interface additions: `SetFeeLevel`, `FishInfoOf`; events `FishInBucket`, `Victory`; `ChainLink.kind`, `Event.catchIndices`; new fields on FishEntity, HookData, RodData, LandedFish, State, FishDetails (see core-interface.md 10).

Flows changed for the Unreal layer: `Advance` moves the sea in every phase; landed/eaten fish leave `state.fish`; fish x about -68..1068; `DayEnded` fires when shipping starts; `Shipping` includes day-end pulls and flights; `SkipShipping` does not open Summary at once (0.8 s after the last item; an early skip is remembered); `SetGoalDepth` clamps (NothingToDo when Hooked/Pull); `SetRodBait` with zero stock is NothingToDo; empty `PullRod` is Ok; `BuyPack` opens the pack; one pack offer per kind with `locked`; `OpenPack`/`PickBait` work in the day; `StartDay` refuses while a pack is open; escape timers are real seconds; the rule is chosen at prep.

Questions:
1. Depth easing 0.6, sway 1, school catch-up 2 and the p-to-0 change are not in the reference; values are data guesses.
2. A Thrifty free portion returns to stock on a bait change (literal reading). This can be farmed. Keep?
3. Pull preview: no Same Again advance, no Big Haul. Is that the intended reading?
4. Cluster Hook: extras plus incoming fish limited to 4. Confirm.
5. No EscapeSoon event while Barbed Hooks is on. Confirm.
6. Every species keeps at least 1 fish (wanted count >= 1). Confirm.
7. The shop's 1.05 s delay, the victory screen and the drag-depth preview are left to the Unreal layer.

Errors in the reference: it says "nothing else was left undetermined", but question 1 lists missing rates; the Same Again preview wording is ambiguous; the Thrifty return on a bait change is not stated.

## Follow-up: section 18

- 18.1 in `PaceData`, x values through `xScale`: school centre (sway 14 at 0.4, easing 1.5), member (x sway 4 at 1.3 + 0.1 phase, depth sway 3 at 0.9, step min(d, (1.15 x speed + 1.5 x d) x dt), faces the target past 30), solo (sway 10 / floor 1 at 0.6 + 0.05 phase, easing 3, new base 0.08), leaving (x1.8, depth easing 1), p falls 1.6 (hook, strike) and 2 (Cluster Hook), 0 on the hook. Cluster join is now "closer than 14". The guesses 0.6 / 1 / 2 are gone.
- Readings: the sway phase is the FishId (the reference does not define "phase"). The reference sets no clamp, so a Sprat school member reached depth -0.1 (base 24 - 14 - 18 - 3). The core now keeps every depth target in the water, between 0 and the floor. Size easing (rate 2) is visual only, so it is not in the core.
- 18.2: items 1, 2, 3 and 6 match the code, so nothing changed there. Item 4 is now behind the new flag `edges.thriftyPortionReturns` (default true), with a test for both values. Item 5 is right: Thrifty makes one roll per cast for all empty hooks (TestCharms).
- Hang 6.0: the core is right. Hang is per species (2.2): Sprat 7; Mackerel, Sea Bass, Cod, Lanternfish, Hatchetfish, Snailfish, Flounder and Gold Sunfish 6; others 4-5. The dump reads `HookData::fullHangSeconds`, which is correct (fish hang + rod level - 1).
- New test TestMovement18 (18.1 numbers, one-step solo depth/x check, p fall rate, Thrifty flag). The tests pass in 11.9 s, and determinism and capacity still pass. The game target builds (Fish, Rods, Tables recompiled, 0 warnings). check_all: 9 ok, 0 fail, 2 skip.
- Runner, person-like on Normal, day 9: 85% alive, median 8524. The other figures are about the same as before.
- core-interface.md 10: lists the new pace fields and the new edge flag.
