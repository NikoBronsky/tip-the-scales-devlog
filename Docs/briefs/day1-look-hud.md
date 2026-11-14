# Brief: day one, look step 2 — the HUD, the panels and the shop look like the prototype

Status: written on 12.11.2026 at 11:16, handed out at 11:23 (owner's clock). Executor: Opus, effort high. Work folder: the worktree `F:\UE\EpicJam_ui` (branch `track/ui`). No interactive Unreal Editor in this task.

## 1. Goal

The owner compared the Unreal build with the browser prototype. His words:

- «Сейчас, если честно, все выглядит очень плохо. Насколько это супер круто выглядело в нашем артефакте, настолько сейчас все ужасно выглядит.»
- «если артефакт у нас выглядит практически как готовая игра, то что получается сейчас, пока выглядит очень плохо.»

The look reference ranks the gaps. Gap 3: the HUD is plain text. Gap 4: the shop is a list of text buttons. Gap 6: HUD parts are missing. Gap 9: no popup words. This step closes these gaps. Another executor works on the sea, the fish, the bite and the catch in `F:\UE\EpicJam` at the same time. You never write there.

## 2. What exists now (verified by the orchestrator)

- **The look reference: `Docs/design/look-reference.md`.** It describes the approved prototype in words, hex colours, sizes on a 1280x720 reference screen and timings. It is your only source for the look. You never see the prototype and you do not look for it. Your parts: 2 (palette), 5.6 (bait icons), 6 (HUD, thirteen parts), 7 (panels: shared look, pack choice, summary, fee paid, run over, victory, shop, title), the HUD rows of 8 (feedback), 10 (ranking and gaps).
- The HUD code: `Catch/Source/Catch/CatchHud.h` (163 lines) and `CatchHud.cpp` (1,098 lines). `UCatchHud` is a `UUserWidget`. The widget asset `Catch/Content/UI/WBP_CatchHud.uasset` is its child and holds a small skeleton with bound names (`MoneyText`, `DayText`, `TimeText`, `FeeText`, `CatchText`, `RodStrip`, `Panel`, `PanelBox`, `FloatLayer`). The script `Catch/SourceArt/SceneScripts/build_hud_widget.py` made the asset inside the editor. The C++ fills everything else at run time: rod cards, counters, phase panels, pack choice, shop buttons, floating numbers, the fish hover card.
- Everything works by function. Every button sends a core command. A whole run can be played: preparation, day, pack choice during the day, shipping, summary, fee, shop, next day. It looks like plain text on flat boxes.
- The rules core and its state: `Docs/design/core-interface.md`. The session object `UCatchSession` (`CatchSession.h`) owns the core, sends events (`OnEvent`) and has one console command per action.
- Console commands for runs without a player (from the earlier reports `Docs/briefs/reports/day1-playable.md` and `day1-integrate.md`): `catch.NewRun [seed] [Soft|Normal|Hard]`, `catch.StartDay`, `catch.Pull <rod>`, `catch.AutoPull <s>`, `catch.Depth <rod> <d>`, `catch.Bait <rod> <name|none|next>`, `catch.OpenPack <i>`, `catch.Pick <i>`, `catch.Skip`, `catch.Pay`, `catch.NextDay`, `catch.Buy <upgrade|card|charm|pack|worms> <i> [rod]`, `catch.Sell`, `catch.Reroll`, `catch.Dump`, `catch.Hover <fish id|-1>`, `catch.HudMoney`, `catch.HudLayout`, `catch.After <s> <cmd>`, `catch.Shot <name>`, `catch.Record <name> <s> [fps]`. Simulated clicks (`catch.SimClick`) are flaky; use console commands.
- A game run for a frame: `"F:/UE_5.9/Engine/Binaries/Win64/UnrealEditor.exe" "F:/UE/EpicJam_ui/Catch/Catch.uproject" -game -windowed -ResX=1920 -ResY=1080 -nosplash -unattended -benchmark -fps=30 -log -ExecCmds="catch.NewRun 42,catch.After 1 catch.StartDay,catch.After 6 catch.Shot ui_test,catch.After 8 quit"`. The log: `F:\UE\EpicJam_ui\Catch\Saved\Logs\Catch.log`. Frames: `F:\UE\EpicJam_ui\Catch\Saved\ue_mcp\`. The frames include the HUD.
- The worktree is made and its editor target is built. Live Coding is off there (`Catch/Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini`). Keep it off: with Live Coding on, your runs would block the builds of the other executor.
- The owner has not answered yet whether a font file from outside may be used. Until he does, the font is the font of the engine.
- Owner's decisions that bind this step (`CLAUDE.md`, section 2): one scene and one screen; the shop is an interface layer between days; polish and feel are the main thing, every action of the player answers with motion, sound and effect; all text for the player is English.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. **No new assets.** Build the look in C++: UMG widgets made in code and Slate brushes (rounded boxes with outlines, a second box as a shadow), custom paint for rings, bars, dots and icons. No textures, no imported files, no material assets. Then this step needs no editor and runs beside the other executor. If a part cannot be done without an asset, leave it, and name it in the report.
2. One style kit in new files `Catch/Source/Catch/CatchUi*.{h,cpp}`: colours by name from the reference, the panel, the five button styles, the outlined popup word, the fonts, the scale. The HUD code uses the kit. No colour or size literal stays scattered in `CatchHud.cpp`.
3. Font: the default font of the engine, in its heaviest faces, through one function of the kit. A later change to another font must be one line.
4. Scale: the sizes of the reference are pixels on 1280x720. Multiply by viewport height / 720.
5. Colours are checked with numbers. Flat HUD colours must match the reference within 6 (of 255) per channel in your frames. Mind the sRGB conversion.
6. Icons of baits (5.6) and small glyphs are drawn in code from the shapes that the reference gives. If an icon takes too long, draw a rounded chip in the rarity colour with the first letter, and say so.
7. The widget asset stays as it is. If you must change the skeleton, do it in C++ at construction. Do not open the editor.
8. Words on screen: English, as the look reference gives them. The reference marks its own English wording with NEW. Use it and list these words in the report, so that the owner can change them.
9. Sound: do not play sounds in this step. Another executor builds the sound class. Declare one static multicast delegate in the kit, `OnUiSound(FName)`, and broadcast the names of `Catch/SourceArt/Sound/README.md` at the right moments (Tick, Deny, Buy, Flip, Rustle, Tear, Clasp, Stamp, Coin, CoinArrive, Chord). The orchestrator joins the two after the merge.
10. Motion from the reference is part of the look: the panel rises into place, the screen dims, buttons lift on hover, the money rolls, the stamp turns, the popup words rise and fade. Use the timings of the reference.
11. The sea and the fish in your frames stay in the old temporary look. That is expected: the other executor changes them on another branch.
12. Keep every function. Every button and every console command works after your change as before.

## 4. Work, in this order

Block A — the kit and the day screen (gaps 3 and 9).
1. The style kit.
2. Day panel with the clock bar and the day rule box (6.1), money panel with the roll and the fee coverage bar (6.2), rod card with bait chip, level dots, state word, value, third line and hang ring (6.4), the Start Day button (6.9), the banner (6.11).
3. Popup words in the outlined style for every HUD row of section 8: values, "CHAIN xN", "got away", bait notices, the ship numbers.

Checkpoint: hand back after block A with the frames `ui_1_prep.png` and `ui_2_day_hooked.png`, the colour table and the gate result. The orchestrator checks the frames, shows one to the owner and resumes you.

Block B — the panels (gap 4). The shared panel look (7.1), pack choice with the fan of cards (7.2), summary (7.3), fee paid with the stamp and the coins (7.4), run over (7.5), victory (7.6).

Block C — the shop (gap 4). Section 7.7 in full: charm cards with class colour and rarity, rod tiles, bait tiles, fish cards, upgrade tiles, locked tiles, the tomorrow box, the notice bar, reroll and sale.

Block D — the missing HUD parts (gap 6). Depth gauge (6.3), bait menu (6.5), bait box with the pack button (6.7), charm shelf (6.8), hint bar (6.10), ship bar (6.12), the fish hover card in the new style (6.13). The title screen (7.8) only if time is left.

If time runs out, hand back complete blocks. Do not leave a block half done.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` is green in the worktree.
2. Frames in `F:\UE\EpicJam_ui\Catch\Saved\ue_mcp\`, 1920x1080, from a `-game` run: `ui_1_prep.png`, `ui_2_day_hooked.png`, `ui_3_pack_choice.png`, `ui_4_summary.png`, `ui_5_fee_paid.png`, `ui_6_shop.png`, `ui_7_run_over.png`, `ui_8_day_two_rods.png`.
3. One full run by console commands from the preparation of day 1 to the preparation of day 3 passes, and `catch.Dump` equals the HUD after every day.
4. The colour table of decision 5 in the report.
5. You compared each of your frames with the text of the reference yourself and wrote what still differs.

## 6. Boundaries

- Files you may create or change, all inside `F:\UE\EpicJam_ui`: `Catch/Source/Catch/CatchHud.*`, `Catch/Source/Catch/CatchPlayerController.*`, new files `Catch/Source/Catch/CatchUi*.{h,cpp}`, `Docs/briefs/reports/day1-look-hud.md`.
- Not yours: `CatchSeaView.*`, `CatchTackleView.*`, `CatchSession.*`, `CatchGameMode.*`, `CatchWorld.h`, `CatchLook*`, `Catch/Content/**`, `Catch/Config/**`, `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/SourceArt/**`, the documents. If you need an accessor in a file that is not yours, write the need into the report and work around it.
- Never write into `F:\UE\EpicJam` or `F:\UE\EpicJam_core`. Never open the interactive Unreal Editor. Never switch Live Coding on.
- Build with `powershell -NoProfile -ExecutionPolicy Bypass -File Tools/build-editor.ps1 -Project F:\UE\EpicJam_ui\Catch\Catch.uproject`. No game run of yours may be open during your build. If the build waits for a mutex, the other executor is building: wait. Keep your game runs short: the other executor measures frame times on the same machine.
- You do not invent the game or the look. A value the reference does not give: write the question, choose the plainest reading, mark it in the report.
- Nothing from outside: no downloaded asset, font, texture or sound.
- Do not commit, do not push, do not delete files.

## 7. Report

English, `Docs/briefs/reports/day1-look-hud.md`, at most 35 lines: blocks done; the colour table; the list of NEW words used; what still differs from the reference, most important first; decisions you changed; needs outside your files; questions.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Look questions the reference does not answer: write them down; the orchestrator asks the owner.
