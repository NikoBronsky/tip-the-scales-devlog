# Brief: core 2.10 - link charms (four candidates that make the risky-fishing charms trigger each other)

Written by the orchestrator on 14.11.2026, 14:29. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `d5701e6` (core 2.9.3, API 9). You do not commit.

## 1. Goal

The game director watched the plan of the video and wrote on 14.11 (Russian, dictated by voice):

"Я просто ждал, что у нас будут какие-нибудь модификаторы которые спавнят динамит когда срабатывает удача, динамит наоборот - динамит не уменьшает количество рыбы, а увеличивает, но забирает рыбу из улова, но вместо. Спавн гарпуна за цепочку, замена удочек гарпунами (в зависимости от наживки у гарпунов разные эффекты). Динамит выгоняет рыбу со дна на верхние уровни. Рыба начинает есть друг друга не только на крючке (вырастает за каждую съеденную рыбу, но если голодает - уменьшается) и т.д. Реально такое успеть реализовать?"

and then: "Динамит наоборот - прибавляет рыбу в море, но забирает улов из ведра (не весь)".

What he misses: charms that make other charms fire. The "madness" of a late build must come from the set on the shelf, not from single strong charms. The orchestrator answered that four of his ideas fit into today. Build the core side of these four as candidates (behind the same switch as Dynamite and Harpoon). He plays them in a build tonight and decides before the rule freeze at 24:00. The other ideas (harpoons instead of rods, fish that eat each other in open water) are NOT part of this task; do not build any part of them.

The default game (switch off) must stay exactly as core 2.9.3: same offers, same rule days, same fingerprints.

## 2. What exists now (verified by the orchestrator on `d5701e6`; verify again before you rely on a line number)

- Read first: `F:\UE\EpicJam\Docs\briefs\day3-core29-risky-fishing.md` (sections 2 and 3: charm data flow, the candidate switch, the extra fish of a pull, Dynamite, Harpoon, the measure) and the newest core reports in `Docs/briefs/reports/` of your worktree. Their conventions hold here.
- Charms are data: `Docs/design/modifiers-v2.data.json` -> `Catch/CoreTests/gen_charm_rows.py` -> `Catch/Source/CatchCore/Private/CharmRows.inc`. Dynamite and Harpoon rows: `CharmRows.inc:681-698`. Their rule texts today: Dynamite `A pull that lands a fish blasts the water: every fish near this rod's hooks lands too, at 35% weight. After each blast the sea holds 20% fewer fish ...`; Harpoon `A pull that lands a fish also throws a harpoon at the nearest fish. It bounces to 2 more fish nearby. The 1st fish weighs x0.5, the 3rd x1.5`.
- Triggers and effects: `enum class CharmTrigger { Passive, OnLand, OnPull, OnEscape, OnChainLink, OnDayEnd, OnShopEnter, OnPurchase, OnLucky }` (`Public/CatchCore/Tables.h:116`); `CharmEffectKind` ends with `Blast, Harpoon` (`Tables.h:132-133`); both are passive effects today, used only at the start of a pull.
- The extra fish of a pull (Harpoon picks, then the blast): `Private/Rods.cpp:643-760` (`PullExtras`, event `HarpoonThrown` at about line 759, the `Blast` event next to it). The count is stored on the rod (`rod.pullExtras`, `Private/Prices.cpp:269`).
- A chain link: the `OnChainLink` rows fire in `Rods.cpp` near line 369 (Lucky Bite). A lucky chance that comes true: event `LuckyChance` and the `OnLucky` rows, `Private/Charms.cpp:454-460` (Fish Tale grows on it).
- The sea factor of Dynamite: `State::seaFactor`, used in `Game::WantedCount` (`Private/Fish.cpp:83`), reset when the day is prepared (`Private/Days.cpp:338`). `Game::Refill` (`Fish.cpp`, about 222-235) adds one fish every `pace.refillSeconds`.
- Zones of kinds: the rule flags `LowTide` (chummed kinds one zone higher) and `MirrorZones` (`Tables.h:146`) change the home zone of a kind when a day is prepared. Find how a free fish holds its depth during the day before you design block C.
- Offer conditions by tag: `Private/Offers.cpp:163` (`lucky_charm` = Lucky Bite is on the shelf; Fish Tale uses it).
- Balance program: `Catch\Saved\CoreTest\core_runs.exe <runs> <days> --only <profile>`; 300 runs x 12 days of one profile took 684 s.
- Band of the careful bot (300 runs, alive after day 12): no charm 22.0 %, Dynamite 42.0 %, Harpoon 52.0 %, accepted band 27.0-52.0 %.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Every number is a temporary balance number set by the orchestrator. Keep each one as data (`PaceData` or the data file), never as a literal in the code. The four charms are candidates: `"candidate": true`, no task, not start charms, open with a profile, offered only with `Tables::candidates` on.

