# Report: day one progression probe (variant `both`)
Output: `catch-progression.html` beside the prototype (2,959 lines). `catch-prototype.html` SHA-256 `c6848b24…d1ef6d` before and after. All gates use `probeOn()` (level Normal, line 431); Soft and Hard play as before. Table `PROBE`, line 233.

| Lever | Old (Normal) | New | Line in new file |
| --- | --- | --- | --- |
| Hunters | in the sea from day 1 | wanted count 0 before day 3; refill from the edges on day 3 | 432, 560 |
| Bare hook | every shoal fish | shoal fish with effective zone 0 only | 506 |
| Fee d1-9 / Deeper Line prices | 15…7000 / 25…12000 | 20,30,60,100,300,600,1000,2200,5000 / 60,300,1500,6000,20000; x2.15 after day 9 kept | 227 |
| Deeper Line in shop | from shop of day 1 | locked tile "Откроется после дня 2" in shop of day 1; buy guarded | 436, 1148, 2635 |
| Charms | own step days | step = max(step, 3); shops of days 1-2 show three locked charm tiles (day 3), no reroll button, no false "У вас есть все модификаторы." | 1061, 2602-2607 |
| New Rod / Small Pack | 20,45,100,220 / 6 | 40,100,240,520 / 4 | 434-435, 1115, 1141, 2556, 2574 |
| Pack rule | a listed fish has an open zone | and it swims today or tomorrow (`G.day + 1`, as in core `BaitUnlocked`) | 518-519 |
| NEW notice | none | day 3 prep: «В море пришли хищники» in the dark-amber tip strip + `sfx.chord` (the new-bait notice style) | 1013 |
| Label / true texts | — | label in the page header, right side, outside the canvas (30, 56); bare-hook menu line adds «зоны Солнце» (2165); hover line only where a bare hook catches (2414); depth label omits absent species (1776); first hang tip waits until hunters swim (776) | as listed |

Checks run (Node, stub canvas, scripts in `scratchpad/probe-work/`): `node --check` on the extracted script; fees d1-10; no hunters in prep and mid-day of days 1-2, hunters enter from the edges on day 3, notice text; shops d1-2 have no charm offers and no reroll, shop d3 has both; Deeper Line absent in shop d1, 60 and 300 in shop d2; rods 40, 100; small pack 4; pack pools d1 {worm, bread, shrimp, spoon}, d2 adds {cut, live}; bare hook in Twilight with two zones: no bite in 3 days except the special gold fish (any hook, unchanged), Sunlight bites; hover cards; Soft keeps old prices, hunters, bare rule; Hard fees. Only read: drawing of locked tiles, label, notice look, sound. Not opened in a browser.
First noticed: the label; day 1 sea has no Sea Bass; shop of day 1 is almost all locked (charms, Deeper Line), so money goes to packs and rods; day 3 notice and the gold-fish banner arrive together.
Questions: (1) the hang tip moves from day 1 to day 3 so it never promises a hunter that cannot come; keep, or reword instead? (2) Deeper Line on shop day 2 gets no НОВОЕ badge (other opening sections do); add? (3) day-3 notice overlaps the special-fish banner (different rows); acceptable?
