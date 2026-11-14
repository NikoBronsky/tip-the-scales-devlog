# Brief: view part 9a - highlights instead of text, the day-end count, the luck sign, the jellyfish, the frame cap

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. The base is the commit of view part 8 (the orchestrator commits it before you start). This part does NOT depend on core 2.5; another executor works on the core in `F:\UE\EpicJam_core` at the same time.

## 1. Goal

The owner played build 10 and gave notes on feel. Each note below is a recorded owner decision (`Docs/decisions/game-frame.answers.json` in the main checkout `F:\UE\EpicJam`). The owner's rule for the whole game: every action of the player answers with motion, sound and effect.

Owner's words (Russian, verbatim; the owner dictates by voice):

- `eat-highlight-replaces-text`: «Мы не будем писать в описании рыбы, кто кого ест. Мы покажем подсветкой красненьким того, кто будет съедать эту рыбу. И когда у нас модификаторы что-то поменяют, это будет отображаться. Если игрок не догонит, хрен с ним. Мы тогда и место на карточке описания сэкономим, и не будем слишком явно говорить подсказку.»
- `eat-and-bite-highlight-idea` (approved by "Делать обе"): «Когда мы крючок с наживкой опускаем на глубину, было бы неплохо показать, кто будет ее клевать, если будет.»
- `summary-count-still-missing-b10`: «у нас все еще нету эффекта, как конвертируются секунды в очки, в монетки, типа одна, вторая, третья, чтобы цифра менялась с единички до двойки, до тройки и так далее. И с очками то же самое, они должны начисляться на таблице окончания дня как-то. Если игрок нажмет на пробел, окей, пускай он инстантно завершится, но в финале должна быть какая-то награда игроку за его подсчет.»
- `lucky-trigger-visual`: «надо какую-то визуализацию, чтобы игрок видел эффекты ... и раз в 3 раза там увеличился вес, или там что какие-то джекпотики на удачу сработали. Ну, какой-то такой знак. Не обязательно именно джекпот, но какой-то вот знак удачи, чтобы игрок видел, что у него сработал.»
- `jellyfish-ugly-b10`: «Все еще медуза какая-то уродливая плавает.»
- `giants-overlap-shadow-b10`: «у меня был модификатор рыбы-гиганты, и когда рыбы наслаивались друг на друга, была какая-то странная тень на рыбках, это портило шейдер.»
- The owner also reported that the game lags. The orchestrator measured: the game has no frame cap and runs at 150 to 240 frames per second, using about half of a 12-core processor.

## 2. What exists now (verify before you rely on it)

