# Addendum 2 to the brief core 2.5: three silly rare charms

Written by the orchestrator on 14.11.2026. It extends `Docs/briefs/day3-core25-fish-reward.md` and `day3-core25-addendum-rod-charms.md`. Same worktree, same boundaries, same report file (add a section "Silly charms").

## 1. Goal

Owner, 14.11: «подумай, может, ты что-то такое же безумное придумаешь, чтобы выглядело немножко даже глупо. Просто надо сделать так, чтобы это было достаточно редко, чтобы игрок мог чувствовать это как достижение.» The orchestrator proposed three charms; the owner answered «Да, все три» (decision `silly-charms-approved`, 14.11).

## 2. The three charms (the rules and numbers are the owner's; do not change them)

| Id | Name | Class | Rarity | Rule text (English, player-facing) | Russian name | Russian rule |
| --- | --- | --- | --- | --- | --- | --- |
| `upside_down` | Upside Down | Sea | Legendary | The zones swap places: deep fish swim near the surface, surface fish go deep. | Вверх дном | Зоны меняются местами: глубинная рыба плавает у поверхности, а рыба поверхности уходит на глубину. |
| `tiny_terror` | Tiny Terror | Fish | Legendary | Any fish of another kind may eat the hooked fish, even a smaller one. | Мелкая, но злая | Любая рыба другого вида может съесть рыбу на крючке, даже более мелкая. |
| `whopper` | Whopper | Fish | Rare | Once a day a fish ten times larger enters the sea. It weighs x10. | Рыба-гора | Раз в день в море приходит одна рыба в десять раз крупнее. Она весит x10. |

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Add them the same way as the three rod charms: a data row plus its rule, the fit filter group, the price by rarity, the weightless variant.
2. Upside Down. While the charm is on the shelf when a day is prepared, a fish kind of zone z lives in zone (last zone - z): the six zones mirror. The kind keeps everything else (bait, size, weight, home zone in its data). The open zones and the reach of the hooks do not change. The Gold Sunfish is not moved. Follow the meanings the code already has for texts like "Sunlight fish" and "caught below Sunlight"; if a charm rule becomes unclear with mirrored zones, report it as a question.
3. Tiny Terror. The size ladder is off for eating: a free fish of any other kind may eat the hooked fish, larger or smaller. A fish never eats its own kind through this charm. Strike chance, reach in space and the chain bonus stay as they are. The Gold Sunfish still eats nothing and is never eaten. Measure in the bot runs the longest chain and the largest single fish score with the charm; if a chain can grow without an end, report it as a question with the numbers (do not invent a cap).
4. Whopper. Once per day, at a random time in the first half of the day, one fish of a random kind that lives in the open zones enters with ten times the size and ten times the weight. It bites, hangs, escapes and can be eaten by the normal rules of its kind. Reuse the per-fish size and weight path that Giants uses if there is one. Give the view the size factor of each fish as data.
5. Tests for each charm: effect on, effect off after a sale, determinism with the charm in the pool.
6. Order: after the three rod charms, before the refit. Report the pass rates with all six new charms in the pool and without them.