1. **One blast, one harpoon throw, many sources.** Make the blast and the harpoon throw callable from a trigger, not only from the start of a pull: `(rod, hook or point, source charm)`. A blast or a throw that happens outside a pull lands its fish at once through the same extra-fish path as in 2.9 (same pick rules, chain 0, no bait effect, events `Blast` / `HarpoonThrown` before the `FishLanded` events, the source charm credited). Its pull size is the number of fish it took. `OnPull` rows do not fire for it. If landing outside a pull costs you more than about 45 minutes, fall back to "stored on the rod and added to the extras of this rod's next pull", write the rule texts to match, and say so.
2. **Lucky Fuse** (kind `LuckyFuse`, id `lucky_fuse`, class `rod`, rarity `rare`). Rule text: `When a lucky chance comes true, the water blasts at that rod: every fish near its hooks lands too, at 35% weight.` Russian for the data file: name `Счастливый фитиль`, rule `Когда удача сбывается, у этой удочки гремит взрыв: вся рыба рядом с её крючками тоже попадает в улов, с весом 35%.`
   - Row: trigger `OnLucky` -> the blast of decision 1 at the rod of the lucky chance. Same radius, weight factor and sea cost as the Dynamite blast (the same `PaceData` numbers; the sea cost applies to every blast).
   - Offered only when a source of lucky chances is on the shelf (the existing tag `lucky_charm`).
3. **Chain Shot** (kind `ChainShot`, id `chain_shot`, class `rod`, rarity `uncommon`). Rule text: `Each new link of a chain throws a harpoon from that hook at the nearest fish. It bounces to 2 more fish nearby.` Russian: name `Цепной выстрел`, rule `Каждое новое звено цепочки бросает гарпун от этого крючка в ближайшую рыбу. Гарпун отскакивает ещё к 2 рыбам рядом.`
   - Row: trigger `OnChainLink` -> the harpoon throw of decision 1 from the hook of the link (its depth now). Same reach, hits and factors as the Harpoon charm. It works without the Harpoon charm on the shelf. The fish that just ate and the fish on hooks are never targets (the pick rules already say "free fish").
4. **Shockwave** (kind `Shockwave`, id `shockwave`, class `sea`, rarity `uncommon`). Rule text: `Each blast drives every fish one zone higher until the day ends.` Russian: name `Ударная волна`, rule `Каждый взрыв гонит всю рыбу на одну зону выше до конца дня.`
   - Day state `State::zoneLift` (0 when the day is prepared). Each blast (from any source, see 5 too) adds 1, not above `shockwaveMaxLift` = 4. The home zone of every kind for the rest of the day is `max(top zone, home - zoneLift)`, applied after Low Tide and Upside Down. Fish that are free now move to their new zone if the core's movement can steer them there; if it cannot without a new movement model, apply the lift only to fish that enter after the blast and say so. A catch counts in the zone where the fish is caught, as today.
   - Event `ZoneLift`: `value` = the lift after, so the view can show the drive upward.
   - Offered only when a blast source is on the shelf: a new tag `blast_charm` = Dynamite, Lucky Fuse or Bait Bomb.
5. **Bait Bomb**, the director's "reverse dynamite" (kind `BaitBomb`, id `bait_bomb`, class `rod`, rarity `rare`). His rule: it adds fish to the sea and takes a part of the catch out of the bucket, not all of it. Rule text: `A pull of 3 or more fish throws 2 fish from your bucket back into the sea. The sea then holds 25% more fish until tomorrow, up to x3.` Russian: name `Прикормочная бомба`, rule `Рывок на 3 рыбы и больше бросает 2 рыбы из ведра обратно в море. В море становится на 25% больше рыбы до завтра, не больше x3.`
   - Trigger: a pull of the player or Auto-Reel that lands `baitBombPull` = 3 or more fish (hooked plus extras), after the fish of this pull are in the bucket. Not at the automatic pulls of the day end.
   - Cost: `baitBombTake` = 2 fish leave the bucket, drawn with the core rng from all fish landed today that are still in the bucket (never a prize kind, never the Gold Sunfish). Find how the day's catch, the points of the day and the sale at the day end are stored, and remove the fish consistently: their points leave the day's points, they are not sold, counters of "fish landed today" that tasks and charms read do not go down (they were landed). If fewer than 2 such fish are in the bucket, the bomb does nothing.
   - Gain: `seaFactor` is multiplied by `baitBombSeaFactor` = 1.25, not above `seaCeil` = 3.0 (the Dynamite floor 0.3 stays). At once `baitBombEnter` = 8 fish enter from the edges, of kinds below their new wanted count (the normal refill rule for the kind, the core rng).
   - It is a blast for decision 4 and for the tag `blast_charm`, but it lands no fish. Event `BaitBomb`: rod, `otherFishIds` = the fish taken from the bucket, `count` = fish that entered, `value` = the sea factor after. Use a new event type; do not reuse `Blast` (the view shows it differently).
   - Day end: if removing points drops the day below a norm that the player had already reached and the day-end rule had already fired, do not un-end the day; say in the report what happens and what you chose.
