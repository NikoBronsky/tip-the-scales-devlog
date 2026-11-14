# Brief: core 2.9 - risky fishing candidates (Dynamite, Harpoon, two rule days)

Written by the orchestrator on 14.11.2026, 10:10. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_core`, branch `track/core`, base `1964061` (core 2.8.1, API 8).

## 1. Goal

The game director asked on 14.11 (Russian, dictated by voice, he then left for several hours): «Я бы ещё, может быть, какие-то варианты типа гарпунов добавил или взрывчаткой, чтобы какой-нибудь вид ловли, который будет зависеть там от рикошетов ... вот динамит какой-нибудь, который достаёт много рыбы, но при этом уменьшает их количество. Или там боссов там таких же ... типа вредитель ... или браконьерская ловля, что за каждую выловленную рыбу у тебя уменьшается количество рыбы, и она там тощает, худеет и так далее. Я бы такие варианты посмотрел.»

Build the core side of four candidates so he can look at them in a build tonight: two charms (Dynamite, Harpoon) and two rule days (Poachers, Pest). They are candidates, not accepted content: the default game must stay exactly as core 2.8.1 (same offers, same rule days, same fingerprints). A switch turns the candidates on. The freeze of new rules is tonight at 24:00, so this is the only pass: keep it small and safe.

## 2. What exists now (verified by the orchestrator; verify again before you rely on a line number)

- Charms are data: `Docs/design/modifiers-v2.data.json` -> `Catch/CoreTests/gen_charm_rows.py` -> `Catch/Source/CatchCore/Private/CharmRows.inc`. 59 kinds, enum `charm` in `Public/CatchCore/Tables.h:27-40` (last: `Whopper`). Each data row has `nameEn`, `nameRu`, `ruleEn`, `ruleRu`, `class`, `rarity`, `price`, `needs`, `task`, `rows`.
- Offers: `Game::RollCharmOffers` (`Private/Shop.cpp:66`) takes kinds with `tables_.charms[kind].inShop` (line 70). Test seam `ShopData::testOfferOrder` (`Tables.h:412`, used at `Shop.cpp:84-85`): the first listed kind that is in the pool and not on the shelf goes into offer 0 at its plain price.
- A pull: `Game::PullRod` -> `Game::StartPull` (`Private/Rods.cpp:643-664`), then `Game::LandRod` (`Rods.cpp:666`) when the rod arrives. `LandRod` computes `pullSize = PullSizeOf(line)`, lands the main fish of each hook with `ContextOf(rod, hook, true, pullSize)` and the Cluster Hook extras with `ContextOf(rod, hook, false, pullSize)`, then fires the `OnPull` rows. `Game::LandFish` is in `Private/Prices.cpp:268`, `Game::WeighFish` in `Prices.cpp:123`.
- Free fish: `State::fish` (`FishEntity`: id, kind, x, depth, state Swim / ToHook / OnHook / Strike / Leave, `entering`, `fleeSeconds`). `Game::Doom` (`Private/Fish.cpp:195`) removes a fish.
- Population: `Game::WantedCount` (`Fish.cpp:67-85`, never below 1); `Game::Refill` (`Fish.cpp:222-235`): every `pace.refillSeconds` one fish of a random kind below its wanted count enters from an edge. It draws from the core rng.
- Rule days: `enum class DayRule` (`Tables.h:45`), rows `t.dayRules` (`Private/Tables.cpp:115-121`: rule, English text, value, light). `Game::DrawRuleOrder` (`Private/Days.cpp:19-57`) draws the order of a run from the seed with its own generator; `Game::RuleOf(day)` (`Days.cpp:9`) is the public lookup; `Game::ActiveRule()`; `Game::RuleValue(rule)` (`Days.cpp:71`).
- Fingerprints of core 2.8.1: default mode `b0d881d44f065645`, timed mode `7d632f58e4dd608e`.
- Conventions of this track: read `Docs/briefs/reports/day3-core28-unlocks.md` (your predecessor's report) before you start.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Every number below is a balance number set by the orchestrator. Keep each one as data (tables or the data file), never as a literal in the code.

1. **Candidate switch.** `Tables::candidates` (bool, default false). `CharmData::candidate` and `DayRuleData::candidate` (bool, default false; the data file gets `"candidate": true` on the two new charms, the generator passes it through). With the switch off a candidate charm is never offered (shop, reroll, reward, prize) and a candidate rule is never drawn; the draws of the default game must be the same as in 2.8.1, bit for bit. With the switch on the candidates are ordinary content: in the offer pool by their rarity, in the rule bag as non-light rules.
2. **Test seam for rule days.** `PaceData::testRuleOrder` (a list of `DayRule`, empty in the game): the first rule days of a run take these rules in order, then the drawn order goes on. It works with the candidate switch on or off (a listed candidate rule is used anyway). The view will set it, together with `shop.testOfferOrder`, for a launch argument that shows the candidates early.
3. **Extra fish of a pull** (shared by Dynamite and Harpoon). When the player or Auto-Reel starts a pull during the day (not the automatic pulls of the day end), the charms pick free fish and land them at once, at `StartPull`, before the rod travels. An extra fish:
   - is a free fish in state Swim or ToHook, not entering, not fleeing, not a prize kind (`FishData::prize`), not `unique`, not a fish already picked in this pull;
   - lands through the normal landing path, so every on-land row, the catch list, the bucket flight, the tasks and the day rules treat it as a landed fish; chain 0; not the main fish; no bait effect (bait None, bait multiplier 1) whatever hangs on the hook; the hook's bait is not used up; rod = the pulling rod; hook = the hook it was picked from (Dynamite: the nearest hook; Harpoon: hook 0);
   - counts in the pull size: the pull size of this pull is the hooked fish plus all extra fish, for the extra fish now and for the hooked fish when they land later (store the count on the rod until `LandRod`). The `OnPull` rows still fire once, in `LandRod`, with the full pull size.
   - If a fish on its way to a hook (ToHook) is taken, release the hook's target the way `ReleaseTargets` does.
   A pull that lands no hooked fish still triggers the charms (an empty pull can blast).
4. **Dynamite** (new charm kind `Dynamite`, id `dynamite`, class `rod`, rarity `rare`, price as other rare charms, `candidate`, no task, not a start charm). Rule text: `Each pull blasts the water: every fish near this rod's hooks lands too. After each blast the sea holds 20% fewer fish until tomorrow.` Russian for the data file: name `Динамит`, rule `Каждый рывок взрывает воду: вся рыба рядом с крючками этой удочки тоже попадает в улов. После каждого взрыва в море на 20% меньше рыбы до завтра.`
   - Near = distance in the (x, depth) plane from a hook of the pulling rod (hook x = rod x, hook depth = its depth now) at most `dynamiteRadius` = 130 core units. Every hook of the rod blasts; a fish in reach of two hooks lands once.
   - Cost: a blast that took at least one fish multiplies a day factor `State::seaFactor` (starts at 1 each day, reset when the day is prepared) by `dynamiteSeaFactor` = 0.8, not below `dynamiteSeaFloor` = 0.3. `WantedCount` of every kind is multiplied by it (still at least 1). Fish already in the sea stay; the refill simply stops for a kind until it is below its new wanted count. No effect on the Gold Sunfish or the Whopper entry.
   - Event `Blast`: rod, x = rod x, count = fish taken, `otherFishIds` = their ids, `value` = the sea factor after. It comes before the `FishLanded` events of those fish. No event when nothing was in reach.
   - A second `run` factor for the rest of the run is NOT built now; keep the day factor as one data number so the orchestrator can change it.
