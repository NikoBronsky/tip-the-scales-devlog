# Brief: view, part 8 - term colours by class, the word "reusable", sound in the background

Executor: Opus, effort high. Written by the orchestrator on 14.11.2026 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`, base `fda7103` (view parts 5-7). This brief lives only in the main checkout.

## 1. Goal

Three small fixes after the owner's first minutes in build 10. He liked the pictures, the menu and the tooltips ("Подсказка выглядит хорошо", "Выглядит супер"). He asked for:

1. A colour per kind of term. Owner, decision `term-colours-by-class`, 14.11: «Неплохо бы каждой подсказке какой-то выдать цвет тега. Ну, грубо говоря, там морю, синий и так далее, чтобы мы сразу понимали, к чему относится. Не только из подсказки, но и там к разному термину разный цвет.»
2. One simple word for a bait that stays on the hook. Owner, decision `reusable-bait-word`, about the tooltip text «Наживка уходит с ней, если только это не наживка, которая не теряется»: «Ну, давай просто сделаем так многоразовое.»
3. Sound while the game window is not in focus. Owner, decision `sound-in-background`: «когда игра не в фокусе, звук я не слышу... Я привык, что у меня обычно игра на втором окне всегда, и чаще всего на фоне играет.»

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\`.

- The glossary of 28 terms, the marks and the tooltip: `Catch/Source/Catch/CatchUiTerms.h/.cpp` (`Glossary[]`). Today a mark has one accent colour per text background (dark ink, warm light text, cool light text). Report: `Docs/briefs/reports/day2-view-term-tooltips.md`.
- The class colours of charms (Rod, Bait, Fish, Sea) are on the band at the top of a charm card: `Catch/Source/Catch/CatchUiPanels.cpp`, `CharmCard`, the `PCol` colours.
- Bait notes in the view: `Catch/Source/Catch/CatchHud.cpp:1196` ("Not used up. Lost when the fish escapes.") and the lines around it ("Not used up. Never lost.", Russian row `Docs/design/text-ru.tsv:227`). Core flags: `reusable` = not used up on landing, `survivesEscape` = never lost (`Catch/Source/CatchCore/Public/CatchCore/Tables.h:77-78`).
- The escape tooltip: `CatchUiTerms.cpp:59` ("...unless the bait is never lost."), Russian row `text-ru.tsv:434`.
- FULLSCREEN in the settings already means a borderless window: `Catch/Source/Catch/CatchUiMenu.cpp:112` (`EWindowMode::WindowedFullscreen`), OFF = `Windowed` (:117). No exclusive fullscreen anywhere.
- `Catch/Config` has no `UnfocusedVolumeMultiplier`, so Unreal mutes the game when its window loses focus (engine default 0).

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. Term classes. Every term of the glossary gets one class: ROD, BAIT, FISH, SEA or RUN. ROD, BAIT, FISH and SEA use the hue of the same charm class band, so one colour means one thing in the whole game. RUN (shelf slot, weightless, target, rarity, lucky chance and other terms of the shop and the run) gets a gold or neutral hue that no class band uses. Suggested sorting: ROD: rod level, pull, bare hook. BAIT: chum, reusable. FISH: chain, shoal, card level, hooked fish, escape, strike, size, points per kg, weight. SEA: the six zones, depth zone, day rule, Gold Sunfish, Sea charm. Sort by what the rule acts on; list the final sorting in the report.
2. The mark of a term (text colour and dotted underline) takes the class colour. Keep two tones per class: one for dark panels, one for the cream cards. Every mark must stay easy to read on its background; look at each class on both backgrounds in frames.
3. The tooltip and the term notes show the class: a small pill with the class word (ROD, BAIT, FISH, SEA, RUN) beside the title, and the tooltip border in the class colour. The class words are labels; add Russian rows.
4. The word "reusable". A bait with `reusable` is called "Reusable" (Russian «Многоразовая»). Rewrite the bait notes and the escape tooltip with this one word, true to the core flags: a reusable bait is not used up by a catch; with `survivesEscape` it also stays after an escape; without it, it is lost at an escape. No phrase like "a bait that is never lost" in a tooltip. Add the glossary term `reusable` (class BAIT) with a tooltip written from the core code, at most 140 characters. Keep each text short and plain.
5. Sound in the background. Add to `Catch/Config/DefaultEngine.ini`, section `[Audio]` (create the section if it is missing): `UnfocusedVolumeMultiplier=1.0`. This is the only line you may change under `Catch/Config`. Log the value once at startup (`CATCH_AUDIO unfocused volume <value>`, read from `FApp::GetUnfocusedVolumeMultiplier()`), so a packaged run can prove it.
6. No new window setting. FULLSCREEN ON is already a borderless window. Confirm it in the report in one line.
8. Rarity frames. Owner, decision `charm-rarity-frames`, 14.11: «На карточках непонятно, что обычное, что необычное. Имеется в виду про модификатор. ... Неплохо бы сделать градацию окантовки там какую-то. То есть у карточки сделать какую-то рамку разного цвета там. Без рамки, с рамкой там серебряной, медной и так далее.» The frame of a charm card shows its rarity as a material: Common: no frame; Uncommon: copper; Rare: silver; Legendary: gold with a slow glint. Painter only (gradient strips, a highlight, small corner pieces), no pictures. Use it wherever a charm is shown as a card: the shop card, the reward card, the hover tips; on the shelf chip a thin rim of the same material. Bait cards have a rarity too: give the pack bait cards the same frames. The frame replaces the coloured border that showed rarity until now. A weightless charm keeps its white rim and halo and must stay easy to tell from a silver frame: show one frame of a weightless Rare card. Do this block second, right after block 1.
7. Leftover of part 7, only if it is cheap: the term notes of a shelf tip may cover a shop tile; make them avoid the buyable tiles the same way the shop tip does.

## 4. Work by blocks

1. Classes and colours of the marks. 2. The class pill and border on tooltips and notes. 3. The word "reusable": bait notes, the escape tooltip, the new term, Russian rows. 4. The audio line and the log line. 5. Decision 7. 6. Proof frames and checks.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- Proofs with 0 errors and 0 missing keys: `run_text_proof.py terms en 1920`, `terms ru 1920`, `plain en 1920`, `screens ru 1280`.
- Frames, looked at by you: a shop with marks of at least three classes on cream cards and on dark tiles; one tooltip per class with its pill; a bait card of a reusable bait in English and in Russian.
- The log of one run shows `CATCH_AUDIO unfocused volume 1.0`.
- The report lists: the class of every term, the colours (hex) per class and background, the changed texts before and after.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms ru 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`: another executor builds the core worktree at the same time, so a build may wait. Do not kill a process you did not start; do not touch `Packaged/`. The owner plays the packaged game from `Packaged\v9` right now.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, the one line of decision 5 in `Catch/Config/DefaultEngine.ini`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, charm rule texts, `Catch/Source/Catch/Catch.Build.cs`, other config, the level, assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game. A tooltip explains a rule the core already has.
- No downloads, no generated pictures, no fonts, no new sound files. No deleting or hiding of old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The Russian word for a charm is «модификатор».

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-term-colours.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