6. No loops: a fish landed by a blast or a throw has chain 0 and rolls no lucky chance. Add a test that a full shelf of all six risky charms (Dynamite, Harpoon and these four) with Lucky Bite cannot recurse.
7. API number 10. Additions only; nothing the view of API 9 uses may change. New enum values go at the END of each enum. Core version 2.10.0.
8. **Open with the director (added 14:33).** He then wrote: "Ну у нас же есть динамит? Он должнен глушить и собирать рыбу но уменьшать ее количество в море на (1 даунгрейд)". The orchestrator's reading: the unit of the sea cost of a blast is one level of the fish-amount upgrade (chum: `ShopData::chumGrowth` = 0.5 per level, `State::chumLevels` / `chumActive`, `Fish.cpp:80-82`), and the Bait Bomb gain is the same unit upward. Whether a level is taken for the day or for the run, and from which kinds, is asked and not answered yet. So:
   - Put the sea cost of a blast and the sea gain of the Bait Bomb behind one function each (source charm, the kinds taken or thrown back), so the unit can change without touching the callers.
   - Build blocks A, B, C and their part of E first. Before you start block D, read `F:\UE\EpicJam\Docs\briefs\day3-core210-addendum1.md` if it exists; it then replaces the cost and gain numbers of decisions 2 and 5. If it does not exist when you get there, build block D as written in decision 5 and leave the Dynamite cost as it is.
9. Do not touch `Docs/design/core-interface.md` (the orchestrator updates it in its window) or anything under `Catch/Source/Catch/` (the view).

## 4. Work by blocks

A. Decision 1 (callable blast and throw, landing outside a pull) with tests; the 2.9 behaviour of Dynamite and Harpoon at a pull is unchanged (their tests stay green untouched).
B. Lucky Fuse and Chain Shot: data rows, generator, offer tag, tests (the blast on a lucky chance, the throw on each link, the pick rules, credit to the source charm, offers only with the switch on, Lucky Fuse only with a luck source).
C. Shockwave: lift state, zones, event, offer tag, tests (lift per blast, the cap, reset on the next day, with Low Tide and Upside Down, catch zone conditions such as Deep Reel see the real catch zone).
D. Bait Bomb: bucket removal, sea factor ceiling, the burst of entering fish, event, tests (trigger size, fewer than 2 fish, points and sale, determinism, with Dynamite on the same shelf).
E. Measure with `core_runs` (careful bot, default mode, candidates on, 300 runs, 12 days, alive after day 12; add profiles the way 2.9 did): Lucky Bite + Lucky Fuse forced; Chain Shot forced; Dynamite + Shockwave forced; Bait Bomb forced; and one run with all six risky charms plus Lucky Bite on a widened shelf as a stress test (no crash, capacity test still passes, time per run). A video take may be recording on this machine: start each measure as its own process with below-normal priority, at most three at once, and rebuild nothing while they run. Report the numbers against the band 27.0-52.0 %. Do not tune numbers yourself; propose.

## 5. Done means

- Switch off: both fingerprints equal core 2.9.3 (record them before your first change), no candidate is ever offered.
- Switch on: the four charms can be offered, bought and work as written; every new number is data.
- `Tools\core-test.ps1` passes (determinism and capacity included). `python -X utf8 Tools/check_all.py` in your worktree gives no fail.
- The measure table of block E is in the report.
- English and Russian texts are in the data file; no Cyrillic under `Catch/`.

## 6. Verification commands

```
powershell -NoProfile -ExecutionPolicy Bypass -File Tools\core-test.ps1
python -X utf8 Tools/check_all.py
Catch\Saved\CoreTest\core_runs.exe 300 12 --only <profile>
```
Run the gate and a bot run one after another, never at the same time (they share object files).

## 7. Boundaries

- Write only inside `F:\UE\EpicJam_core`: the core module, `Catch/CoreTests/`, `Docs/design/modifiers-v2.data.json`, your report. No other worktree, no editor, no Unreal build, no packaged game.
- Do not start, stop or touch any process you did not start (a game and a screen grab may be running for the video).
- Invent no rule beyond this brief. If a rule question appears, write it into the report and build the part that needs no answer.
- Delete nothing. No commits.

## 8. Report

`Docs/briefs/reports/day3-core210-link-charms.md` in your worktree, in English: what changed by file; every decision you changed and why; the measure table; for the view executor a short list "new events, new fields, new charm ids, what a frame must show for each charm"; open questions; what is not verified. Final message to the orchestrator: the same in short, with the fingerprints and the test and gate results. Time box: about 75 minutes. You report to the orchestrator, not to the director.
