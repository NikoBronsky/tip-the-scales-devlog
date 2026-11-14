# day1-look-hud — report (checkpoint 3: blocks A, B, C, D)

**Blocks done:** A (kit, day screen; ac45d42). B and C (panels 7.1–7.7, shop, purchase answers; c8dff31). D in new `CatchUiSide.{h,cpp}` plus `CatchHud`: depth gauge 6.3 (water ramp, veil line, zone ticks, locked names, view window, rod dots; click or drag moves the camera), bait menu 6.5 (opens from the rod card chip, bare hook row, current row gold, Deny on none), bait box 6.7 (OPEN PACK button with SMALL/BIG and note, tiles with counts, bump on pour arrival, bait card on hover, pours fly to their tile), shelf 6.8 outside the shop with tooltip and wobble (CharmFired, ShipItem charms), hint bar 6.10 (phase hints days 1–2, first-hook/chain/escape tips), ship bar 6.12 (count-up, skip line), drag label 5.5 (depth, zone fish, "line ends here", FLOOR), fish hover card 6.13 (painted, wraps, stays inside the viewport). Old UMG panel is always collapsed: no "Open Small Pack" button or "Bait: Worm" line remains. Title screen 7.8 not done (no time).
Money: every money, fee and catch read goes through `UCatchHud::MoneyFacts()` (decision money-like-balatro). Charm tags: keyed by core name, unknown name falls back to its longest word in capitals; no code assumes 20 charms.
Frames (`Catch/Saved/ue_mcp/`): `ui_8_day_two_rods`, `ui_9_16x10_two_rods` (1728x1080), `ui_d1_prep_box`, `t_d_pour`, `ui_d2_box_card`, `ui_d3_bait_menu`, `ui_d4_drag`, `ui_d5_day_hint`, `ui_d6_ship_rods`, `ui_d6_ship_bar`, `ui_d7_hover_card`, `ui_d8_shop_tip`, `ui_d9_shelf_tip`, `ui_d11_gauge_deep`, `t_wob_732` (wobble), `t_menu_day2`.
New camera checked: drag line, handle and gauge dots sit on the hook lines (`ui_d4_drag`, `ui_9_16x10_two_rods`).

**Colour table (reference → measured, 1920x1080; earlier rows unchanged):**
| Part | Reference | Measured |
| --- | --- | --- |
| Start Day / NEXT DAY, clock bar, bait chip | #ffd166, #7fe3d6, #123247 | same |
| Shop panel, locked tile, bait menu | #0e2636 | #0e2636 |
| Money amount | #ffe08a | #ffe08a |
| Depth gauge panel | #061622 60% over sky | #081926 (calc #091b29) |
| Bait box panel | #061622 66% over sky | #081a27 |
| Hint bar / tip bar | #061622 75% / #3c2406 88%, text #ffe7a8 | as set, not measured |
| Ship bar / fish card | #04101a 82% / #0a1e2c 96% | as set, not measured |

**NEW words (D):** BAIT BOX, OPEN PACK, SMALL/BIG, "Pick a of b.", "Bare hook", "Catches: shoal fish only, four times slower", "<Bait> xN", none, phase hints and tips ("Fish on! Click the rod card to pull it in." etc.), RODS COMING UP, "SELLING THE CATCH +$N", "Click or press Space to skip", "Caught on:", card notes (shoal, hunter, floor), "Unknown fish.", "Buy Deeper Line to open the X zone.", "Zone X · price now $N · lv L", "Special fish · price now $N", "Class: Rod", "Click to sell.", "depth N · Zone: fish", "line ends here", FLOOR, "x " prefix on locked zone names.

**Still differs from reference (most important first):**
1. Title screen 7.8 not done.
2. Charm-fired tag popups (section 8: PULLS, HOLDS…) not done; chips only wobble.
3. Five-rod frame not possible by console (need 2); 1728x1080 frame has 2 rods.
4. Charm card has no tilt toward the pointer and no sheen; fish in fish cards are drawn shapes.
5. Not seen in a frame: unknown-fish card, pack hover note, drag tick, victory panel, notice bar.

**Decisions changed (revocable):** as checkpoint 2. New: day hints show on days 1–2 only; teaching tips once per launch; bait box shows every bait held this run (0 counts stay); a tile click does nothing; the menu lists the same baits; the gauge centres the view on the clicked depth.

**Needs outside my files:** 1) Charm rule text in the core tables. 2) A money or five-rod console command. 3) Seen-bait flags in the core. 4) `CatchSeaView` should read `UCatchHud::GetHoverFish()` (the sea still picks its own hover fish; review item 7).

**Review items:** 6 hover card inside the viewport, wrapped text — fixed (`ui_d7_hover_card`). 7 hover ownership — HUD side done, sea side is need 4.

Gate: `check_all.py --ue` 10 ok, 0 fail, 1 skip (submission-zip). Not committed.
