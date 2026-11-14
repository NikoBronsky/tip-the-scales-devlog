# Brief: the interface on points and coins (HUD v2)

Date: 12.11.2026. Executor: Opus, effort high. Worktree: `F:\UE\EpicJam_ui`, branch `track/ui` at `797b76c` (main `c48dfb9` plus the rules core v2, commit `e657058`). You report to the orchestrator.

## 1. Goal

The rules core now counts the catch in points by weight and pays coins; the Unreal view still prints dollars and a fee. Make the view tell the truth of core v2, then bring the new screens to the look of the owner's probe.

Owner decisions this rests on (`Docs/decisions/game-frame.answers.json`):
- `score-in-points-by-weight`: «Улов дня считаем в очках. Долларов в игре нет. Монеты остаются единственными деньгами. У рыбы есть вес. Тяжелее рыба — больше очков. [...] Игрок видит таблицу наград по видам рыбы.»
- `scales-catch-against-norm`: «Левая чаша показывает очки улова. Правая чаша показывает норму, которую нужно набрать за день. Игрок видит, какая чаша перевешивает.»
- `money-like-balatro`: «Улов дня закрывает плату. Деньги магазина приходят отдельной малой выплатой. Цены в магазине малые.»
- `food-ladder-next-size`: «Рыба ловится на рыбу на одну ступень меньше.»
- `chum-raises-fish-count`: «Игрок за деньги увеличивает число рыб выбранного вида в море (прикормка).»

## 2. What exists now (verified by the orchestrator)

- Core v2 is built and tested (`Tools/core-test.ps1` green, 340 checks). Its exact names: `Docs/design/core-interface.md`, section 11, subsection "Final names". Its report: `Docs/briefs/reports/day1-core-points.md` (section 2 "Final surface", section 3 "Mapping of the 20 v1 charms").
- Every v1 name still compiles; the meanings changed. `State::money` is coins. `catchTotal`, `bucketTotal`, every `price`, `PullEstimate::money`, `FishDetails::price` are points. `FeeOf(day)` returns the norm (`NormOf`). `PayFee()` settles the day (`SettleDay`). `FeePaid` is never emitted; `DaySettled` replaces it (new, last in `EventType`). `ShipItem` no longer changes money. `RunOver` carries norm, points, coins kept. New: `LandedFish::points/weightTenths`, `FishDetails::points, weightTenths, size, chum, award, eats, eatenBy`, `PullEstimate::points, weightTenths`, `ShopState::chum`, `BuyChum(offerIndex)`, `PayoutFor(...)`, `MoneyReason::Payout`, `MoneyReason::Gold`.
- Nothing eats a hooked fish on day 1 (`pace.eatFromDay` = 2). Chains start on day 2.
- The view prints dollars and the fee in these places (line numbers of `F:\UE\EpicJam_ui\Catch\Source\Catch`):
  - `CatchHud.cpp:43` `Money()` formats `$%lld`; `:360` pull preview; `:582` banner "Fee tonight"; `:1132` "Fee tomorrow"; `:1420` the tip "Pull now and get the money. Or wait: a bigger hunter may eat your fish."; `:1435` and `:1570` `OnFeePaid`; `:1696` the hover card reads `FishInfoOf`; `:1977-2019` `MoneyFacts()` (`FeeOf`, `catchTotal`, `bucketTotal`, the words "fee tonight" / "fee tomorrow").
  - `CatchUiHud.cpp:59-78` the money panel ("MONEY", `$`, "CATCH: %d fish, $%lld", fee line); `CatchUiHud.h:61`.
  - `CatchUiPanels.cpp:505-540` the summary rows ("Earned today", "Money", "Fee tonight", "Fee tomorrow"), the button "PAY $…" / "CAN'T PAY", the stamp "PAID"; `:556` and `:580` end texts about the fee.
  - `CatchUiText.cpp:15, 23` charm texts with money words.
  - `CatchTackleView.cpp:416` `Scales.Step(..., bucketTotal, FeeOf(day), ...)`; `:597, :796` an empty `OnFeePaid`.
  - `CatchSession.cpp:240` the table of event names (it has no entry for `DaySettled`); `:299-345, :353, :391-400` log and dump lines with `$`; `:137` `Pay()` calls `PayFee()`; `:477` `catch.Pay`.
  - `CatchHud.cpp:2703-2708` `catch.HudMoney` logs `CATCH_HUD money $…`.
  - `CatchSeaView.cpp:252` uses `fish[].tier` as the drawn size. That stays.
