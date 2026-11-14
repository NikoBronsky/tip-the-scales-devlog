# Addendum 1 to brief day3-core210-link-charms: a blast takes a chum level for good (14.11.2026 15:03)

Executor: the same Opus executor, effort high. Everything else in the brief holds. You do not commit.

## 1. The director's decision

Asked for how long a Dynamite blast takes one level of the fish-amount upgrade ("до конца дня" or "насовсем"), he answered: "Насовсем". Before that: "Ну у нас же есть динамит? Он должнен глушить и собирать рыбу но уменьшать ее количество в море на (1 даунгрейд)" and, about the reverse one: "Динамит наоборот - прибавляет рыбу в море, но забирает улов из ведра (не весь)".

So the sea cost of a blast is no longer a day factor. A blast takes chum levels, and the reverse dynamite gives them. This replaces the cost of decision 2 and of the 2.9 Dynamite, and the gain of decision 5 of the brief.

## 2. Technical decisions (orchestrator, temporary balance, each revocable; all numbers are data)

1. **The cost of a blast** (Dynamite at a pull, Lucky Fuse on a lucky chance; one shared function). For every fish kind the blast took at least one fish of: the owned chum level `State::chumLevels[kind]` and today's active level `State::chumActive[kind]` both drop by `blastChumCost` = 1, at once, for the rest of the run. The player can buy the level again in the shop at the normal price for the level held.
2. **A blast takes only fish that have something to lose.** A free fish is in reach of a blast only if its kind has an owned chum level of 1 or more at that moment. A kind at level 0 is not blasted and pays nothing. (Reason: otherwise a player without chum blasts for free and the charm has no price.) All other pick rules of 2.9 stay.
3. The day factor of 2.9 goes away for blasts: `dynamiteSeaFactor` and `dynamiteSeaFloor` are no longer applied by a blast (keep `State::seaFactor` only if something else still uses it; say what you did). The weight factor of blasted fish (35 %) and the radius stay.
4. **Dynamite rule text** (data file): `A pull that lands a fish blasts the water: every chummed fish near this rod's hooks lands too, at 35% weight. Each kind it takes loses 1 chum level for good.` Russian: `Рывок с рыбой даёт взрыв: вся рыба прикормленных видов рядом с крючками этой удочки тоже попадает в улов, с весом 35%. Каждый такой вид навсегда теряет 1 уровень прикормки.`
   **Lucky Fuse rule text**: `When a lucky chance comes true, the water blasts at that rod: every chummed fish near its hooks lands too, at 35% weight. Each kind it takes loses 1 chum level for good.` Russian: `Когда удача сбывается, у этой удочки гремит взрыв: вся рыба прикормленных видов рядом с её крючками тоже попадает в улов, с весом 35%. Каждый такой вид навсегда теряет 1 уровень прикормки.`
5. **Bait Bomb, the reverse** (replaces the gain of decision 5; the trigger and the take stay: a pull of `baitBombPull` = 3 or more fish throws `baitBombTake` = 2 fish from the bucket back).
   - Only fish of kinds that can hold chum and are below `ShopData::chumMaxLevel` are thrown (same eligibility as the shop's chum pool, `Shop.cpp` near 282). Fewer than 2 such fish in the bucket: it throws what there is; none: it does nothing.
   - Each thrown fish adds 1 to a run-long counter of its kind. Every `baitBombFishPerLevel` = 5 thrown fish of a kind give that kind +1 owned chum level and +1 active level at once, for the rest of the run, not above the maximum. When a level is gained, up to `baitBombEnter` = 8 fish of that kind enter from the edges at once, not above its new wanted count.
   - No sea factor and no ceiling: drop `baitBombSeaFactor` and `seaCeil`.
   - Rule text: `A pull of 3 or more fish throws 2 fish from your bucket back into the sea. Every 5 fish of a kind thrown back give that kind +1 chum level for good.` Russian: `Рывок на 3 рыбы и больше бросает 2 рыбы из ведра обратно в море. Каждые 5 брошенных рыб одного вида навсегда дают этому виду +1 уровень прикормки.`
   - Event `BaitBomb`: rod, `otherFishIds` = the thrown fish, plus the kinds that gained a level and how many fish entered (use the event fields the way the other events do; say which).
6. Shockwave is unchanged: a blast that took at least one fish, and a Bait Bomb throw of at least one fish, each lift the zones by one for the day.
7. Events for the view: a chum level lost or gained outside the shop must be visible. Emit one event per kind (`ChumChanged`: kind, the level after, `value` = the change, the source charm) after the `Blast` or `BaitBomb` event.
8. Charms that read chum levels (Well Fed, Sprat Fever, Chum Bucket, Cluster Hook, Big Shoals, Pure Waters, Coral Garden, Cold Spring, Bare Steel, Groundbait and others) simply see the new levels. If a charm reads a level that was fixed when the day was prepared, say which and what happens now; do not redesign it.

## 3. Tests to add

A blast with no chummed kind in reach takes nothing and costs nothing; a blast over two chummed kinds lowers both by one and not below 0; the level stays down on the next day and can be bought again; today's wanted count drops at once; the Lucky Fuse blast pays the same; five thrown fish of one kind give one level, the counter survives the day end, the maximum holds; a full loop Bait Bomb + Dynamite on one shelf stays deterministic; with the candidate switch off both fingerprints still equal core 2.9.3.

## 4. Measure (replaces the Dynamite and Bait Bomb lines of block E)

Careful bot, default mode, candidates on, 300 runs, 12 days, alive after day 12: Dynamite forced; Dynamite + Bait Bomb forced; Lucky Bite + Lucky Fuse forced. Say whether the bot buys chum at all: if it does not, Dynamite does nothing for it and the number only shows that. Then propose, do not tune.

## 5. If you already built block D or the measures

Rework the cost and the gain as above, rerun the tests, and rerun only the measures that the change touches. Say in the report what was built before this addendum and what changed.
