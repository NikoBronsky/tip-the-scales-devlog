# Brief: view, part 5 - a picture on every shop item

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 18:30 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message. This brief lives only in the main checkout.

## 1. Goal

Every item the player can buy or take shows its own picture, in the style of the fish pictures: the 12 baits, the two bait packs, the worm tin, the four upgrades, the three start gifts and the 53 charms. A shop item must look like a thing you buy, not like a text tile.

Why it matters. The owner (game director), decision `shop-items-get-pictures`, 13.11: «скажи а ты картинки добавишь к пачкам с наживкой, к улучшениям рыбы и всему остальному? чтобы это выглядело как покупная штука?» He judges every frame against a browser prototype he liked. Fish cards and chum already show the fish pictures; everything else in the shop is a painter icon or a chip with a word.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\`.

- The source pictures, made by the orchestrator: `Catch/SourceArt/Items/<id>.png` (RGBA, trimmed, about 300-480 px on the long side) and `Catch/SourceArt/Items/items.json` (for each id: kind `bait`, `pack`, `tin`, `upgrade` or `charm`; the English name exactly as the core names the item; size; aspect). All 72 pictures exist: 12 baits, 2 packs, the tin, 4 upgrades, 53 charms. The folder is not committed yet; the orchestrator commits it with your work. The prompts and the cutter are recorded in the main checkout (`Docs/briefs/item-art-prompts.md`, `Tools/item_cut.py`).
- The fish picture pipeline of part 3, to copy: `Catch/SourceArt/Fish/make_fish_atlas.py` (the atlas and its table), `build_fish_art.py` (the import), `Catch/Source/Catch/CatchFishArt.h/.cpp/.inc`, `FishArtPicture` and `Ui::PaintFishPicture` in `CatchUiPanels.cpp` (about lines 290-353: the atlas cell drawn with `FSlateDrawElement::MakeCustomVerts`). The picture assets are loaded with `ConstructorHelpers::FObjectFinder` in the `ACatchSeaView` constructor, so the cook takes them. Report: `Docs/briefs/reports/day2-view-fish-art.md`.
- Bait: one function draws every bait icon, `FPaint::BaitIcon` (`CatchUi.cpp:657`). Call sites: the pack window cards (`CatchUiPanels.cpp:504`, the pour `:665`), the bait box (`CatchUiSide.cpp:231`), the bait menu (`:301`), the sea hover card (`:470`), the rod card (`CatchUiHud.cpp:316`), the popup (`:739`), a rule icon (`:450`), the gift card badge (`CatchUiFlow.cpp:169`), `CatchUiPanels.cpp:242`.
- Charms: `Ui::PaintCharmChip` (`CatchUiPanels.cpp:1294`) draws a chip with a word from `CharmTag` (`:358`, a table keyed by the core charm name). The chip is used on the shelf (`:1363`), on the shop charm cards, on the reward cards (`CatchUiFlow.cpp:241`, `:541`) and in flight (`CatchHud.cpp:883`).
- Start gifts: `Ui::PaintPickPicture` (`CatchUiFlow.cpp:73`).
- Shop tiles: the packs and the Worm Tin (`CatchHud.cpp:1878-1905`), the upgrades, `Ui::PaintShop` (`CatchUiPanels.cpp:1047`), the hover card `Ui::PaintShopTip` (`:1130`).
- The text layer and the Russian testing table: `Catch/SourceArt/Text/gen_text_ru.py` (`--check`), `run_text_proof.py` (modes plain, flow, fail, big, depth, screens, title, reward). The table: `Docs/design/text-ru.tsv`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. One script `Catch/SourceArt/Items/make_item_atlas.py` packs every picture of the folder into one atlas texture (or two if one is too small) and writes the table the code reads. The cell size is a parameter. A picture keeps at least 256 px on its long side in the atlas, because the reward card shows it large. The script runs again whenever new pictures arrive: a new picture must need no code change.
2. The table is keyed the way the code already finds things: baits by bait kind, upgrades and packs by their core id, charms by the core charm name (as `CharmTag` does).
3. A missing picture falls back to what the game draws today. So a charm added later works before its picture exists. Add the console switch `catch.ItemArt 0|1` (0 = today's painter icons) and one log line at start, `CATCH_ITEMART ...`, that names the loaded assets and the number of pictures.
4. Bait: `FPaint::BaitIcon` draws the picture. If a picture is unreadable at the smallest sizes (the popup, the sea hover card), keep the painter icon there; decide by frames and list where each is used.
5. Charms: the picture is the main thing on the chip. The class colour and the rarity stay readable (a plate, a ring or a frame of the class colour; the rarity mark as today). The word tag stays only for a charm without a picture. On the reward card and on the shop charm card the picture is large; the enlarged chip of part 4 goes away.
6. Packs, the Worm Tin and the upgrades: the tile shows the picture large, with the name and the price as a price tag. The tile must read as a product on a shop counter. Keep the existing hover pose and motion of the cards; a small idle motion is welcome. No bend like the fish.
7. Start gifts: Second Rod uses the picture `new_rod`, Sharp Rod uses `sharpen_rod`, Deeper Line uses `deeper_line` (the bait badge with the count stays).
8. The bait on a hook in the sea and the tackle in the sea do not change in this block.
9. Texture settings as for the fish atlas; the picture must stay sharp on the reward card at 1920x1080.
10. The Russian row for `One is free.` reads badly for the reward («Одна — бесплатно.» while a charm is masculine in Russian). Change the row to «Один выбор — бесплатно.» in `Docs/design/text-ru.tsv` and regenerate.
11. No new player text is expected. If you add one, add its Russian row.

## 4. Work by blocks (in this order; extend the report after each block)

1. The atlas script, the import, the table, the loader, the switch and the log line.
2. Bait pictures everywhere `BaitIcon` draws.
3. Packs, the Worm Tin, the upgrades, the start gifts.
4. Charms: the chip, the shop card, the reward card, the shelf, the flight.
5. Proof frames, the Russian row, checks.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- The proofs pass with 0 errors and 0 missing keys: `run_text_proof.py plain en 1920`, `flow en 1920`, `reward en 1920`, `screens ru 1280`.
- `catch.ItemArt 0` gives today's look with no error; `catch.ItemArt 1` gives the pictures.
- A run with the cooked or `-game` build prints the `CATCH_ITEMART` line with the real picture count.
- Frames you looked at yourself, 1920x1080: the shop with the packs, the Worm Tin and the upgrades; the pack window with bait cards; the bait box and a rod card with bait; the start gift window; the reward window with three charm pictures; the shop charm cards; the shelf with five charms; a charm in flight; the shop with `catch.ItemArt 0`. One frame of the shop at 1280x720. One frame of the shop in Russian.
- The report lists every place where a painter icon stays and why.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py reward en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`. Do not kill a process you did not start. The owner may be playing a packaged build on this machine while you work: do not close any game window you did not open. If the gate reports a time-out of the core tests while the machine is busy, run `Tools\core-test.ps1` alone and report both results.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, the new picture assets under `Catch/Content/UI/Items/` (or next to the fish atlas, your choice), the row named in decision 10 of `Docs/design/text-ru.tsv`, and your report file. Use absolute paths.
- Do not change the pictures themselves. If a picture is wrong (a cut-off edge, a magenta rim, an unreadable shape), write its id into the report; the orchestrator replaces the file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Source/Catch/Catch.Build.cs`, `Catch/Config/**`, the level, other assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game.
- No downloads, no generated pictures of your own, no fonts, no new sounds.
- Do not delete or hide old work: the painter icons and the chip stay in the code as the fallback. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/`. No director, cinema or "cut" wordplay.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-item-art.md`, in English. Create it right after reading, with one status line per block, and extend it after each block. Sections: what changed by block, where a picture is used and where a painter icon stays, the atlas (size, cell, count), frames with paths, checks with their output lines, picture ids with defects, not checked, placeholders that remain, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