- Hover of a fish in the sea: `CatchHud.cpp:3304-3342` (`UpdateHover`, `HoverFish`), `CatchHud.h:76` (`GetHoverFish`), passed to the sea look at `CatchSeaView.cpp:612` (`Frame.HoverId`). The look draws the outline of the hovered fish.
- The fish hover card writes who eats whom: `CatchHud.cpp:3392` ("Eats on a hook: ...") and `:3393` ("Eaten by: ..."). The data is `FishDetails` of the core (`CatchCore/Public/CatchCore/Game.h:184`: `eats`, `eatenBy`). The fish table row also writes "Eats on a hook" at `CatchHud.cpp:2463`.
- The day-end flow: `CatchHudFlow.cpp:322-367` (`OpenDayFlow`, `BuildFlowRows`: rows PAYOUT, TIME SAVED, INTEREST, charm rows; `R.Coins`, `Ui::Flow::CoinGap`, `CoinFall`), settle at `:369`. The row "TIME SAVED: N s" is one row with its coins (`:348`). Painting: `CatchUiFlow.cpp`. Earlier decisions it had to meet: `day-end-count-with-feedback` ("очки растут числом на глазах, монеты начисляются по одной, итог получает эффект") and `summary-counts-in-order` ("Улов растёт от нуля до очков дня. Норма появляется разом. Монеты выплаты падают по одной: 1, 2, 3, 4."). The owner says he still does not see it in build 10.
- The luck sign: `CatchHud.cpp:2757` (`OnLuckyChance`: popup "LUCKY x3" at the hook, the shelf chip wobbles), cheat frames `catch.HudShotLucky` (`CatchHud.cpp:543`). Core event `LuckyChance` (`CatchCore/Private/Charms.cpp:389`); the core test of the chance is `Catch/CoreTests/core_tests.cpp:1910`. In the owner's run: 9 chain links with Lucky Bite on the shelf, 0 lucky hits (chance of that is about 7 %), so he never saw the sign.
- Jellyfish: `CatchLookSea.cpp:226-237` (three jellyfish), `:362-380` (motion, pulse), the reference numbers in `Docs/design/look-reference.md:246`. The report of day 1 says: "the jellyfish bell is a flat ellipse, because the card shapes have no dome. The tentacles are single strokes, so the jellies look a little like stools." Frame with the present jellyfish: `F:\UE\EpicJam\Saved\owner\lucky_sign_now.png` (bottom right).
- Frame rate: no cap anywhere. The only frame-rate code is the bench (`CatchSeaView.cpp:1126-1134`, `StartBench` sets `t.MaxFPS 0` and `r.VSync 0`).
- Term notes beside a hover card: in the shop they cover a buyable tile. Frame: `F:\UE\EpicJam_ui\Catch\Saved\ue_mcp\text_classes_en_1920_card_notes.png` (the notes of Bare Steel stand over DEEPER LINE). Part 8 fixed this only for the shelf tip.
- The bait box tiles and the bait menu use the old rarity colours (green, blue, gold); the cards use the metals of part 8 (`CatchUiPanels.cpp` `PaintRarityFrame`).

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Red highlight. While the cursor is on a fish (free or hooked), every visible fish that can eat it gets a red outline of the same kind as the hover outline. "Can eat it" comes from the core (`FishDetails.eatenBy` of the hovered fish, or a better core query if one exists), never from a table in the view. If the core answer ignores a charm that is on the shelf (for example Free Feast), do not rebuild the rule in the view: use the core answer and write the exact case into the report as a core request.
2. Green highlight. While the player drags the depth of a rod, or holds the cursor on a hook or on a bait in the bait box, every visible fish that would bite that hook gets a green outline: kinds the bait of the hook catches today (core data; day rules and charms included, for example "Fish ignore Worm"), which live at the depth where the hook will hang. A bare hook uses the bare hook rules of the core. If no fish would bite, nothing lights up: that is the answer the player reads.
3. The two colours never mix on one fish: the hover outline wins, then red, then green. Outlines fade in and out in about 0.12 s. Cost: no per-frame allocation, no extra pass per fish (reuse the outline path of the hovered fish; up to 1000 fish swim).
4. Text. Remove the lines "Eats on a hook" and "Eaten by" from the fish hover card. Keep the row in the fish table (`CatchHud.cpp:2463`) as a reference. Keep the Russian rows in the table file (old text is not deleted), unused rows are fine.
5. Day-end count. First record what the player sees today: a frame every 0.25 s through one whole day-end flow of a passed day with time saved, and look at them. Then make the flow read as a count, in this order: (a) the catch grows from 0 to the points of the day as a rolling number, with a rising tick sound; the target appears at once; (b) each coin row counts one coin at a time: the coin flies, the number of the row goes 1, 2, 3, the pitch of the sound rises per coin; on TIME SAVED the seconds count down while the coins count up; (c) the total gets a reward: the number pops, a burst, a longer sound, the coins fly to the wallet and the wallet counts up. Long rows speed up so the whole flow stays under about 6 s. Space or a click ends the count at once at the final numbers; the reward of the total still plays (short). All numbers come from the events of the core; no rule changes.
6. Luck sign. When `LuckyChance` fires: a gold flash and a ring at the hook, a short burst of sparks, the words bigger than now, the weight on the rod card rolls from the old value to the new one, the shelf chip of the charm flashes, a sound of its own (an existing sound with a pitch change; no new files). The same sign serves every charm that fires by chance. In the report: a cheat run with Lucky Bite, the number of chain links and of lucky hits (expect about 1 in 4).
7. Jellyfish. Redraw it in code, no assets: a dome bell (half ellipse with a rim and a soft inner glow, translucent), short frilled arms under the bell, 6 to 8 soft tapered tentacles that trail and wave with a phase lag, the bell pulse drives the tentacles. Keep the numbers of `look-reference.md:246` (place, size, speed, colours) unless the look needs a change; say which. Judge it in frames from a running game, at normal size and enlarged.
8. Frame cap. At game start set the frame cap to 120 (console variable `t.MaxFPS`), not during the bench. Log once `CATCH_PERF frame cap 120`. Add a light measure: every 10 s of play log one line `CATCH_PERF 10 s: avg <ms>, max <ms>, frames over 33 ms: <n>`. No settings line for it.
9. Term notes beside hover cards in the shop: do not cover a buyable tile while a free place exists (reuse the way of part 8 for the shelf tip).
10. Bait box tiles and bait menu: rarity colours follow the metals of the cards (copper, silver, gold; Common plain).
11. Shadow on overlapping fish. With the charm Giants (`CatchCore/Private/CharmRows.inc:263`, "Half as many fish. Every fish is twice as big and weighs x2.") fish overlap and a dark shadow shows on the fish. Reproduce it first (a run with Giants and many fish; take a frame where two fish overlap), find the source (fish casting or receiving shadows, contact shadows, ambient occlusion, the outline or a depth write of the fish material), then remove it so that a fish in front of another fish looks the same as a fish alone. Core 2.5 brings a fish ten times larger (Whopper), so also check a fish at scale 10 if you can scale one by a debug command. Report the cause in one sentence.

