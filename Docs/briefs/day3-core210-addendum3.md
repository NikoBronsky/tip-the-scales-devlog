# Addendum 3 to brief day3-core210-link-charms: a charge source for each event, chances, a charm that adds to every chance (14.11.2026 15:14)

Executor: the same Opus executor, effort high. The brief and addenda 1 and 2 hold except where this file says otherwise. You do not commit.

## 1. The director's words

The orchestrator listed the events the core knows and asked which of them should throw a charge. He answered: "Да сделай для каждого модификатор, пусть игрок попробует собрать. Сам бросать игрок не будет. Возможно для баланса просто дать шанс (типа 1 из 5 или 1 из 1000), потом отдельный модификатор на +1 к шансам (было 1 из 5 стало 2 из 5) и т.п."

The orchestrator's reading: every event of the fishing day gets its own charm that throws a charge of dynamite; the player collects them. The player never throws a charge by hand. Each source has a chance "1 in N", and one more charm adds +1 to every chance.

## 2. Technical decisions (orchestrator, temporary balance, each revocable; every number is data)

1. **No hand throw.** No command, button or input lets the player throw a charge.
2. **Charge sources.** Each is a candidate charm of class `rod` that throws the charge of addendum 2 from the rod of its event; each works alone. The chance is a `Chance` condition on its row, so a success is a lucky chance like Lucky Bite's (event `LuckyChance`, the `OnLucky` rows fire).

   | Kind | id | Rarity | Trigger | Chance | English rule text |
   | --- | --- | --- | --- | --- | --- |
   | `Dynamite` (exists) | `dynamite` | rare | a pull of the player or Auto-Reel that lands a fish | 1 in 5 | `A pull that lands a fish: 1 chance in 5 that the rod throws a charge of dynamite.` |
   | `PowderKeg` | `powder_keg` | uncommon | each hooked fish that lands (not a fish landed by a blast, a harpoon or the Bait Bomb) | 1 in 25 | `Each hooked fish you land: 1 chance in 25 that the rod throws a charge of dynamite.` |
   | `SoreLoser` | `sore_loser` | uncommon | a fish escapes from a hook | 1 in 3 | `When a fish escapes: 1 chance in 3 that the rod throws a charge of dynamite.` |
   | `ChainReaction` | `chain_reaction` | uncommon | each new link of a chain | 1 in 6 | `Each new link of a chain: 1 chance in 6 that the rod throws a charge of dynamite.` |
   | `LuckyFuse` (addendum 2) | `lucky_fuse` | rare | a lucky chance comes true (any, also the chances above) | none | `When a lucky chance comes true, that rod throws a charge of dynamite.` |

   Russian names and rules for the data file: Dynamite `Динамит` / `Рывок с рыбой: 1 шанс из 5, что удочка бросает заряд динамита.`; Powder Keg `Пороховая бочка` / `Каждая рыба с крючка в улове: 1 шанс из 25, что удочка бросает заряд динамита.`; Sore Loser `Обидчивый рыбак` / `Когда рыба срывается: 1 шанс из 3, что удочка бросает заряд динамита.`; Chain Reaction `Цепная реакция` / `Каждое новое звено цепочки: 1 шанс из 6, что удочка бросает заряд динамита.`; Lucky Fuse as in addendum 2.
   - What a charge does is the same for every source (addendum 2). Put one glossary text into the report for the view (the cards underline terms): `Charge of dynamite: it bounces off the walls and blasts after a random time. Fish near the blast land too, at 35% weight. Each kind it takes loses 1 chum level for good.` The Dynamite rule text of addendum 2 is replaced by the short one above.
   - The events "day end" and "a purchase in the shop" get no source: no fishing runs at those moments. Say so in the report; do not build them.
   - Loop guard: a charge thrown by an `OnLucky` row rolls no chance and fires no `OnLucky`; fish landed by a blast, a harpoon or the Bait Bomb roll no chance and fire no source. One event can throw at most one charge per source charm.
3. **Horseshoe** (kind `Horseshoe`, id `horseshoe`, class `fish`, rarity `rare`, candidate). Rule text: `Every chance gets +1: 1 in 5 becomes 2 in 5.` Russian: name `Подкова`, rule `Каждый шанс получает +1: было 1 из 5, стало 2 из 5.`
   - A passive effect (`ChanceBonus`, value 1): wherever a `Chance` condition rolls, its first number is raised by the sum of the bonuses on the shelf, not above its second number. It applies to every chance in the game: Lucky Bite and all sources above.
   - The rule texts of the chance charms should show the chance that holds now. If the data already has a live-number mechanism that fits (`{n}`), use it for the first number; if not, give the view a way to read the effective chance of a charm and say which.
4. **Offer tags.** `lucky_charm` = a charm with a `Chance` condition is on the shelf (today only Lucky Bite; with the candidate switch on also the sources above). With the switch off the tag must behave exactly as in core 2.9.3. Fish Tale, Lucky Fuse and Horseshoe use it. `blast_charm` (Shockwave) = any charge source or the Bait Bomb is on the shelf.
5. Chain Shot (the harpoon on a chain link), Shockwave and the Bait Bomb stay as they are in the brief and the addenda.

## 3. Tests to add

Each source throws only on its event and only on a success; the share of successes over 10,000 rolls is near the chance; Horseshoe turns 1 in 5 into 2 in 5 and 1 in 3 into 2 in 3 and never passes N in N; Lucky Fuse throws on a success of every source and of Lucky Bite, once per success; no recursion with all sources, Lucky Bite, Lucky Fuse, Fish Tale and Horseshoe on one shelf; fish landed by a blast roll nothing; offers: Horseshoe and Lucky Fuse only with a chance charm on the shelf; candidate switch off: both fingerprints equal core 2.9.3 and Fish Tale is offered exactly as before.

## 4. Measure (replaces the lists of the brief and of addendum 1; still at most three processes at once, below-normal priority)

Careful bot, default mode, candidates on, 300 runs, 12 days, alive after day 12:
- Dynamite forced; Dynamite + Horseshoe forced;
- Lucky Bite + Lucky Fuse forced; Chain Shot forced; Dynamite + Shockwave forced; Dynamite + Bait Bomb forced;
- stress: all five sources, Horseshoe, Lucky Bite, Fish Tale, Chain Shot, Shockwave, Bait Bomb, Harpoon on a widened shelf: no crash, time per run, charges thrown per day, share of blasts that take no fish, kinds erased by day 12.
Report against the band 27.0-52.0 %. Propose numbers; do not tune.

## 5. Time

About 20 minutes more. The order of work if time runs short: the charge and its price (addendum 2) first, then the sources and Horseshoe (this file), then Shockwave and the Bait Bomb, then the measures. Report what is built and what is not.
