# Addendum to the brief core 2.5: three rare rod charms

Written by the orchestrator on 14.11.2026. It extends `Docs/briefs/day3-core25-fish-reward.md`. Same worktree, same boundaries, same report file (add a section "Rod charms").

## 1. Goal

The owner wants a little madness: rare charms that let the player build wild rods. Owner, decision `crazy-rod-charms-idea`, 14.11: «нам нужен какой-то модификатор, который позволит увеличивать количество крючков в длину, что позволит игроку делать какие-то безумные удочки, супер длинные... я просто хочу добавлять немножко безумия, чтобы игрок иногда мог делать какие-то модификаторы, которые ему, например, позволят снять ограничения с максимума прокачки удочки или там давать максимум +2 уровня там, или +3, какие-то такие редкие.»

The orchestrator proposed three charms with numbers. The owner answered «Да, все три» (decision `crazy-rod-charms-approved`, 14.11).

## 2. The three charms (the numbers are the owner's; do not change them)

| Id | Name | Class | Rarity | Rule text (English, player-facing) | Russian name | Russian rule |
| --- | --- | --- | --- | --- | --- | --- |
| `trotline` | Trotline | Rod | Legendary | Every rod has 2 more hooks. | Перемёт | Каждая удочка получает +2 крючка. |
| `extra_edge` | Extra Edge | Rod | Rare | Rods can be sharpened 3 levels past the limit. | Сверх предела | Удочку можно заточить на 3 уровня выше предела. |
| `bottom_rig` | Bottom Rig | Rod | Rare | The lowest hook of each rod can go one zone deeper than your line. | Донная снасть | Нижний крючок каждой удочки достаёт на одну зону глубже открытой. |

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Add the charms the way the 53 existing charms are added (the data file, the generated rows, the fit filter group, the price by rarity, the weightless variant). Verify how before you rely on it.
2. Trotline. The two extra hooks hang below the rod's own hooks at the same spacing, so the rig gets longer. They obey the depth limit like every hook. When the charm leaves the shelf, the extra hooks go away at once: a fish on them escapes by the normal escape path and their bait returns to the stock.
3. Extra Edge. The rod level limit is the end of the sharpen price list (`Private/Shop.cpp`, the level price lookup near line 51). With the charm the limit is 3 higher. Each extra level gives the same per-level effects as the levels below it. Extend the price list by the same growth as its last steps. Hook levels stay 3 and 5 (`Public/CatchCore/Tables.h:210`). When the charm leaves the shelf, a rod keeps the levels it has; it cannot be sharpened further.
4. Bottom Rig. Hooks reach only the open zones today (`Private/Rods.cpp:50-53`). With the charm the lowest hook of each rod may enter the next zone below the last open one. That zone holds its fish while the charm is on the shelf, as if it were open, for that hook only. With all zones open the charm does nothing. When the charm leaves the shelf, a hook in that zone rises to the allowed depth.
5. Tell the view what it needs as data: the number of hooks of a rod and the allowed depth per hook must come from the core state, not from a view formula. List the fields in "Interface changes".
6. Tests for each charm: the effect on, the effect off after a sale, determinism with the charm in the pool.
7. Order: add the charms before the refit of the targets (block 6 of the brief), so the bot runs see them in the pool. Report the pass rates with and without the three charms in the pool.
8. If a rule here clashes with an existing charm or rule, do not invent a fix: report it as a question and finish the rest.
