# Addendum 2 to core 2.9: the shop offers a chain-chance charm after a day with a chain (core 2.9.2)

Written by the orchestrator on 14.11.2026, 10:58. Same executor, same worktree `F:\UE\EpicJam_core`. Do it AFTER the balance pass 2.9.1 is complete (tests pass, gate green, report section written). Everything in the brief `day3-core29-risky-fishing.md` still holds (boundaries, commands, report rules).

## 1. Why

The game director played many runs and said: «мне до сих пор ни разу не попалась карточка увеличивающая вероятность цепочек». The orchestrator found the cause in `Shop.cpp` (`Game::RollCharmOffers`) and `Offers.cpp` (`CharmFitGroups`, `NeedFits`): the charms that raise the chance of a chain are few, three of them need chum and one needs rare bait, and the two left come up in about one shop in nine. The orchestrator asked: «После дня с цепочкой магазин должен обязательно предлагать модификатор цепочек?» The director answered: «сделай». This is a recorded owner decision (`chain-charm-guarantee`). It changes the default game.

The feeling the director wants: the player sees a chain happen (a hunter eats the hooked fish and is caught itself), and the very next shop shows a card that builds on it.

## 2. The rule

After a day in which the player landed at least one chain, the first roll of the next shop holds one chain-chance charm among its charm offers.

Decisions of the orchestrator (each is revocable; the executor may change one if it breaks something and must say so):

1. **A day with a chain** = a day in which at least one fish was landed with a chain of 1 or more (the same definition as the profile count `ChainsLanded`, `Tables.h:193`). Keep one flag in the state, set at that landing, cleared when the shop after that day has made its first roll. A chain link whose fish escaped before it was landed does not count.
2. **A chain-chance charm** = a charm whose effect raises the chance or the reach of a hunter eating a hooked fish (it contributes to `EatFactor` or `EatReach`, `Game.h:530,543`). The orchestrator's list from the shop investigation: `fresh_scent`, `thin_line`, `big_mouth`, `free_feast`, `tiny_terror`. Check it against the charm data and report the final list. Mark these charms by data (a flag in the charm data file and in `CharmData`), not by ids in code.
3. **Which one.** Take the chain-chance charms that the normal roll could offer at all (in the shop, not on the shelf, open, in the game, not reserved by a pending reward, the Tide Pool rule). Among them keep only the charms of the best fit group found among THEM (not the best group of the whole shop), and never a charm of the group "does not fit" - a card that cannot work for this tackle is worse than no card. Draw a rarity by the usual rarity weights among those left, then a charm of it. Nothing left: no guarantee, the roll is the normal one.
4. **Where.** Only the shop roll (`RollCharms`), never the reward roll. The guaranteed charm takes the first offer slot; it is not an extra offer. Price and the weightless draw follow the usual rules of an offer. The test seam `testOfferOrder` comes first when it is set; then the guaranteed charm takes the next slot.
5. **A reroll** is a normal roll: the guarantee is used up by the first roll of that shop.
6. **Data switch.** `ShopData::chainOffer` (bool, default true). With the switch off the game is exactly core 2.9.1: both old fingerprints must come out (`b0d881d44f065645`, `7d632f58e4dd608e`). Record the two new fingerprints of the default tables.
7. No new event and no new text. The view needs no change. (If you see that the view must learn about the guaranteed offer, write it into the report as a question; do not add interface for it.)
8. Version: core 2.9.2, API stays 9 (additions only).

## 3. Measure

- The careful bot, 300 runs, both modes: the survival table with the switch on and off (share alive after day 6, 9, 12), and how often a run sees at least one chain-chance charm offered by day 6 with the switch on and off.
- If the bot lands no chains, say so and measure the offer share with a script that lands a chain on day 2 through the existing test seams.
- If survival moves by more than 5 points, say so plainly; do not retune anything in this pass.

## 4. Done means

- Tests: a day with a landed chain puts a chain-chance charm into the first slot of the next shop's first roll; a day without a chain does not; a reroll is normal; every chain-chance charm on the shelf means no guarantee; a charm of the group "does not fit" is never forced; the reward roll is untouched; the switch off gives the two old fingerprints.
- All tests pass. `python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue` prints `11 ok, 0 fail, 0 skip`.
- The report file gets a section "2.9.2 chain offer": the final list of chain-chance charms, where the code changed, the two new fingerprints, the measure tables, what you could not verify.
- Keep the report sections of 2.9.1 and 2.9.2 apart, so the orchestrator can check each on its own.

Final message to the orchestrator: at most 20 lines for both passes together. No commits, no pushes.
