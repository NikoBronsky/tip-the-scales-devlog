# Brief: view, part 2 - the view on core 2.3 (start gift, day that ends at the target), the day end as one flow of points

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 12:10 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message (part 1, `day2-view-menu-fixes.md`, and core 2.3 are already merged in it). This brief lives only in the main checkout.

## 1. Goal

Make the game playable on core 2.3 and turn the end of a day into one clear, pretty flow: the catch is counted in order, and the player always sees where points and coins come from.

Why it matters. The owner (game director) said on 13.11, by voice, after playing build 7:

- Start gift (`start-gift-instead-of-first-shop`): «Одна бесплатная карточка из 3 вместо магазина. Да, пускай будет так.»
- Day end (`day-ends-at-target`): «когда ты наловил на нужное очко, то у нас игра заканчивается, а в бонус … мы даем то, сколько времени сэкономлено».
- Summary (`summary-counts-in-order`): «чтобы игрок понимал, за что он получает очки, оно должно по порядку заполняться. Типа, улов, тык-тык-тык, циферки начали расти от 0 до 40 … Норма должна быть разом, появляется цифра. Выплата должна тоже аккуратно набрасываться … Не вот сразу, допустим, 4, а 1, 2, 3, 4.»
- One flow (`one-points-flow`): «У нас несколько экранов, где отображается рыба на весах слева, на весах вверху справа на интерфейсе и потом на итоговом дне. Надо сделать какую-то, ну, желательно не дублирующую информацию, и чтобы очки начислялись по красоте.»
- Camera (`day-end-camera-returns-to-surface`): «я рыбачу внизу, поэтому для меня это выглядит очень странно. Надо … камеру назад наверх перематывать … и там уже показывать, как рыбка летит из нашего ведерка в финальный улов.»
- Upgrades are invisible (13.11, about 11:25): «Мы улучшаем, типа, и не видим эти все карточки.» He also did not notice that a modifier can be sold.
- His main rule (`responsiveness-covers-everything`): «Все должно быть приятным.»

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\Catch\`.

- Core 2.3 interface delta: `Docs/briefs/reports/day2-core23.md`, section 3 (merged into this worktree with the core). In short: the run opens in `Prep` of day 1 with 0 coins and `State::giftPending`; after the starting bait pack is picked the core emits `GiftOffered`; `GiftOffers()` returns three cards (`name`, `text`); `TakeGift(index)` takes one; `StartDay` returns `Locked` while the gift is pending. During the day, when the points reach the norm, the core emits `TargetReached` (with `secondsSaved`, `dayLength`, `payoutTime`) and ends the day at once: every rod pulls, every hooked fish lands and counts, then shipping and the summary as before. `DaySettled` carries `payoutBase`, `payoutTime`, `interest`, `secondsSaved`, `dayLength`; `payoutOver` is always 0. `TimeBonusNow()` gives the bonus the player would get if the norm were reached now. `State::firstShop` is always false.
- The view today assumes the shop before day 1: `CatchHud.cpp` reads `St.firstShop` (target words "TARGET DAY 1:"), `catch.CheatRun` buys charms "in the first shop" (`CatchSession.cpp:550`), and the proof scripts drive that shop (`SourceArt/Text/run_text_proof.py`, `Saved/core22_run_check.sh`, `Saved/core22_run_feedback.sh`).
- The catch is shown in three places: the scales on the raft (`CatchLookScales.cpp`; the pan plate and the weight number follow them, `CatchHud.cpp:649`), the small scales in the coins panel when the raft is off the screen (`CatchHud.cpp:650`, `CatchUiHud.cpp:160-164`), and the day summary panel (`CatchUiPanels.cpp`).
- Shipping: fish leave the bucket on the raft one by one and coins pop up (owner decision: "the catch is sold at the end of the day, fish are shipped one by one"). When the camera is deep, the raft is off the screen and the player sees only numbers.
- The camera travels between the surface and the depth with the player's rods; find where the view decides the camera depth.
- The bait pack window (the choice of bait from a pack) is the model for the gift window.
- Selling a modifier: a `SELL` button appears after the player selects a modifier on the shelf (`CatchHud.cpp:1668`, `:2117`).
- Text layer, proofs, gate, port 8000 note: as in `day2-view-menu-fixes.md`, section 2. The reference numbers of the proofs will change with core 2.3; record the new ones.
- Part 1 is done and committed in this worktree. Read its report first: `Docs/briefs/reports/day2-view-menu-fixes.md`. It added the title and menu layer (`CatchHudMenu.cpp`, `CatchUiMenu.*`, `catch.Menu 0|1|2`; PLAY on the title starts the run that already runs under the title), the shop hover cards (`CatchHudShopTip.cpp`, `Ui::PaintShopTip`), `Ui::PaintFishPicture`, the shop hint `Click a charm on the shelf to sell it.`, the cheats `catch.FewWormsRun`, `catch.BiteRun` and the probe `catch.BiteProbe`.
- The game's word for a modifier is `charm`. Keep it in every player text.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Start of the run

1. Order: title screen, `PLAY`, the starting bait pack (as now), then the gift window, then the preparation of day 1. The gift window uses the look and the motion of the bait pack window: three large cards (name, effect line from the core, a simple picture drawn by the painter: two rods / a sharp rod with two hooks / a line going deeper), header `PICK YOUR START`, sub-line `One is free.` A card answers hover and click like the pack cards. After the pick, the card flies to what it changes (the new rod appears on the raft, the rod flashes, the depth strip opens). Build the window as a reusable "pick one of N cards" window (header, sub-line, a list of cards with name, text and picture): the director may approve a second use (a free charm after each day), so nothing in it may assume the three gift cards.
2. Remove what the first shop needed in the view only where it now misleads (target words, hints). Do not delete code that the legacy table mode may still use; leave it unreachable and say so.
3. `catch.CheatRun` gives its charms directly at the start (no shop). Add `catch.Gift <i>` for proofs. Update the proof scripts to the new start and print the new reference numbers.

The day that ends at the target

4. The day clock shows the stake while the day runs: next to the time, the coins of the time bonus the player would earn now (`TimeBonusNow()`), for example `+3` with a coin, stepping down as the day passes. Each step down answers with a small tick and motion.
5. `TargetReached` is the big moment of the day: the scales tip with the full "first overweight" effect that already exists, a short stinger sound, the words `TARGET REACHED` and `N s saved`, then every rod pulls at once (the core does it) and the last fish fly to the bucket. Give the moment about one second of emphasis before the shipping begins; do not freeze the game.
6. If the day runs out below the norm, nothing changes from today.

One flow of points

7. During the day there is exactly one number for "catch against target", in one place: the scales. When the raft is on the screen, the number lives on the raft scales; when the raft is off the screen, the small scales in the coins panel carry it. Never both at once, and both use the same words, the same number format and the same motion when a fish lands. Each landed fish sends its `+N pts` from the fish to that one place.
8. At the end of the day the camera returns to the surface first (a quick, eased move, about 0.6 s), then the shipping starts, so the player always sees the fish leave the bucket.
9. The summary counts in this order, each step with its own sound and motion, and the player can skip with a click (everything jumps to the final state):
   - a. `CATCH`: the number counts up from 0 to the day points while the fish are shipped (tick per fish, faster for many fish; at most about 4 s in total).
   - b. `TARGET`: the number appears at once, with a stamp. Then the verdict (`PASSED` / `RUN OVER`).
   - c. `PAYOUT`: coins drop one by one: 1, 2, 3, 4.
   - d. `TIME SAVED: N s`: its coins drop one by one. A row with 0 coins is shown greyed, without drops.
   - e. `INTEREST`: its coins drop one by one.
   - f. Modifier coins (rows that pay after the day): one by one, each from its modifier on the shelf.
   - g. The total flies to the coin counter, which counts up.
   Do not show the same number twice on the summary (for example the catch as a row and again inside a sentence).
10. Words: use `TARGET` for the norm everywhere the player reads it, `CATCH` for the day points, `TIME SAVED` for the bonus. One term per thing.

Seeing what you own

11. The player can see every permanent upgrade without a purchase: the fish table (per fish kind: card level as points per kg, chum level) must be reachable in the preparation and in the shop with one click, and the rod cards show the rod level. Find the existing prize table and make it this table; do not build a second one.
12. Selling: the shop already shows the hint `Click a charm on the shelf to sell it.` (part 1). Make the answer to that click impossible to miss: the selected charm on the shelf shows `SELL +N` with a coin, large, with motion.
13. Big numbers. Points and coins may grow to seven digits after the next balance pass. Every place that shows points or coins (scales, coins panel, rod card, pop-ups over fish, summary, shop prices) must hold 9 999 999 without overlap or clipping. Use one number format everywhere: a thin space between thousands (`12 345`); do not shorten to `K` or `M`. Check the layout with a view-only override of the shown numbers (a console command for proofs; it must not change the core state).

## 4. Work by blocks (in this order; note progress in the report after each block)

1. The game runs on core 2.3: start, gift window, cheat and proof scripts, no first shop (decisions 1-3).
2. Day clock stake and the `TargetReached` moment (decisions 4-6).
3. Camera to the surface, shipping, the summary in order (decisions 8-10).
4. One number for the catch (decision 7).
5. Fish table, selling, big numbers (decisions 11-13).
6. Russian rows for every new text; regenerate the table. The sea label of a locked zone (`TWILIGHT - LOCKED`, `Buy Deeper Line`) stays English in the Russian frames today: put it through the text layer.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- English proofs run from a clean start through gift, day 1, summary, shop, day 2 with 0 errors; the report gives the new reference numbers (events, points, coins after day 1) for the plain day and the flow run.
- Russian screens run at 1280x720: 0 missing keys, 0 errors.
- Frames you looked at yourself, English, 1280x720 (and 1920x1080 for the gift window and the summary): the gift window, the pick moment, the day clock with the stake, the `TARGET REACHED` moment with the camera deep and with the camera at the surface, three frames of the summary in order (catch counting, payout coins dropping, total flying), a day that failed, the fish table, the selected charm with `SELL +N`, one day frame and one summary frame with seven-digit numbers. One Russian frame each of the gift window and the summary.
- A real-time run (not stepped) of one full day at depth shows the camera return and the shipping from the bucket; say in the report how you checked it.
- The report lists every new player text and every place where the same number was shown twice before.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`; do not kill a process you did not start.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Catch/Saved/*.sh` proof scripts, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**` or `Catch/CoreTests/**`. If the view needs something the core does not give, write it into the report and work around it.
- Do not open the editor to change assets or the level. No new assets, no downloads, no fonts, no generated pictures or sounds. Reuse the existing sounds.
- Do not invent rules or numbers of the game. A question for the game director goes into the report; continue with what needs no decision.
- Do not delete or hide old work. No commits, no pushes. Do not edit `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate.
- All game text is English; Russian only through the table outside `Catch/`.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-core23.md`, in English. Sections: what changed by block, new reference numbers, new player texts, duplicates removed, frames with paths, checks with their output lines, not checked, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
