# Brief: interface fixes and feedback from the owner's play session (13.11)

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 02:30 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`, base `7aa3ee4` (clean). This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-hud-fixes.md`).

## 1. Goal

Fix the interface bugs the owner found by playing packaged build 4 to day 15, and add the feedback he asked for. No game rule changes: the core (`CatchCore`) is not yours.

Why it matters. The owner (game director) judges the game by feel: «Я хочу, чтобы везде было бы максимально отзыва.» Two of the bugs are the kind a scripted run cannot see, because the scripted runs never use the mouse. Each item below quotes his words (voice input, Russian, verbatim) and names the recorded decision in `F:\UE\EpicJam\Docs\decisions\game-frame.answers.json`.

## 2. What exists now (found by a read-only agent on `f7e7a97`; verify each line before you rely on it)

Paths are under `F:\UE\EpicJam_ui\Catch\Source\`. The sound commit `7aa3ee4` added about 140 lines to `Catch\CatchTackleView.cpp`, so its line numbers below have moved down.

1. Click on a fish. `Catch\CatchHud.cpp:2419` `NativeOnMouseButtonDown` calls `PickHandle(Pixel)` (2396-2417: the nearest rod handle within 45 px, the handle sits at `goalDepth`, `CatchTackleView.cpp:292-293`) and starts a depth drag without looking at the rod state. The "menu" the owner saw is the drag label (`MakeDragView`, `CatchHud.cpp:2884`, painted by `Ui::PaintDrag`, 3067). On release, `NativeOnMouseButtonUp` (2487-2495) calls `S->SetDepth`, which the core refuses silently while the rod is Hooked or in Pull (`CatchCore\Private\Game.cpp:93`). A pull is only possible through the rod card button `Rod%d.Pull` (`CatchHud.cpp:299-306`). The core accepts a pull in Sink, Wait or Hooked (`CatchCore\Private\Rods.cpp:581`).
2. Scales. The 3D scales on the raft (`Catch\CatchLookScales.cpp:36-42`, clamp +-10 degrees; inputs at `CatchTackleView.cpp:416`: `Scales.Step(Dt, St.bucketTotal, NormOf(St.day), ...)`) tilt the right way. The small scales in the COINS panel (`Catch\CatchUiHud.cpp:85-114`, drawn only when the raft deck is off screen, `CatchHud.cpp:525`) are inverted: `EndL = Pivot - Dir*18` (the bucket pan) goes up when the angle is positive, although the comment says "+ = the catch side, left, down".
3. Bait pack. Picks are `UCatchButton` hits that fire on mouse release (`CatchUi.cpp:754-762`). The hits are declared at `CatchHud.cpp:1646-1668` (`PackTear` full screen, then `Pick%d` at `PackCardRest`). There is no input lock: the Pick hits are live as soon as `bOpened`, while the cards appear from 0.2 s and fly until about 0.74 s (`CatchUiPanels.h:62-67`, `CatchUiPanels.cpp:472-473`). The middle card's rest rect (176x256 at (W/2, 353)) covers the pack centre, so the click that tears or buys the pack also picks. `PanelHits` reuses buttons by index and swaps the action every frame (`CatchHud.cpp:796-799`). `BuyPack` opens the pack in the core at once (`CatchCore\Private\Shop.cpp:295`); auto-tear after `AutoTearSeconds = 0.5`; the starting pack auto-opens after `AutoPackDelay = 0.9` (`CatchHud.cpp:1159`).
4. Layout, reference pixels at 720 high. Day panel (20,16) 232x70 (`CatchUiHud.h:15-16`). Rule box top 92. Forecast plate x 20, y 92, or 168 on a rule day, 232x66 (`CatchUiHud.h:53-56`, `CatchHud.cpp:690-691`). Bait box x 8, width 114, top 104/168/166/242 (`CatchUiSide.h:26-32`, `CatchHud.cpp:2616-2622`), painted on layer z 8 above the plate layer z -10. The "0 pts" chip is `PaintPanPlate` (`CatchUiHud.cpp:117-124`) at a projected world point (`CatchHud.cpp:528-531`). The scales and the bucket are 3D components at `LookScalesOrigin = (825,240,16)` (`CatchTackleView.h:68`). The plates and the bait box cover them.
5. The "ball" at the rod foot is the rod level badge (`CatchTackleView.cpp:1130-1131`, colour `RodColorOf(Level)`). The small ring in the water is the depth handle at `goalDepth` (`CatchTackleView.cpp:1279-1284`).
6. End of day. `EndDay` pulls the rods, then Shipping ("RODS COMING UP", `CatchHud.cpp:2850`). The core emits one `ShipItem` per fish with shrinking gaps (`CatchCore\Private\Days.cpp:77-108`). Per item the fish flies to the coins panel (`CatchTackleView.cpp:774-793`), a "+N pts" popup, sparks and the Coin sound play (`CatchHud.cpp:2020-2027`), and the bar "WEIGH-IN +X pts" lerps (3043-3044). Then Summary: `Session()->Pay()` (1587-1594), `OnDaySettled` (2094-2117) shows one "+N coins" popup, the coin counter lerps (457), `PayoutInFlight = 0`: no coin flies. `PaintSummary` (`CatchUiPanels.cpp:528-560`) draws static rows. Unused leftovers you may reuse: `PayoutHold`, `FeeCoins`, `FeeCoinGap`, `FeeCoinFly`, `FeeCoinArc` (`CatchUiPanels.h:39-46`), `EPanel::Paid` (`CatchHud.cpp:1781`), `PaidTime`, `PayoutLandsAt` (453).
7. Day timer. `PaintDayPanel` (`CatchUiHud.cpp:20-31`); `bLate = St.dusk || St.secondsLeft <= 15.0f` (`CatchHud.cpp:488`) only changes colours. No pulse, no countdown, no sound.
8. Rule day. A banner only at the start of preparation (`CatchHud.cpp:2158-2161`, text 613-618, forecast line 626-634) and a small "DAY RULE" box (`CatchUiHud.cpp:33-47`). Nothing marks the Worm tile or the Sunlight zone. The drag label ticks Worm fish on an IgnoreWorm day because it uses `BaitMatches`, not `HookAccepts` (`CatchHud.cpp:2881`).
9. Shop tiles. Fish cards draw `FishShape` (`CatchUiPanels.cpp:184-210`, colours `FishLooks` 90-94). The chum tile has no fish picture (835-846, `CatchHud.cpp:1541-1544`).
10. Card edges. `FPaint::Tilted` uses `FSlateRenderTransform` (`CatchUi.cpp:222-243`). `Fan` is `MakeCustomVerts` without anti-aliasing (343, 349). `Band` is a Fan inside a rectangular `PushClip` (97-104, `CatchUi.cpp:527`): under a tilt that clip gives hard stair-step edges.
11. Line and bait sway. The owner sees the line and the bait sway in preparation and stand still during the day. The cause is not located yet; start in `CatchTackleView.cpp` where the line and bait transforms are built.
12. Console commands of the HUD exist for hover (`catch.HudCursor x y seconds`) but none for a click.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Bugs

1. Click to pull. Decision `click-fish-to-pull`: «Я не могу вытащить рыбу, щелкнув на ней. Вместо этого вылезает меню для того, чтобы поменять высоту.» A left click on a fish that hangs on a hook, or on the handle of a rod that holds a fish, pulls that rod (`S->Pull(Rod)`) and starts no depth drag. A rod without a fish keeps the depth drag. The hit area of a hooked fish is its projected position with a generous radius; when two rods compete, the nearest fish wins. The hover highlight of the fish must tell the player that a click pulls (reuse the existing highlight; a short "PULL" hint is fine).
2. Small scales. Decision `scales-tilt-right-way`: «весы работают неправильно, наоборот они работают». Fix the sign so the catch pan goes down as the catch grows against the norm, exactly like the 3D scales.
3. Pack never picks by itself. Decision `pack-never-picks-for-player`: «Попробовал купить набор карточек, и у меня сразу же одна куплена. Хотя я открыл только упаковку». A pick is accepted only when both hold: the cards have landed (fly-in over), and the mouse button was pressed after the cards landed. A press that began earlier (the buying click, the tearing click, a held button) never picks. Apply the same lock to the starting pack.
4. Raft scales visible. Decision `raft-scales-visible`: «Весы, которые находятся на деревянном плоту, их не видно, они закрыты табличками и подписями.» At the default camera, no plate, chip or box may cover the scales, the pans or the bucket. Move or reshape the forecast plate and the bait box (your choice; keep them readable and near their present place). Prove it at 1920x1080 and 1280x720 with frames in preparation, in a normal day and on a rule day.
5. Line sway. Decision `line-sways-in-day`: «раньше [...] леска двигалась вместе с наживкой. Сейчас она так делает только на стадии приготовления.» The line and the bait sway during the day too, for a rod in Sink and Wait and, smaller, with a fish on the hook. Use the sway that preparation already has; no new numbers unless the hooked case needs one.
6. Card edges. Decision `cards-smooth-edges`: «когда они двигаются, как будто появляется лесенка». Tilted and moving cards get smooth edges. Suggested: stop clipping tilted bands with a rectangle; draw the band as geometry in the card's own space and feather the outer edge with a thin strip of vertices whose alpha goes to 0. Prove it with zoomed frames before and after.
7. Rod badge. Decision `rod-ball-unclear`: «я не понимаю, что за точка, за шарик рядом с удочкой». Give the badge a meaning the player can read: show the rod level as a digit on it and a hover line "Rod level N". At level 1 hide the badge.

Feedback

8. Rule day on the full screen. Decision `rule-day-fullscreen`: «раунду с ограничением надо писать на весь экран, какое именно ограничение. Потому что я не сразу догнал». When the preparation of a rule day begins, show a full-screen card: the title "DAY RULE", the rule text from the core in large type, and what it switches off as one icon line. It holds until a click, or 3.5 s. A smaller reminder repeats when the day starts. The day cannot be started while the card is up.
9. Mark what a rule switches off. Decision `mark-what-rule-disables`: «как-то отметить заблокированные зоны [...] Приконку на червя как-нибудь крестиком перечеркнуть.» Required: on an IgnoreWorm day the Worm tile in the bait box and the Worm on rod cards are crossed out, and the drag label uses `HookAccepts`; on a NoSunlightBites day the Sunlight zone carries a visible mark across its width (a hatch or a tint plus the label "NO BITES"). Also, as far as time allows: FastEscape marks the escape timers; CheapFishWorthless shows "0 pts" in the fish hover for a fish under the limit; ShortDay marks the timer. Read the rule from the core state; do not hard-code which day has which rule (the order will become random).
10. Countdown. Decision `day-end-countdown`: «когда остаются секунды, на какую-то отбивку, 2, 3, 1, там, чтобы игрок понял, что осталось 5 секунд.» In the last 5 seconds of a day show a large digit 5, 4, 3, 2, 1 that pops each second, pulse the day timer, and play a tick each second from the existing sound files. No new sound files.
11. End-of-day count. Decision `day-end-count-with-feedback`: «Не вижу эффекта в конце, когда идет подсчет рыбы. Я не вижу справа [...] чтобы как-то что-то менялось. Цифры увеличилось, какие-то эффекты. [...] хотелось бы, чтобы монетки по одной начислялись. А очки, тоже числом постоянно разгонять». During Shipping the points total on the right grows with each fish, with a pop of the number and of its panel on every hit, faster as the gaps shrink. In the Summary the rows appear one after another and each number counts up (points, norm, base, over the norm, interest); then the coins fly one by one from the summary to the coin counter, the counter adds 1 with a pop and a coin sound for each, and the last coin gets a bigger pop. The whole Summary sequence stays under about 6 s and a click skips to the end state. The settled numbers must equal the core's (`DaySettled`).
12. Fish on the tiles. Decision `upgrade-shows-its-fish`: «в начале не очень очевидно, какая рыба улучшается. Неплохо бы видеть ее иконку [...] А вот на чам, я так понимаю, прикормка, нет ничего.» The chum tile shows the shape of its fish kind, as the fish card does; both show the fish name clearly. These shapes are placeholders: pictures made by an image generator replace them later, so keep one function that draws "the picture of fish kind K at this rect".

Proof by mouse

13. Add a console command `catch.HudClick x y` (reference pixels, same as `catch.HudCursor`) that sends a press and a release through the same code path as a real left click, and `catch.HudPress x y` / `catch.HudRelease x y` if you need them apart. Use them in a scripted run that proves decisions 1 and 3.

Texts. All player-facing text is English. New texts allowed by this brief: "DAY RULE", "NO BITES", "PULL", "Rod level N", the digits. Take rule texts and fish names from the core.

## 4. Work order

Bugs first (1, 3, 2, 4, 5), then feedback (8, 9, 10, 11), then 12, 6, 7, then 13 and the proof runs. If you run short of session, stop after a finished item with a green build and report exactly what is left.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui`: `10 ok, 0 fail, 1 skip`.
2. The plain scripted day is unchanged: 102 events, DaySettled `31 pts, target 10 pts`, coins 8, 0 errors (command in section 6).
3. A scripted mouse run shows in the log: a click on a hooked fish gives `CATCH cmd Pull ...: Ok` and no depth change; a click on an empty rod's handle still starts the depth drag; buying a pack with the cursor over the middle card's place gives no `Pick` until a later click, and that later click picks.
4. Frames, looked at by you, for: the full-screen rule card; a crossed Worm tile on an IgnoreWorm day; the marked Sunlight zone on a NoSunlightBites day (`catch.SetRule` or the existing way to force a rule; add a debug command if none exists); the countdown digit; the Shipping total mid-count; the Summary mid-count with a coin in flight; the chum tile with its fish; the raft scales uncovered at both resolutions; a zoomed tilted card edge before and after; the rod badge with its digit.
5. The small scales tilt the same way as the 3D scales for the same catch and norm (a frame with the camera low enough to show the small scales, or a log line with both angles).
6. No change in `Catch/Source/CatchCore/**`.

