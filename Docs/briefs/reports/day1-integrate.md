# Report: day1-integrate (executor Opus, 12.11.2026)

| Flow | State |
| --- | --- |
| A1 fish at the walls, sea across days | done: the mirror is keyed by fish id (slot pool, no index reuse); Y comes from the core `layer`; `M_Fish` clips every pixel with world \|x\| > 1000; no reset at the day end |
| A2 landing | done: a landed fish leaves the mirror; the flight takes `catchList[i].flightSeconds` and `pace.flightStagger`; it waits in the bucket mouth until `FishInBucket` (rock, splash, sound); a Knife Sprat shows "+N <bait>" in green, no money |
| A3 day end | done: day-end flights during `Shipping`; a batch `ShipItem` shows "N Sprat +$X" and flies up to 4 fish; the 0.8 s summary delay and the remembered skip come from the core |
| A4 packs | done: Open/Pick work in the preparation and the day (bait cards at the bottom); with a pack open the prep panel says "Pick a bait first" and START DAY is disabled; locked packs show "(day N)" |
| A5 rod | done: the handle reads the core goal; bait change and empty pull go straight to the core; the Auto-Reel countdown shows on the rod card only while Auto-Reel is on the shelf |
| A6 hunter | done: HUNTER NEAR (blinking, orange card) from the hook state |
| A7 dusk, day rule | done: the clock turns orange and pulses at dusk; the rule text shows orange next to the counters |
| A8 victory | done, not reached in a test run: panel "Day N" and a "Keep fishing" button on `Victory`; `SetFeeLevel` only via `catch.NewRun <seed> Soft/Normal/Hard` |
| A9 Dump | done: fee level, dusk, landed/caught/bucket/shipped, fish in flight, rod x, bare-by-choice, auto-reel, cluster, attracted, catch depth, price x, 8 fish ids near the surface, locked packs |
| B hover card | done: nearest mirrored fish within 30 px of the cursor; name, price at its depth now, "Catches: <bait> <portions>..." and "Bare hook"; an unknown fish names the upgrade that opens its zone; follows the fish, hides on mouse leave, works in Prep; `catch.Hover <id/-1>` |
| C water rim | done: `M_WaterRim` (unlit, translucent, soft glint) on a thin strip `WaterRim` along the front water line, tag WaterA |

**New for a player.** The hover card. The rim. Pack choice during the day. Batch floats. HUNTER NEAR. Dusk and the day rule. Floating numbers above the bucket stack: a new one starts at the bucket, older ones move up one line.
**Checked.** `check_all.py --ue`: 10 ok, 0 fail, editor closed. One `-game -benchmark -fps=30` run, seed 42: day 1 by console (pack opened and bait picked at 3-4 s of the day), New Rod bought by a simulated click, days 2-3 played only by simulated clicks (NEXT DAY, START DAY, rod-card pulls, PAY), day 4 with 4 rods and batches (`Sprat x15`). Dump = HUD: $100/$85, $242/$212, $255/$185 (day 3 = rule day), $162. A new agent tool `catch.AutoPull <s>` pulls a rod when its first escape is this close (used for day 1 and 4 only).
**Frames** (`F:\UE\EpicJam\Catch\Saved\ue_mcp\`, 1920x1080): `int_1_prep`, `int_2_day_pack_open`, `int_3_hunter`, `int_4_day_end_flights`, `int_5_ship_batch` (= `_b`; `_a`..`_d` are the sequence), `int_6_hover`, `int_7_rim_before` (1280x720, taken before the rim existed), `int_8_rim_after`. No fish outside the block in any frame.
**Values the picture needed and the core does not give.**
1. Fish Y across the block: the view maps `layer` 0..`pace.layerMax` to the block depth (near cut to back wall - 150).
2. The hover pick radius (30 px) and refresh (0.25 s) are view constants.
3. The core keeps `autoReelSeconds` on every rod; the HUD hides it without the charm (a core flag would be cleaner).
**Notes.** Simulated panel clicks are flaky: in 2 of 6 final runs the shop clicks after PAY hit the right button (log names it) but did nothing, and the run went another way. `catch.SimClick` now drops any pointer capture first and logs the widget under the click; no capture was found, so the cause is still open. A rerun of the same commands passed. HLSL: a Custom node variable named `line` fails to compile (reserved word); that made the first rim a dark band.
**Assets (by script).** `Catch/SourceArt/SceneScripts/build_play_integrate.py` (M_Fish clip, `M_WaterRim`, actor `WaterRim` in `/Game/Maps/Sea`); the same steps are in `build_sea_materials.py` and `build_sea_level.py`.
**Questions.**
1. Victory title: reference 14 gives only "Keep fishing"; the look reference proposes "YOU MADE IT" (NEW). Which words?
2. "Pick a bait first" is a temporary word. Keep it?
3. The hover card lists baits with 0 portions (as the reference lists all baits of a fish). Show only the baits the player holds?