- A look sheet for the new screens is being written now by another agent: `F:\UE\EpicJam\Docs\design\look-reference-v2.md` (read it by this absolute path). It describes the owner's probe in words and numbers: the numbers at the scale pans, the top-right panel, the rod card and popups, the fish hover card, the prize table, the end of the day with the payout, the shop with chum, and a table of English strings. It may not exist when you start.
- The look of everything else: `Docs/design/look-reference.md` (sections 6 and 7).

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. Player-facing words. Points: "pts" after a number, "POINTS" as a heading. Weight: "kg", one decimal only when the weight is not whole ("7.5 kg", "12 kg"). The norm of the day: "TARGET" ("Target today: 50 pts"). Coins: a coin icon drawn by the painter plus the number; the word "COINS" as a heading. The award table: "PRIZE TABLE". The day-end count: "WEIGH-IN". The payout: "PAYOUT" with the parts "Base", "Over target", "Interest". Chum: "CHUM". No "$" and no "fee" anywhere the player can see. When the look sheet gives an English string for a place, the sheet wins.
2. Two formatters replace `Money()`: one for points, one for coins. No number of the rules is typed in the view; every value comes from the core state, queries or tables.
3. The summary panel becomes the weigh-in: catch in points, the target, coins held, the target of tomorrow, one button "WEIGH IN" that calls the same command as before. Under the target the core answers with `RunOver`; the run-over text reads "Your catch did not reach the target of day N." with the points and the target. After `DaySettled` the view shows the payout parts (base, over target, interest, total) and the coins go to the coin counter; then the shop opens as before. Until the look sheet says otherwise, reuse the existing stamp moment with the words "TARGET MET".
4. The tip at the first hooked fish must be true: on day 1 nothing eats. Show the "wait" part only from the day `pace.eatFromDay`, with the text "Pull now and bank the points. Or wait: a fish one size bigger may eat your fish."
5. Charm texts follow the mapping of the core report, section 3 (for example Same Again: "Each fish of the same kind in a row weighs 3 kg more than the last."; Bait Knife: "Each Sprat you land becomes 1 Live Sprat and gives no points.").
6. Logs and dumps keep their prefixes (`CATCH ev`, `CATCH_DUMP`, `CATCH_HUD`) and print "pts" and "c" instead of `$`. `catch.Pay` and `catch.HudMoney` keep their names. Add `catch.BuyChum <index>` in the style of the other purchase commands.
7. `CatchLookScales` keeps its geometry and place; the owner accepted them. You add what the sheet describes (numbers and labels at the pans) and feed it points and the target.
8. `tier` stays the drawn size of a fish. Do not use `size` for drawing.

## 4. Work, in this order

Block 1 — the view tells the truth (does not need the look sheet).
1. Build the worktree as it is (`python -X utf8 Tools/check_all.py --ue`) and run one console day (section 6) to see the starting state.
2. Events: handle `DaySettled` where `FeePaid` was handled; add the missing event name; handle `MoneyReason::Payout` and `MoneyReason::Gold`; make sure shipping no longer animates money that does not change.
3. Words and numbers: every place of section 2 and every other `$`, "fee", "money", "earned", "pay" the player can see (banner, money panel, rod card, pull preview, popups, hover card, summary, stamp, end screens, shop prices, hints and tips, charm texts).
4. The money panel: coins as the big number; the catch in points; the target with the coverage bar.
5. The weigh-in and the payout of decision 3; the run-over text.
6. The shop: coin prices everywhere; a working chum row (offers from `shop.chum`, the level held, the price, `BuyChum`, the same "not enough" answer as other purchases). A plain row in the style of the other shop rows is enough in this block.
7. Logs, dumps, console commands of decision 6.

Block 2 — the look of the probe (needs `look-reference-v2.md`).
8. Numbers and labels at the scale pans; what happens when the catch reaches the target.
9. Rod card, pull preview and world popups in points and kilograms.
10. The fish hover card with weight, size, award, what it eats, what eats it, chum level.
11. The prize table.
12. The end of the day as the sheet describes it: the count against the target, the payout parts, the Gold Sunfish coins.
13. The chum offers and the other shop changes of the sheet.
14. Every string of the sheet's string table.
15. Small extra, only if look-reference 5.3 or 6.4 asks for it: the pointer over a rod card lights that rod in the world (`ACatchTackleView::LookRodHover` exists: -1 the mouse decides, r forces rod r).