5. **Harpoon** (new charm kind `Harpoon`, id `harpoon`, class `rod`, rarity `uncommon`, `candidate`, no task). Rule text: `Each pull throws a harpoon at the nearest fish. It bounces to 2 more fish nearby. The 2nd fish weighs x1.5, the 3rd x2.` Russian: name `Гарпун`, rule `Каждый рывок бросает гарпун в ближайшую рыбу. Гарпун отскакивает ещё в 2 рыбы рядом. Вторая рыба весит x1,5, третья — x2.`
   - The first target: the nearest free fish to hook 0 of the pulling rod (its depth now) within `harpoonReach` = 260 core units. Each bounce: the nearest fish not yet hit within `harpoonReach` of the fish hit last. At most `harpoonHits` = 3 fish. Ties: the lower fish id.
   - Weight factors by hit order: `harpoonFactors` = {1, 1.5, 2}, applied as a weight multiplier credited to the charm (one effect entry of the weigh, the way a Mult row is credited, so the charm shows in the fish's charm list and fires `CharmFired`).
   - Event `HarpoonThrown`: rod, x = rod x, `seconds` = depth of hook 0, count = hits, `otherFishIds` = the hit fish in order. Before their `FishLanded` events. No event when no fish was in reach.
   - With both charms on the shelf the harpoon picks first, then the blast takes the rest.
6. **Rule day Poachers** (`DayRule::Poachers`, candidate, not light). Text: `Poachers: no new fish swim in today.` Value 0 = the share of the refill that stays (data; 0 = none). On that day `Refill` adds no fish (with a value between 0 and 1: the refill clock runs at that share). The sea starts the day as it is and only shrinks. The Gold Sunfish and the Whopper still enter. The next day the refill works again and fills the sea from the first second (say in the report how many seconds a drained sea needs).
7. **Rule day Pest** (`DayRule::Pest`, candidate, not light). Text: `Pest: each fish you land makes the next ones 3% lighter, down to half weight.` Value 3 (percent per landed fish); floor `pestFloor` = 0.5 (data). The factor `max(floor, 1 - value/100 x fish landed today)` multiplies the weight of every fish that lands after (combine step of the other weight multipliers; the Gold Sunfish is exempt; a Bait Knife Sprat that becomes bait does not count as landed). Expose it as `State::dayWeightFactor` (1 on other days) so the view can draw thinner fish and the info card (`FishInfo`, `FishInfoOf`, `PullPreview`) shows the weight a fish would have now.
8. Storm Glass switches a rule day off (`State::dayRuleOff`): both new rules then do nothing, like every other rule.
9. API number 9. Additions only; nothing the view of API 8 uses may change. New enum values go at the END of each enum (`charm`, `DayRule`, `EventType`, any other).
10. Tasks and profile: the two charms have no task; with a profile they are open (and still offered only with the candidate switch on).

## 4. Work by blocks

1. Candidate switch, the two seams, tests: switch off -> 2000 rolls never offer a candidate, a run of 60 days never has a candidate rule, both fingerprints unchanged; switch on -> the candidates appear; `testRuleOrder` gives the listed rules on days 3 and 6.
2. Extra fish of a pull + Dynamite + Harpoon, data rows, generator, tests: reach, the pick rules of 3.3, pull size (Big Haul sees the extras), no bait used up, the sea factor and its floor, the factor resets the next day, the harpoon order and factors, both charms together, no blast at the day-end pulls, an empty pull, determinism (two runs of one seed give the same event log).
3. The two rule days, tests: no fish enters on a Poachers day, the sea refills the next day; the Pest factor per landed fish, the floor, the Gold Sunfish exempt, the info card shows the reduced weight; Storm Glass switches both off.
4. Bot measure with `core_runs` (add an option for the candidate switch and for a forced charm, reusing the BUILD STRENGTH seam): careful bot, default mode, 300 runs, days 4 to 12:
   - survival table with candidates off (must equal the 2.8.1 table);
   - Dynamite forced into the first shop vs no forced charm vs a strong known charm forced (for example Big Haul): survival by day, median points of a day, fish landed per day;
   - Harpoon the same;
   - a run where day 3 is Poachers and day 6 is Pest (seam) vs the default rule days: share of runs that pass day 3 and day 6. If Poachers passes day 3 in under half of the runs that pass it by default, also measure value 0.25 and report both; do not change the default number.
   The bot may not use the charms well; say what it does.

Extend the report after each block. If time runs short, the order of value is: block 1, Dynamite, the two rule days, Harpoon, the measure. Stop and report rather than deliver a half-working block.

## 5. Done means

- New tests pass, all old tests pass, both fingerprints unchanged with the default tables.
- `python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue` prints `11 ok, 0 fail, 0 skip`.
- The report holds: every interface addition with one line on how the view uses it (events, state fields, tables fields, enum values); what the view must set to show the candidates early; the measure tables; the exact English texts; anything you changed in section 3 and why; anything you could not verify; questions.

## 6. Verification commands

```
powershell -NoProfile -ExecutionPolicy Bypass -File F:\UE\EpicJam_core\Tools\core-test.ps1 *> <a log file in your scratch folder>
python -X utf8 F:\UE\EpicJam_core\Catch\CoreTests\gen_charm_rows.py --check
python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue
```

PowerShell 5.1 marks a native program as failed when you use `2>&1`; write the output to a file with `*>` and read the file. The editor build waits for the build of another worktree (`-WaitMutex`); that is normal. If the orchestrator writes "pause", stop every build and test at once until "resume". Do not kill a process you did not start. Use absolute paths in every command; never write into `F:\UE\EpicJam`, `F:\UE\EpicJam_ui` or `F:\UE\EpicJam_submit`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/modifiers-v2.data.json` (add the two charms and the `candidate` field only; every existing charm stays as it is), and your report file.
- Do not change `Catch/Source/Catch/**` (the view) beyond what a compile needs. If the view in this worktree stops compiling because of an addition (for example a `switch` over an enum), make the smallest fix and list it; the view work is another executor's.
- Do not change existing charm rules, prices, day targets, config files, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, `Docs/design/core-interface.md`, the check gate, permission settings.
- Do not invent rules beyond this brief. A point that needs a decision goes into the report as a question; go on with the rest. The theme rule of the jam stands: no director, cinema or "cut" wording in any text.
- All player-facing text is English. No Cyrillic under `Catch/` (the Russian texts live only in the data file under `Docs/design/`).
- No deleting or hiding of old work. No commits, no pushes.

## 8. Report

File: `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core29-risky-fishing.md`, in English. Create it right after reading, extend it after each block. You report to the orchestrator, not to the director. Final message to the orchestrator: at most 25 lines.