## 6. Verification commands

```
python -X utf8 Tools/check_all.py --ue
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_check.sh" /f/UE/EpicJam_ui fx "catch.HudAutoPack 0,"
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_feedback.sh" /f/UE/EpicJam_ui fxf
```

`run_check.sh` runs the gate and the plain day (expected numbers in section 5). `run_feedback.sh` runs the flow with the starting pack: expected `OpenPack 0 Ok`, `Pick 1 Ok`, `StartDay Ok`, DaySettled `23 pts, target 10 pts, coins 8`, `Buy pack 0 Ok`, `Pick 0 Ok`, `NextDay Ok`, 0 errors. If decision 3 changes when a scripted `catch.Pick` is accepted, keep the console command working (it calls the core directly) and say so. Frames land in `Catch/Saved/ue_mcp/`. Read the two scripts before you copy them for your own runs; do not edit them in place.

Build notes. The editor must be closed while UnrealBuildTool builds. You are the only one who builds the Unreal project now; another executor works in `F:\UE\EpicJam_core` on the engine-free core and does not build Unreal. Scripted runs use `-benchmark`, which turns the audio device off; that is expected.

## 7. Boundaries

- You may change: `Catch/Source/Catch/**` and your report. Nothing else.
- Do not touch `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Content/**` (no editor work is needed; if you find you need an asset change, stop and write it into the report), `Catch/SourceArt/**`, `Tools/**`, `CLAUDE.md`, `AGENTS.md`, `Docs/**` except your report.
- No rule, number or screen beyond this brief. No shop-before-day-1 work, no modifier shelf changes, no fish patterns, no pictures from generators: those are separate tasks.
- Nothing from the browser prototype goes into the game. No downloads, no new fonts, no new sound files.
- Keep what works: the card tilt and hover, the pack tear, the bite and pull feedback, the sound bed. Do not delete or hide existing elements; if one must move, say where.
- No commits, no pushes, no packaging. Close the editor if you opened it. Never kill an Unreal process you did not start.

## 8. Report

Write `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-hud-fixes.md` in English and hand back a short summary: per item of section 3 what is done and where (file and function), what is not done, the check results with the log lines, the list of frames with one line on what each shows, every decision you changed, and questions.

## 9. Chain of command

You report to the orchestrator, not to the owner. Answer technical questions yourself and list them as revocable readings. A question about what the player should feel or see beyond this brief goes into the report, and you continue with the items that do not depend on it.
