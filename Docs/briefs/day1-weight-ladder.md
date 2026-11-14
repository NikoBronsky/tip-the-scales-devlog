# Brief: modifiers v2.1 — weight, points and the food ladder

Status: written on 12.11.2026 at 14:59 (owner's clock). Executor: the Opus design agent who wrote modifiers v2, effort high. Paper work: no code, no prototype, no Unreal.

## 1. Goal

The owner played the money probe and answered by voice in three messages. Voice recognition garbles some words. The owner's words (Russian, verbatim):

1. «у нас получается две валюты, доллары и монетки. Надо тогда сделать категорию наград. Да, у нас, грубо говоря, рыболовное соревнование. Вот, и у нас награда и должна быть таблица награды за каждый вид рыбки.»
2. «в Балатру там нет долларов и монет, там есть очки. Нам, возможно, тоже надо сделать категорию, что вот за такой-то вес такие-то очки. И у рыбы будет вес, который дает больше очков. То есть мы не напрямую цену увеличим, мы увеличиваем вес рыбы. Чем больше веса, тем больше она очков приносит.»
3. «Рыбки не должны все ловиться на кильку. Это должен быть какой-то модификатор. То есть у нас должно быть рыбка 2 должна ловиться на рыбку 1, рыбка 3 должна ловиться на рыбку 2, рыбка 4 должна ловиться на рыбку 3 и так далее. И с модификаторами мы можем что-то сделать, чтобы рыба побольше ловилась на рыбу поменьше там, и в таком духе. [...] пока что у нас вся рыба очень крупная может как будто бы на кильку запрыгнуть.»
4. «положить весы какие-то, которые бы показывали, какая сторона перевешивает. [...] слева будет вес в очках, а справа будет вес, который нужно достичь, чтобы пройти уровень.»

Convert the modifier data and the proposal to these directions, so that the next browser probe can show them to the owner today.

## 2. What exists (verified by the orchestrator)

- Your files: `Docs/design/modifiers-v2.data.json` and `Docs/design/modifiers-v2-proposal.md`. First 16 hex digits of SHA-256 now: data 28c143db625bd4af, proposal 2b829c094d61c839.
- A frozen copy of both lies in the prototype folder of the scratchpad. Another executor builds the probe from it right now. Do not touch that folder.
- The owner's words are recorded as three decisions in `Docs/decisions/game-frame.answers.json`: `score-in-points-by-weight`, `scales-catch-against-norm`, `food-ladder-next-size`. Read them: each carries the orchestrator's reading in its comment.
- `Docs/design/rules-reference.md`: 2.2 is the fish table with the Tier column; 2.3 gives the depth band of every kind; 5.6 is the hunter rule; 7 is the price order.
- Facts from those sections:
  - Tiers: Sprat 1, Lanternfish 1; Mackerel 2, Hatchetfish 2, Snailfish 2; Sea Bass 3, Cod 3, Flounder 3; Tuna 4, Anglerfish 4; Oarfish 5, Gulper Eel 5; Giant Squid 6, Coelacanth 6.
  - Hunter kinds now: Sea Bass 3, Tuna 4, Anglerfish 4, Gulper Eel 5, Giant Squid 6. Today any hunter of a greater tier may eat the hooked fish (5.6). That is what the owner saw and rejects.
  - With "exactly one tier greater" and only these five hunters, nothing eats a Sprat or a Lanternfish (no tier-2 hunter), and nothing eats a Hatchetfish in the Midnight zone (tier 2 there, the hunter there is tier 4).
  - Bands (2.3): Sprat and Mackerel 24–278; Sea Bass 150–500; Lanternfish and Cod 324–678; Tuna 150–950; Hatchetfish 724–1178; Anglerfish 500–1500; Oarfish 1224–1778; Gulper Eel 950–2150; Snailfish 1824–2478; Giant Squid 1500–2950; Flounder on the floor.

## 3. Orchestrator's decisions (revocable: change one if it breaks something, and say so)

1. **Units.** The catch of a day is scored in points. There are no dollars. Coins stay the only money and do not change. A fish has a weight in kilograms. A kind has an award: points per kilogram. Points of a fish = weight x award, rounded halves up.
2. **Numbers.** The table weight of a kind is its present number in `fishPrices` (Sprat 2 ... Coelacanth 400), read as kilograms. The award of every kind starts at 1. A fish card of level L makes the award L. This is the present "table price x card level".
3. **What raises what.** Every charm effect that changed the price now changes the weight: `flat` is kilograms, `percent` and `mult` scale the weight, the chain factor scales the weight (a hunter that has eaten is heavier), the Golden Grub and Giants double the weight. New combine order: weight = (table weight + flat sum) x (1 + percent sum / 100) x multipliers x Golden Grub x chain factor; points = weight x award, rounded halves up. One arithmetic change follows: the flat sum is now multiplied by the card level. The day rule "under $10 pays nothing" becomes "a fish lighter than 10 kg gives no points" (the final weight). The Gold Sunfish gives half of the day's norm in points and no charm changes it, as now. A Bait Knife Sprat gives 0 points.
4. If "heavier" reads wrong for a charm and "more points per kilogram" reads right, you may put its effect on the award. Then add the effect parameter `"on": "award"` and name the charm in the report. Keep this rare: the owner's words are «мы увеличиваем вес рыбы».
5. **The fee** becomes the day's norm, in points. The three fee curves keep their numbers.
6. **Size** is the Tier column, 1 to 6. **The ladder** (the owner's rule): a hooked fish of size N may be eaten only by a free fish of size N + 1. Sizes N + 2 and greater do not take it.
7. **Who eats** is your proposal. The owner's words need "fish 2 takes fish 1", so a size-2 kind must be able to take a hooked Sprat. Default reading: any kind may eat a hooked fish exactly one size smaller when the other conditions of 5.6 hold (open zone, hook depth inside its band widened by 30, reach along x). Hunter kinds keep their wide bands; other kinds have their home band. If a zone has no ladder, you may change the size of a kind. Give the new table and the reason. Do not add or remove kinds.
8. **Charms widen the ladder.** Propose 2 to 4 charms, new or reworked from the present chain line. Each needs ground, as every v2 charm. The set must hold: one charm that lets a fish take a hooked fish two sizes smaller; one charm that brings back "any larger fish takes" under a hard condition (the owner: «Это должен быть какой-то модификатор»). Use at most two new primitives and define each in one line (for example a rule value `eat_reach`). The total stays 50. If you add charms, say which leave and why. Keep the class and rarity counts close to the present ones.
9. **Live bait.** Say how the Live Sprat bait sits on the ladder (a size-1 fish on the hook, or a bait with its own accept list) and why.
10. **The scales** are a display matter. They need nothing from the data.
11. **Texts.** Every `ruleRu` and `ruleEn` speaks in weight, points and coins. No "$", no «стоит», «дороже», "worth", "price". One term per thing. Russian: «вес», «кг», «очки», «очки за кг», «норма дня», «монеты», «размер». English: weight, kg, points, points per kg, target, coins, size. The limits stay: name at most 14 characters, rule at most 90.

## 4. Work

1. Data file, version "2.1": add `units`; rename `fishPrices` to `fishWeights` (keep `fishPricesOld`); add `sizes` and the starting `awards` per kind; add a `ladder` block (the rule, which kinds eat, the widening values); rewrite `combineOrder`; update `primitives`; rewrite the rule texts of all 50 charms; add or rework the chain charms; reword `dayRules`; update `builds` where the chain line changed.
2. Proposal: add a note at the top: units changed on 12.11 at 15:00, sections 1–11 speak of dollars and prices, read "price" as "weight". Do not delete old sections. Add section 12 "Weight, points and the ladder" with:
   - the ladder table: for each zone, which chains can form at a typical hook depth by default, the longest default chain, and the longest chain with each widening charm;
   - the changed and the new charms, with the reason for each;
   - the Food Chain build arithmetic again for days 3, 6 and 9 against the Normal norm;
   - what changes in `rules-reference.md` 5.6 and 7 (a list; do not edit that file);
   - your questions for the orchestrator.
3. Run your own check script. Then run the orchestrator's: `python -X utf8 "C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\check_mod_v2.py"`. It must print `problems: 0`.

Time: aim at 30 minutes. If the ladder analysis runs long, hand in the data file first and say what is missing.

## 5. Done means

1. The JSON loads; 50 charms; every row uses only primitives that `primitives` lists; every `needs` and `combos` entry resolves.
2. No rule text holds "$", «стоит», «дороже», "worth" or "price". All texts are inside the length limits.
3. Every kind has a size. Every zone has at least one default chain, or section 12 says why not and what the player does there.
4. Section 12 exists with its five parts.
5. The prototype folder is untouched. No file other than the two design files changed, except your own scratch files.

## 6. Boundaries

- You do not invent the game beyond the owner's words and section 3. A thing that needs a director's decision goes to "questions for the orchestrator", and you carry on with the plainest reading.
- No new fish kinds, baits, upgrades, screens or money rules. Coins, the payout and the shop prices do not change.
- The theme rule stands: no director, no film, no play on the word "cut".
- No git. No commits.

## 7. Report

English, at most 40 lines: what changed in the data; the ladder in one small table; the new primitives; the charms added, reworked and removed; the Food Chain arithmetic; your questions; what is not done.

## 8. Chain of command

You report to the orchestrator. The orchestrator checks the data with its own script, runs a critique pass, and passes the data to the probe executor.
