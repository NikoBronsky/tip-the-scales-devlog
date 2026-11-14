# Brief: the owner's notes after the first play of the build on points (interface)

Written by the orchestrator on 12.11.2026, about 23:00. Executor: Opus, effort high.
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui` (HEAD `36e1851`). Nothing else.

## 1. Goal

The owner played the packaged build and said four things (voice, 12.11, about 22:40). Each is a recorded decision in
`Docs/decisions/game-frame.answers.json` (main checkout `F:\UE\EpicJam`).

1. `easy-start-hard-later`: «стало ультра хардкорно, ультра тяжело. [...] В начале все должно быть чуть-чуть полегче.»
   Your part: a new run starts on the level Soft. (The numbers are tuned by another executor in the core.)
2. `rule-days-known-ahead`: «игроку непонятно, когда начнется [...] уровень с условиями. [...] ты это узнаешь по факту, что на
   уровне нельзя будет кормить червем. Ты это узнаешь после того, как начал раунд. [...] Может, это где-то написано, но это
   очень неочевидно. Надо готовиться сильно заранее. [...] хотя бы за пару раундов.»
3. `pack-opens-by-itself`: «Мне непонятно открытие набора в самом начале игры. Зачем мне его отдельно открывать? По идее, у нас
   раунд должен начинаться с этого открытия.»
4. `cards-lean-to-cursor`: «У нас просто всплывают карточки. Надо, чтобы они еще немножко двигались. Чтобы они двигались
   относительно курсора. То есть не просто всплывали, а еще и как-то плавали красиво. [...] как будто бы они нормально бы
   смотрели на курсор и от нее гнулись.»

A "round" in the owner's words is a day. The model for the card feel is the cards of Balatro (behaviour only; copy no art).

## 2. What exists now (verified by the orchestrator in `F:\UE\EpicJam_ui`)

- New run level: `Catch/Source/Catch/CatchSession.cpp:33-34` builds the game with `FeeLevel::Normal` and logs
  "fee level Normal". `catch.NewRun [seed] [Soft|Normal|Hard]` (`:445-455`) keeps the current level when no level is given.
- Rule days: the core gives `State().dayRule` and `State().tomorrowRule`. New since `83c3d8b`: `Game::RuleOf(int day)` is
  public (`Catch/Source/CatchCore/Public/CatchCore/Game.h`), None on a day without the Gold Sunfish. The period is
  `GetTables().pace.specialPeriod` (3). Rule text: `UCatchHud::DayRuleTextOf` (`CatchHud.cpp:454`).
- Where a rule shows today: the preparation banner on the rule day itself (`CatchHud.cpp:603`); the shop box "TOMORROW"
  only on the evening before (`CatchHud.cpp:1168-1173`, painted at `CatchUiPanels.cpp:867`). Nothing shows it earlier.
- Pack: the bait box has an "OPEN PACK" button (`CatchUiSide.cpp:178`, hit at `CatchHud.cpp:2585-2588`). The click calls
  `Session()->OpenPack(i)`. Then a closed pack stands on the screen and waits for a second click that tears it
  (`PaintPack`, `CatchUiPanels.cpp:384-470`; `catch.HudOpenPack`, `CatchHud.cpp:2877`). Then the bait cards come out
  (`PaintBaitCard`, `CatchUiPanels.cpp:316`, called at `:463`). The preparation tip says "Click OPEN PACK on the left. Then
  click to open the pack." (`CatchHud.cpp:2444`). A run starts with one unopened Small Pack.
- Cards: `PaintBaitCard` (`CatchUiPanels.cpp:316`), `CharmCard` (`:624`, drawn in the loop at `:820`), the fish cards, pack
  tiles, worm tin and chum tiles of the shop in the same file. The closed pack already tilts (`Panel::PackMaxTilt`,
  `CatchUiPanels.h:59`). The painter turns and scales through `FPaint::Transformed(center, scale, angle)`.
- Agent commands that exist: `catch.Shot`, `catch.After`, `catch.Hover <fish name>`, `catch.HudToShop`, `catch.PrizeTable`,
  `catch.HudOpenPack`, `catch.BuyChum`, `catch.Buy`, `catch.AutoPull`.

## 3. Technical decisions (revocable: change one if it breaks something, and say so in the report)

1. Level: a new run starts on `FeeLevel::Soft`. The level name is not shown to the player (a level picker is a later layer).
2. Forecast: the game always shows the next Gold Sunfish day, how far it is, and its rule. Find it by scanning
   `RuleOf(day + k)` forward, at most `specialPeriod` days. Do not copy the core's rule into the view.
3. Forecast places:
   - a plate under the day panel (top left), visible in the preparation, in the day and in the shop: line 1
     "GOLD SUNFISH IN 2 DAYS" / "GOLD SUNFISH TOMORROW"; line 2 the rule text in `Col::RuleText`; a strip of the next three
     day numbers with the Gold Sunfish day in gold. On the rule day itself the plate shows the next one;
   - the preparation banner: a second line when the rule day is two days ahead or nearer ("In 2 days: Gold Sunfish. Fish
     ignore Worm.");
   - the shop box "TOMORROW" becomes "COMING" and always shows the next rule day and its rule; when it is tomorrow, keep the
     stronger look that exists today.
4. Pack: no separate "open" action.
   - When the preparation begins and a pack is unopened, the view opens it by itself: the day banner stays readable for
     about 0.9 s, the closed pack rises, tears itself after about 0.5 s (same animation and sound as the click), the cards
     come out. The player only picks. More packs follow one by one.
   - After a successful pack purchase in the shop the pack opens the same way at once, so the player can still buy bait
     after seeing the cards. If the core refuses `OpenPack` in the shop, leave the pack for the next preparation and report.
   - The "OPEN PACK" button stays as a fallback for a pack that is still unopened.
   - The preparation tip no longer tells the player to click OPEN PACK.
   - Add `catch.HudAutoPack 0|1` (default 1). The orchestrator's scripted day starts with `catch.HudAutoPack 0`.
5. Cards float and lean toward the cursor. Applies to: bait cards of the pack choice, charm cards, fish cards, pack tiles
   and the worm tin, chum tiles (half the amounts). Numbers to start from; tune by eye:
   - idle: each card bobs 3 px and rocks 1.2 degrees with its own phase, period 2.8 to 3.6 s;
   - lean: a card whose centre is within 320 px of the cursor leans toward it, up to 9 degrees of fake 3D tilt, falling to 0
     with distance;
   - hover: the card under the cursor lifts 8 px, grows to 1.06 and tilts up to 14 degrees by the cursor position inside
     the card; its shadow slides the other way (up to 6 px); a soft sheen band moves across the card with the tilt;
   - every value follows through a damped spring (settles in about 0.12 s), frame-rate independent; nothing snaps;
   - a click gives a short squash (0.94 to 1.0 in 0.12 s);
   - text and icons move with the card. Hit boxes stay at the rest pose (no flicker at the edges).
6. The HUD stays a painter HUD: no meshes, no materials, no assets. Fake 3D is an affine transform (shear plus uneven
   scale); extend `FPaint::Transformed` or add a sibling.
7. Add `catch.HudCursor <x> <y>` (a fake cursor position for frames, `-1 -1` switches it off) if no such command exists.

## 4. Work

- Block A: level Soft for a new run (`CatchSession.cpp`).
- Block B: the forecast (decisions 2 and 3).
- Block C: the pack opens by itself (decision 4).
- Block D: the cards float and lean (decisions 5 to 7).

Files you may touch: everything in `Catch/Source/Catch/` and `Docs/briefs/reports/day1-hud-feedback.md`.

## 5. Done means

- `check_all.py --ue` is green.
- The scripted day of section 6 ends with the same numbers as before: 8 fish, 31 pts against a target of 15, payout
  4 + 4 + 0, coins 8, no "Error:" line. If a number moves, say why.
- A new run without `catch.HudAutoPack 0`: the pack opens by itself in the first preparation; a pick puts the bait into
  the box; Start Day works after the pick. A pack bought in the shop opens at once.
- Day 1 preparation shows "GOLD SUNFISH IN 2 DAYS" and "Fish ignore Worm." without any click.
- Frames (1920x1080, `Catch/Saved/ue_mcp/`, you looked at each): `fb_pack_self` (the pack tearing by itself),
  `fb_pack_cards` (cards out, cursor away), `fb_pack_hover` (one bait card under the fake cursor), `fb_tilt_1`, `fb_tilt_2`,
  `fb_tilt_3` (three frames 0.15 s apart, cursor moving across the charm cards in the shop), `fb_forecast_prep`,
  `fb_forecast_day`, `fb_forecast_shop`, `fb_shop_pack` (a pack bought in the shop, open).

## 6. Verification (run exactly; quote `-ExecCmds`, an unquoted line hangs the game)

```
python -X utf8 Tools/check_all.py --ue
"F:\UE_5.9\Engine\Binaries\Win64\UnrealEditor.exe" "F:\UE\EpicJam_ui\Catch\Catch.uproject" -game -windowed -ResX=1920 -ResY=1080 -nosplash -unattended -Multiprocess -benchmark -fps=30 -log -ExecCmds="catch.HudAutoPack 0,catch.NewRun 42,catch.After 0.5 catch.Shot fb_prep,catch.After 1 catch.StartDay,catch.AutoPull 3,catch.After 23 catch.Shot fb_d23,catch.After 70 catch.Shot fb_summary,catch.After 78 catch.Dump,catch.After 80 catch.HudMoney,catch.After 81 catch.HudToShop,catch.After 83 catch.Shot fb_shop,catch.After 86 quit"
```

The log is `Catch/Saved/Logs/Catch.log`: look for `DaySettled`, `CATCH_DUMP`, `CATCH_HUD`, and count "Error:" lines.

## 7. Boundaries

- Work only in `F:\UE\EpicJam_ui`. Do not edit `Catch/Source/CatchCore` or `Catch/CoreTests`: another executor owns the core
  in `F:\UE\EpicJam_core`. You need a core change: write it in the report.
- No Unreal editor (use `-game` runs that you start and that end by themselves). No `.uasset` or `.umap` changes.
- You are the only executor who runs UnrealBuildTool now. Never stop an Unreal process that you did not start.
- All player text is English. No logo, no brand name.
- No new rule, number of the rules, or screen. The amounts of decision 5 are look numbers and are yours to tune.
- Do not edit `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md` or `Docs/design`.
- No commits. No deletions. Old code that becomes unused stays, listed in the report.

## 8. Report

`F:\UE\EpicJam_ui\Docs\briefs\reports\day1-hud-feedback.md`, in English: state and gate; what each block changed (files);
the scripted day numbers; the frames with one line each; decisions of section 3 you changed; what still differs; questions.

## 9. Chain of command

You report to the orchestrator, not to the owner. Questions go into the report. The orchestrator answers technical
questions and takes director-level ones to the owner.