If the sheet does not exist when block 1 is done, hand back after block 1. Otherwise hand back after block 2, or after 120 minutes with complete blocks only.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` is green in `F:\UE\EpicJam_ui` with the editor closed. `Tools/core-test.ps1` is still green and untouched.
2. No player-facing text shows "$", "fee", "pay" or "money" (search the view sources and say what you searched).
3. A console day 1 with seed 42 ends with a weigh-in, a payout and the shop, with no "Error:" line in the log. The report gives the numbers of that run: fish, points, target, payout parts, coins after, and the count of `CATCH ev` lines.
4. A lost day shows the run-over screen with points and target (use a console command or a short script to get there; say how).
5. Frames, 1920x1080, from `-game` runs, in `F:\UE\EpicJam_ui\Catch\Saved\ue_mcp\`: `hud2_prep.png`, `hud2_day_hooked.png` (rod card and popup in points), `hud2_chain.png` (day 2, a chain), `hud2_weighin.png`, `hud2_payout.png`, `hud2_shop.png` (coin prices and the chum row), `hud2_runover.png`; after block 2 also `hud2_scales.png`, `hud2_prize_table.png`, `hud2_hover.png`. Look at every frame yourself before you hand back.
6. You compared each frame with the text of the sheet (block 2) and wrote what still differs.

## 6. Verification commands

```
cd F:\UE\EpicJam_ui
python -X utf8 Tools/check_all.py --ue
powershell -ExecutionPolicy Bypass -File Tools\core-test.ps1
```

A console day (one line; `-Multiprocess` is required; change the frame names as you need):

```
"F:/UE_5.9/Engine/Binaries/Win64/UnrealEditor.exe" "F:/UE/EpicJam_ui/Catch/Catch.uproject" -game -windowed -ResX=1920 -ResY=1080 -nosplash -unattended -Multiprocess -benchmark -fps=30 -log -ExecCmds="catch.NewRun 42,catch.After 0.5 catch.Shot hud2_prep,catch.After 1 catch.StartDay,catch.AutoPull 3,catch.After 23 catch.Shot hud2_day_hooked,catch.After 70 catch.Shot hud2_weighin,catch.After 78 catch.Dump,catch.After 79 catch.Pay,catch.After 80 catch.Dump,catch.After 81 catch.HudMoney,catch.After 82 catch.Shot hud2_shop,catch.After 85 quit"
```

The log is `Catch/Saved/Logs/Catch.log`. The orchestrator will run this same line after you; put the line you want the orchestrator to use, with the expected numbers, into the report.

The Bash tool of this session is broken. Use PowerShell. PowerShell 5.1 has no `&&`.

## 7. Boundaries

- Work only in `F:\UE\EpicJam_ui`. Do not change anything in `F:\UE\EpicJam` or `F:\UE\EpicJam_core`.
- Do not edit `Catch/Source/CatchCore` or `Catch/CoreTests`. If you find a core defect or a missing query, write it in the report and work around it in the view only if that needs no rule.
- No new rules, numbers or screens beyond this brief and the sheet. If something needs a decision of the owner, write the question in the report and do the parts that do not need it.
- Player-facing text is English, fit for age 13+, no logos, no brand names. Engine font as now; download nothing.
- Nothing from the browser probe goes into the game: no code, no art, no sound. You work from the two look documents.
- Keep the sea look files (`CatchLook*`, `CatchSeaView*`) as they are, except `CatchLookScales*` for decision 7 and the event handlers of `CatchTackleView`.
- If you need the editor, you may open the one of this worktree; nobody else uses an editor now. Save everything and close it before the gate. Never kill an Unreal process you did not start.
- No commits. Delete no file.

## 8. Report

`F:\UE\EpicJam_ui\Docs\briefs\reports\day1-hud-points.md`, English, at most 100 lines: state and gate result; what changed per block with files; the console-day line and its numbers; the frame list; decisions of section 3 you changed; what still differs from the sheet; not done; questions.

## 10. Addition of 12.11, after the look sheet arrived

`F:\UE\EpicJam\Docs\design\look-reference-v2.md` exists. Its part "Rulings of the orchestrator" (R1 to R16) settles its UNKNOWN list and wins over the sections below it. Three rulings change section 3 of this brief:

- Decision 3 is replaced by R13: no "WEIGH IN" button and no stamp. The weigh-in runs by itself after the rods come up, the view settles the day by itself after the last item, the summary panel shows the result with the payout rows, and "TO THE SHOP" closes it. `catch.Pay` stays for console runs and must stay safe when the day is already settled.
- Decision 1 wording: the payout rows read "Payout", "Over target", "Interest", "Coins total" (R4). Exactly 1 point reads "1 pt" (R7).
- Decision 7 is made exact by R1: keep the built scales; add only what sections 1.4 to 1.8 of the sheet describe and the game lacks.

Order of block 2 with the sheet: scales (R1) and the small scales in the coins panel; coins panel; rod card and popups; hover card; the end of the day; shop layout with chum; prize table; strings; last, only if time is left, the chips and cards of R15.

## 9. Chain of command

You report to the orchestrator. Technical questions: decide, mark the decision revocable, go on. Questions for the owner: write them in the report; the orchestrator carries them.
