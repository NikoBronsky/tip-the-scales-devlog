# Brief: the modifier probe, pass 2 — points by weight, the award table, the scales, the food ladder

Status: written on 12.11.2026 at 15:17 (owner's clock). Executor: the Opus agent of pass 1 (`day1-modifiers-probe.md`), effort high. Work folder: the prototype folder in the scratchpad, as in pass 1. The Unreal project is not touched.

## 1. Goal

The owner played the money probe and answered by voice. Voice recognition garbles some words. The owner's words (Russian, verbatim):

1. «у нас получается две валюты, доллары и монетки. [...] у нас, грубо говоря, рыболовное соревнование. Вот, и у нас награда и должна быть таблица награды за каждый вид рыбки.»
2. «в Балатру там нет долларов и монет, там есть очки. [...] у рыбы будет вес, который дает больше очков. То есть мы не напрямую цену увеличим, мы увеличиваем вес рыбы. Чем больше веса, тем больше она очков приносит.»
3. «Рыбки не должны все ловиться на кильку. Это должен быть какой-то модификатор. То есть у нас должно быть рыбка 2 должна ловиться на рыбку 1, рыбка 3 должна ловиться на рыбку 2, рыбка 4 должна ловиться на рыбку 3 и так далее. [...] пока что у нас вся рыба очень крупная может как будто бы на кильку запрыгнуть.»
4. «положить весы какие-то, которые бы показывали, какая сторона перевешивает. [...] слева будет вес в очках, а справа будет вес, который нужно достичь, чтобы пройти уровень.»

The owner then confirmed the orchestrator's reading: points of a fish = weight x the kind's points per kilogram («Да, так»).

This pass makes the probe show all of it, so that the owner can judge today. It is a decision aid. Nothing from it goes into the Unreal game.

## 2. What exists

- Your page after pass 1: `catch-modifiers.html` with the row evaluator, `embed_mod_data.py`, `preview-modifiers.html`, `modifiers-probe-report.md`.
- New in the folder: `modifiers-v2.1.data.json` (SHA-256 starts 2fb580cad7c2057d) and `modifiers-v2.1-proposal.md` (17f8da85a7f5cd87). They are copies of the repository files. The v2 copies stay in the folder, unchanged.
- What v2.1 changes in the data:
  - `units`: weight (kg), award (points per kg), points, norm, coins, size.
  - `fishWeights` replaces `fishPrices` (same numbers, read as kg). `awards`: 1 for every kind. `sizes`: 1 to 6 per kind (Lanternfish is 2, Hatchetfish is 3).
  - `combineOrder`: nine steps. Weight = (table weight + flat) x (1 + percent) x multipliers x Golden Grub x chain factor; day rule 4 tests the final weight; award = card level x multipliers with `"on": "award"`; points = weight x award, halves up.
  - `ladder`: the rule, who eats, the bands, the eat factor, the default reach, the widening charms, Live Sprat, Giants.
  - `primitiveChanges`: one new effect `eat_reach`; renamed `fish.priceMin/priceMax` to `weightMin/weightMax` and `rate.what = hunter` to `eat`; the focus of conditions inside the eat check.
  - Charms: still 50. Bait Hoard and Feeding Time left. Big Mouth and Free Feast are new. Apex Hunter, Live Well, Fresh Scent and Small Fry are reworked. Blue Ribbon is the only charm on the award. Every rule text is reworded: no dollars.
  - `dayRules`: rule 4 is "Fish lighter than 10 kg give no points".
- Read in `modifiers-v2.1-proposal.md`: 12.1 (units and order), 12.2 (sizes and the ladder), 12.3 (changed and new charms), 12.5 (rule changes) and 13 (the orchestrator's answers).
- In `catch-economy.html` the catch point is the bucket on the raft at x 250 while the raft is in view, and a small icon at the money panel (x 1065, y 66) while the camera is deep (the function near line 1510 returns `raft: true` or `false`). The bucket is a damped spring (near line 1519).

## 3. Technical decisions (orchestrator; revocable — if one breaks the page, change it and say so)

1. **Data source.** `embed_mod_data.py` reads `modifiers-v2.1.data.json`. The embedding stays stable when run twice.
2. **Scoring** follows `MOD.combineOrder` steps 1 to 9. One function returns the weight, the award and the points of a fish. `CatchProto.price(kind, opts)` stays and returns the points. New: `CatchProto.weigh(kind, opts)` returns `{ weight, award, points }`. Neither changes any state.
3. **No dollars.** No "$" is drawn anywhere. Coins keep their look. Two format helpers: weight with one decimal and a comma, without ",0" («3,5 кг», «12 кг»); points as a whole number with «оч.» in tight places and the full word with the right Russian ending in panels (1 очко, 2 очка, 5 очков).
4. **Words** (Russian; one term per thing): «очки», «вес», «кг», «очки за кг», «норма дня» (the old «плата»), «монеты», «размер», «таблица наград». The top panel shows «УЛОВ: N рыб, X очков» and «норма дня: Y очков». The day summary, the shop line about tomorrow, the run end and the hints use the same words. Run end: «Улов не набрал норму дня. Забег окончен.»
5. **The scales** replace the bar "catch against the fee".
   - A two-pan balance stands on the raft where the bucket stands. The bucket sits on the left pan and keeps its spring, squash and water. The right pan carries a weight with the norm of the day written on it. The catch points are written under the left pan.
   - Beam angle from r = catch points / norm. r from 0 to 1: the weight side is down, angle = 10 degrees x (1 - r). r = 1: level. r above 1: the catch side is down, angle = 10 degrees x min(1, (r - 1) / 0.5).
   - The beam is a damped spring: it overshoots and settles in about 0.6 s. Every fish that lands in the bucket kicks it. When r crosses 1 for the first time in a day: one larger swing, a flash, and a short rising chime from the existing sound code.
   - In the prep phase the weight of the new day drops onto the pan.
   - While the raft is out of view (the catch point is the panel icon), draw a small pair of scales in the top panel with the same angle and the two numbers.
   - The scales do not hide a rod. If the left end is crowded, move the bucket and the scales, not the rods.
6. **The award table.** A button «Таблица наград» in the prep phase, in the day summary and in the shop. It opens a panel with one row per kind of the open zones, in `FISH_ORDER` order: the fish drawing, the name, «размер N», the table weight, the award («K оч. за кг», marked when above 1), the chum level when above 0, the points of a plain fish now, and the line «берёт на крючке: ...» with the kinds it may eat now (or «—»). The Gold Sunfish row says «половина нормы дня». A foot line says how many kinds live in locked zones. Text must fit; the panel may scroll.
7. **The fish hover card** shows the weight, the award, the points now, the size, «берёт на крючке: ...» and «её берут: ...».
8. **The ladder.** One function decides "may this free fish eat the main fish of that hook":
   - Every kind except the Gold Sunfish may eat. A hooked Gold Sunfish is never eaten.
   - Size gap = size of the eater - size of the hooked main fish, sizes from `MOD.sizes`. Allowed when 1 <= gap <= reach.
   - Reach = `MOD.ladder.defaultReach`, or the largest `eat_reach` value whose row holds. Inside this check the conditions `fish` and `ground_level` test the eater; `chain`, `hook_bait`, `hook_index` and `catch_zone` test the hook (`MOD.primitiveChanges.eatReachFocus`).
   - The other conditions of the present hunter check stay: open zone, hook depth inside the band widened by 30, reach along x, nearest first, the check every 0.6 s, chance 0.28 x eat factor. Hunter kinds keep their wide bands; other kinds use their home band. `ECON.hunterDay` stays: before that day nothing eats.
   - Live Sprat bait: a kind bites a Live Sprat hook only if it may eat a size-1 fish at that hook, by the same function (`MOD.ladder.liveSprat`).
   - The rod card warning that a hunter is near now shows for any eater.
9. **Day rule 4** tests the final weight.
10. **Run log**: no dollars. Per day: the norm, the catch in points, the coins, the heaviest fish (kind and kg), fish by kind, chains, charms with counters.
11. **The "what is new" box** (replace the pass-1 list):
    - «Улов считается в очках. Долларов нет. Монеты остались деньгами магазина.»
    - «У рыбы есть вес. Очки рыбы = вес × очки вида за килограмм.»
    - «Модификаторы делают рыбу тяжелее. Карточка рыбы поднимает очки вида за килограмм.»
    - «Весы на плоту показывают улов против нормы дня. Улов должен перевесить гирю.»
    - «Кнопка „Таблица наград“ показывает вес, размер и очки каждого вида.»
    - «Рыбу на крючке берёт только рыба на один размер крупнее. Кильку берёт скумбрия, скумбрию берёт морской окунь.»
    - «Модификаторы „Большая пасть“ и „Общий пир“ расширяют это правило.»
    - «В магазине 50 новых модификаторов. Модификатор работает, когда вы подготовили для него почву.»
    - «Строка „нужно“ на карточке называет почву.»
    - «Журнал забега пишет вес, очки, цепочки и счётчики.»
    Probe label: «Проба: очки по весу, весы, лестница рыбы, 50 модификаторов.»
12. **Self-test.** The price group moves to the v2.1 order: at least 12 cases with hand arithmetic, among them a flat inside a card level, Blue Ribbon on the award, a chain with and without Food Chain, rounding at .5. A new ladder group, at least 8 cases at the level of the function of item 8: Sprat hooked: Mackerel yes, Sea Bass no, Tuna no; Mackerel hooked: Sea Bass yes; Big Mouth with a chummed Sea Bass: Sea Bass takes a Sprat; Big Mouth with no chum: no; Free Feast with a Rare bait on the hook: Giant Squid takes a Sprat; a hooked Gold Sunfish: nobody; Live Sprat hook: Mackerel bites, Tuna does not. The "no ground, no pay" group also proves that the reach stays at the default for every charm with `"unconditional": false`.
13. Money does not change: `ECON`, the norm curves (the old fee numbers), the payout, coin prices, chum, fish cards.

## 4. Done means

1. The page loads with no error in your Node run; a run plays through at least three days with a chain of two links seen (Sprat, Mackerel, Sea Bass) and a charm bought and sold.
2. `CatchProto.selfTest().ok` is true, with the rule of pass 1 for a "no ground, no pay" failure that the data causes.
3. `node --check` passes. `python embed_mod_data.py` twice leaves the page byte-identical.
4. No dollar sign reaches the screen: give the search you used.
5. No run of `??` where Russian text belongs.
6. Unchanged byte for byte: the seven files of pass 1, and the two v2.1 copies.

## 5. Boundaries

As in pass 1. You do not change the JSON; a row you cannot implement as written goes to "data questions" with the plainest reading. No new charm, number, rule or screen beyond this brief. Keep UTF-8. No browser, no git, no network, no publishing.

## 6. Report

Add a part "Pass 2" to `modifiers-probe-report.md`, English, at most 40 lines: what changed; the price cases and the ladder cases with hand arithmetic; data questions; design findings; what your Node run cannot prove (the look of the scales and of the award table above all).

## 7. Chain of command

You report to the orchestrator. The orchestrator loads the page in a browser, checks weights and points by hand through `CatchProto.weigh`, looks at the scales and the award table, plays three days, and gives the page to the owner.
