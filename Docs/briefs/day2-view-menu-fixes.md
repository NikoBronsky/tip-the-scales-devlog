# Brief: view, part 1 - title screen and main menu, bait box, the escape warning, the bite at the hook, shop explanations

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 11:10 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`, base `a037129` (clean). This brief lives only in the main checkout.

## 1. Goal

Five player-facing improvements from the owner's playtest of build 7, none of which depends on the core changes that run in parallel. A second part (the start gift window, the day that ends at the target, the summary that counts in order, one flow of points) follows after the core lands; do not start it.

Why it matters. The owner (game director) said on 13.11, by voice:

- Menu (`title-and-main-menu`): «нам нужна заставка и какое-нибудь простенькое главное меню, типа игра, настройки и выход. Просто чтобы какое-нибудь простое с прыгающими буквами, с названиями игры, какими-нибудь красивыми, вот, с тем же самым морем и шумом.»
- Bait (`empty-bait-cards-disappear`): «наживка, когда она по нулям, то есть она кончается, у нас остаются пустые карточки слева, где наживка. По-хорошему они должны исчезать … И … когда их нет, надо писать большими буквами, что нет наживки. … Я вот сейчас … про это совершенно забыл.»
- Red bar (`red-bar-on-hooked-fish-unclear`): «Есть, короче, непонятки, что означает красная полоса там, где рыба, когда она ловится.»
- Bite (`fish-bites-at-the-hook`): «Рыба не всегда клюет крючок. Она иногда рядом с ним где-то клюет.»
- Shop (`shop-explains-items`): «в интерфейсе не все понятно, что такое прикормка, например. … надо выделять какое-то пояснение для того, чтобы игрок понимал, что такое прикорм, что прикорм это делает то-то».
- His main rule (`responsiveness-covers-everything`): «Все должно быть приятным.» Every action answers with motion, sound and effect.

## 2. What exists now (verified by the orchestrator; verify again before you rely on a line number)

Paths are under `F:\UE\EpicJam_ui\Catch\`.

- No menu. `UCatchSession::OnWorldBeginPlay` (`Source/Catch/CatchSession.cpp:29`) starts a run; the starting bait pack opens by itself. Console commands are `FCatchCommand` objects (`CatchSession.cpp:538` is `catch.NewRun`; `catch.StartDay`, `catch.Pull`, `catch.Pick` follow). Scripted proof runs start the game with `-ExecCmds="catch.NewRun 42,..."`.
- The HUD is a painter HUD (`CatchHud.cpp`, `CatchUi*.cpp`): text and shapes are drawn every frame. The start button is `Ui::PaintStartButton` (`CatchHud.cpp:709`).
- Text layer: every drawn text passes `CatchUi::L` at paint time (`CatchUiText.cpp`). A new player text needs a row in `Docs/design/text-ru.tsv` and a regenerated `Source/Catch/CatchTextRu.inc` (`python -X utf8 Catch/SourceArt/Text/gen_text_ru.py`, check with `--check`). In Russian, `catch.TextMissing` must report 0 missing keys. No Cyrillic letter may appear under `Catch/` (the generator writes escapes).
- Sound: about 30 action sounds by name through `FCatchLookSound` (`CatchLookSound.cpp`; list in `SourceArt/Sound/README.md`). A sea and music bed plays from the start of the level. Component volumes are set in `CatchTackleView.cpp:298` and `:487`.
- Bait box (the cards at the left): `PaintBaitBox` (`CatchUiSide.cpp:162`), tiles in two columns (`:183`), hover card (`:199`). The rod's bait menu: `PaintBaitMenu` (`:245`).
- Escape warning of a hooked fish: the HUD rod card (`CatchHud.cpp:429`, `bSlipping`), the tackle view (`CatchTackleView.cpp:1214` and `:1546`), the fish look (`CatchLookFish.cpp:358`, `D.bWarn`). The core gives `escapeSeconds` per hook and `pace.escapeWarning`; a rod with no-escape never warns.
- Bite: `CatchLookFish.cpp:498` says the core aims the mouth at the hook and the fish nibbles there; `GetMouth` (`CatchLookFish.cpp:761-771`) puts the mouth at 0.52 of the body length along the body axis.
- Shop: chum tile `ChumTile` (`CatchUiPanels.cpp:947-965`), section label "CHUM" (`:1019`). Core meaning of the two fish upgrades (`Docs/design/economy-v2.data.json`, `shop`): chum = the wanted count of that fish kind x (1 + 0.5 x level), from the next day; fish card = the award (points per kilogram) of that kind + 1.
- Proof runner: `python -X utf8 Catch/SourceArt/Text/run_text_proof.py <plain|flow|probe|screens> <en|ru> <1280|1920>`. Reference numbers on this base, seed 42: plain day `ev: 199`, 37 pts against 20, coins 13; flow run `ev: 206`, 25 pts, coins 11; 0 errors. Frames land in `Saved/ue_mcp/`.
- Two `-game` runs that overlap log two errors about `127.0.0.1:8000`. That is the editor link port, not a game bug. Run proofs one at a time.
- The menu part of the historical brief `F:\UE\EpicJam\Docs\briefs\day2-menu-language.md` is background only; this brief replaces it. There is no language switch.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Title screen and main menu

1. One screen, drawn by the painter HUD over the live sea of the same level. The sea, the fish and the sound bed run behind it. No second map.
2. The game title comes from one constant (for example `Ui::GameTitle`), today `CATCH`. The owner will name the game later; the change must be one line. Each letter bounces on its own phase (a wave through the word), reacts to the cursor, and the word lands with a splash when the screen opens. Make it pretty: this is the first thing the jury sees.
3. Three buttons: `PLAY`, `SETTINGS`, `EXIT`. `PLAY` starts a new run (the bait pack opens as now). `EXIT` quits the game. Buttons answer hover and click with motion and sound, like the rest of the HUD.
4. `SETTINGS` is a small panel: `MUSIC` and `SOUND` volume (0-100 % in steps of 10) and `FULLSCREEN` on / off. Store them in the user's config and read them at start. Find the simplest reliable way to scale the music bed and the action sounds separately.
5. `Esc` during a run opens the same screen with `RESUME` in place of `PLAY` and pauses the day. `Esc` again resumes. After "run over" the player can also reach the menu.
6. Scripted runs must work as before: a run started by `catch.NewRun` (console or `-ExecCmds`) closes the menu. Add `catch.Menu 0|1` for proofs. No logos anywhere.

Bait box

7. A bait kind with 0 pieces has no tile. The remaining tiles close the gap with a short move.
8. When the player holds no bait at all, the bait box shows a big sign `NO BAIT` with one line under it: `Bare hooks catch little.` The sign pulses once when the last bait is used up, with a sound. It shows in the preparation and in the day.
9. Check what a rod card shows when its bait ran out, and make it say the same thing in the same words (`BARE HOOK` if that term is already used; one term per thing).

Escape warning ("the red bar")

10. First find out what the owner saw: make a probe frame of a hooked fish in its warning window and list every red element near the fish, the line and the rod card, with what each one means today.
11. Then make it read without a tutorial. Target (adjust to what you find): red means one thing only - "this fish is about to escape". The escape time is one shrinking element near the hooked fish (it empties as the time runs out), calm while there is time, red and pulsing in the warning window, with the word `SLIPPING!` next to the fish and on the rod card in that window. Anything else near a hooked fish that is red today (for example the lucky hit arc) gets another colour. The first time in a run that a fish enters the warning window, one hint line appears near the rod card: `Pull before it slips away.`

Bite at the hook

12. Reproduce with a probe: log the distance between the drawn mouth of a nibbling or hooked fish and the drawn hook for several kinds and sizes (small Sprat, Sea Bass, the largest kind you can reach), also while the line swings and on a second hook. Fix the cause so the mouth sits on the hook in every case. Report the distances before and after.

Shop explanations

13. Every shop item says in plain words what it does: one short line on the tile itself and the full text in its hover card. Derive each line from the core data; do not invent or change a rule. Chum: `More <Fish> in the sea from tomorrow.` plus the level effect (`+50% per level`). Fish card: `<Fish> give +1 pt per kg.` Audit all items (rods, sharpening, deeper line, bigger shelf, packs, worm tin, reroll, chum, fish cards, modifiers) and put a table "item - line on the tile - hover text" into the report. Fix the items a new player cannot understand; chum is the owner's example.
14. Chum tiles and fish cards show their fish through one function (for example `Ui::PaintFishPicture(kind, rect)`) that today draws the fish the way the view already draws it. Painted fish pictures will replace the drawing inside that function later; use the same function in the fish hover card.

## 4. Work by blocks (in this order; report after each block in the report file)

1. Bait box (decisions 7-9).
2. Bite at the hook (decision 12).
3. Escape warning (decisions 10-11).
4. Shop explanations (decisions 13-14).
5. Title screen, menu, settings, pause (decisions 1-6).
6. Russian rows for every new text; regenerate the table.

Do not touch in this part: the day summary, the scales on the raft and in the coins panel, the first shop flow, the payout rows. They change in part 2.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- English proofs keep the reference numbers of section 2 (plain and flow), 0 errors.
- Russian screens run at 1280x720: 0 missing keys, 0 errors.
- Frames you looked at yourself and list in the report, English, 1280x720 and 1920x1080 where layout matters: the title screen (two frames of a real-time run that show the letters at different heights), the settings panel, the pause screen, the bait box with a missing kind, the `NO BAIT` sign, a hooked fish before and inside the warning window, a bite close-up for three fish sizes, the shop with the new lines, the chum hover card. One Russian frame each of the menu and the shop.
- The report lists every new player text.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Builds from several worktrees wait for each other (`-WaitMutex`); another executor compiles the core module in `F:\UE\EpicJam_core` from time to time. Do not kill a build you did not start.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Catch/Config/**` if settings need it, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**` or `Catch/CoreTests/**`: the core changes in parallel in another worktree. If you need a core fact that is missing, write it into the report.
- Do not open the editor to change assets or the level. No new assets, no downloads, no fonts, no generated pictures or sounds. Reuse the existing sounds.
- Do not invent rules, numbers of the game, or new screens beyond this brief. A question for the game director goes into the report; continue with what needs no decision.
- Do not delete or hide old work. No commits, no pushes. Do not edit `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate.
- All game text is English; Russian only through the table outside `Catch/`.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-menu-fixes.md`, in English. Sections: what changed by block, the red elements you found and what each meant, bite distances before and after, the shop table, new player texts, frames with paths, checks with their output lines, not checked, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
