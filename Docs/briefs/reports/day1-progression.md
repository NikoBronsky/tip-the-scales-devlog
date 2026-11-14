# Day 1: progression measurement (report to the orchestrator)

Done: `edges.thriftyPortionReturns` is now off by default (the test sets both values; core-interface 10 has one line). The runner prints `reference` (3 policies x 3 levels) and the variants `earned-depth`, `steady-build`, `tight-start` (person-like x 3 levels) by name, each with a diagnosis on Normal. The proposal is `Docs/design/progression-proposal.md`. `MakeDefaultTables()` is otherwise unchanged. core-test passes (the run with `-RunBalance -Runs 20 -Days 9` takes about 96 s). Game target build: Succeeded, 0 warnings. check_all: 9 ok, 0 fail, 2 skip.

Causes (reference, Normal, person-like): (1) chains from minute one: 57% of day-1 income, 94% on day 2, in 100% of runs (Sea Bass, strike 0.28). (2) Deeper Line costs 25 in the first shop: bought day 1 in 100% of runs; day-2 income x7, 98% from Twilight; bare hooks catch it. (3) The fee does not press: money is 8x the fee on day 1, 32-44x on days 2-4. (4) The shop is cheap: 4.3 buys after day 1, 9-10 after days 2-3; second rod on day 1. (5) Charms come before the base works: bought day 2, 14-28% of income by days 3-5.

| Normal, 20 runs | reference | earned-depth | steady-build | tight-start |
| --- | --- | --- | --- | --- |
| Money/fee day 2 | 32 | 8.5 | 6.8 | 5.2 |
| Buys after fee d1/d2 | 4.3/9.4 | 4.8/6.2 | 3.1/2.6 | 1.8/1.6 |
| First Deeper Line / charm | d1 / d2 | d3 / d1 | d2 / d3 | never / d3 (30%) |
| Alive d5 / d9 | 100/100 | 100/100 | 90/90 | 30/0 |
| Money/fee d6 / d9 | 30/3.7 | 13/3.7 | 15/10 | 0.9/- |

Recommendation: steady-build (Deeper Line 60/300/1500/6000/20000 from day 2, charms from day 3, New Rod 40/100/240/520, strike 0.18), plus the core gate "hunters strike from day 3". Only this variant spreads the firsts over days 1-3, keeps 90% alive on day 5, and lets a working build reach 10-15x the fee. No data lever removes the day-1 chain: it stays in 100% of runs in every variant.

Policy fix: person-like now also buys charms (value 25% of last income). Before, it never bought one, so the charm share was 0 and the reference looked weaker than a person plays it. With the fix the reference survives day 9 in 100% of runs on Normal (65-85% in the two earlier runs).

Levers that need a core change (not used in the runs): hunters strike from day N (~3 lines + 1 field, recommended); a bare hook does not catch below zone N (~3 lines + 1 field); a species swims from day N (~4 lines + 1 field per fish).

Orchestrator decisions: none changed. Target 3.4 day 1-2 "no chain without a choice" cannot be met with data only. Target "one purchase" is met only by tight-start, which fails the 80% day-5 target on Normal (it passes on Soft).

Questions for the owner: (1) which variant: steady-build, earned-depth or tight-start? (2) may the core get "hunters strike from day N"? (3) about 10x the fee on day 9 for a working build, or a harder late fee? (4) unlocking between runs: no for the jam (recommended: save file, meta UI, second balance curve, the jury plays 1-2 runs), or yes?

## Round 2

- Gates are data with defaults that keep the reference: `FishData::swimsFromDay` (1), `PaceData::bareDeepestZone` (5), and a list bait is offered only when one of its fish swims today or tomorrow. Test: TestProgressionGates.
- Policy fix, round-2 variants only: the reserve is tomorrow's fee minus half of today's income. The bait choice respects the bare gate. Round 1 still reproduces.
- The runner prints `fee-led`, `bait-gated` and `both` (person-like Normal with diagnosis, buys-cheapest-useful Normal, person-like Soft and Hard). `core_runs.exe N D round2` runs only these.
- Best: **both** (Sunlight-only bare hook, hunters from day 3, Small Pack 4, Normal fee 20/30/60/100/300/600/1000/2200/5000). Alive 90% on day 5 and 45% on day 9. Money/fee 2.6/2.3/3.1 on days 1-3. Chain, Deeper Line and charm first come on days 3, 3 and 4.
- Target B misses in both: money/fee 5.1 on day 4; 7.8 buys after day 4 and 2.0 after day 1; second rod on day 4; days 6-9 at 1.2-2.8x, not 5-15x. buys-cheapest-useful dies on days 4-5, not 5-7. Hard is untuned (dead on day 4).
- Cause of the misses: opening Twilight multiplies the income about 3.5x in one day. A fee that presses on day 4 blocks the day-3 investment (a day-4 fee of 220-250 killed every run). Owner question: make the depth step smaller (Twilight prices), or keep it as the big moment of the run?
- fee-led is a knife edge (35% alive on day 5). bait-gated with the reference fee dies on day 5 (20% alive).
- Game target builds with 0 warnings. core-test passes. The full runner prints 30 blocks.