## 4. Work by blocks

Order: 5 (day-end count), 1-4 (highlights and text), 11 (shadow on overlapping fish), 6 (luck sign), 7 (jellyfish), 8 (frame cap), then 9 and 10 if time is left. After each block extend the report.

Files you will touch: `Catch/Source/Catch/CatchHud*.cpp/.h`, `CatchUiFlow.*`, `CatchUiFx.*`, `CatchSeaView.*`, `CatchLookFish.*`, `CatchLookSea.*`, `CatchUiPanels.*`, `CatchUiSide.*`, `CatchSession.*` (cheats and proof steps), `Catch/SourceArt/Text/run_text_proof.py` or a proof script of your own, `Docs/design/text-ru.tsv` with the generated `CatchTextRu.inc`.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- Proof runs with 0 errors and 0 missing keys in English and in Russian.
- Frames, looked at by you, from a running game: red outlines on the eaters of a hovered fish; green outlines while a rod depth is dragged; no outlines when nothing would bite; the fish hover card without the two lines; a sequence of the day-end count (catch rolling, coins 1-2-3 on TIME SAVED, the reward of the total); the count ended by Space; the luck sign; two overlapping fish with Giants before and after the fix; the new jellyfish at normal size and enlarged; the shop with notes that do not cover a tile; the bait box in metals.
- The log shows `CATCH_PERF frame cap 120` and the 10 s lines; the report gives avg and max frame time in a day with many fish.
- The report lists every changed text before and after, every timing you chose, and every decision you changed.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py classes en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms ru 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

The owner plays and shows the packaged game on this machine. Run game proofs one at a time, as few as you need, grouped near the end of a block. Never leave a proof game running. If the orchestrator writes "pause", stop every build, test and game at once and go on with reading and editing only, until "resume". Builds use `-WaitMutex`. Do not kill a process you did not start; do not touch `Packaged/`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, charm rule texts, `Catch/Source/Catch/Catch.Build.cs`, config files, the level, assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game. A highlight shows a rule the core already has. Timings, colours and sizes of effects are yours (list them).
- No downloads, no generated pictures, no fonts, no new sound files. No deleting or hiding of old work (the two text lines of decision 4 are removed by an owner decision). No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The Russian word for a charm is «модификатор».
- Nothing from the browser prototype (code, art, sound) goes into the game. Do not copy the look, names or sounds of any existing game.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-feel-highlights.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
