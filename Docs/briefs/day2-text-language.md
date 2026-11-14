# Brief: one text layer and Russian for testing (chosen at launch, no menu)

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 02:45 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message (the interface fixes of `day2-hud-fixes.md` are already in it). This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-text-language.md`). It replaces `day2-menu-language.md`, which was never started.

## 1. Goal

Put every player-facing text of the view through one text layer, add a Russian table, and let the game start in Russian by a launch argument or a console command. English is the default and the only language a normal start shows. There is no menu and no language switch on screen.

Why it matters. The owner (game director) tests the game himself and reads Russian faster. His decisions of 13.11 (voice input, verbatim):

- `main-menu-language-switch`: «В игре я бы хотел тестировать на русском, просто потому что я умею переводить, но это так долго, и она нам же важно делать быстрый тест.»
- `language-by-launch-command`: «Окей, давай сделаем сборку, будем сдавать без переключателя. Давай так, давай тогда не будем добавлять в меню языки. Просто ты какую-нибудь консольную команду пропишешь, и будем запускать экзешник с консольной командой на русском языке.»

The jam rule says all game text must be in English. So Russian is a testing aid: off by default, and removable from the build by one flag.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\Catch\`.

- The HUD is a painter HUD: text is drawn every frame, so a language change can show at once. Font: `Ui::Font(LocalPx, Weight)` (`Source/Catch/CatchUi.cpp:171-176`) returns the engine default Slate font (Roboto). The zone names in the sea are 3D text with the engine font `RobotoDistanceField` (`Source/Catch/CatchSeaView.h:72`); that font may have no Cyrillic glyphs.
- There is no single text function. A read-only agent counted about 336 player-facing literals on `f7e7a97`: `CatchHud.cpp` 148, `CatchUiPanels.cpp` 112, `CatchUiText.cpp` 40, `CatchUiHud.cpp` 18, `CatchUiSide.cpp` 9, `CatchUi.cpp` 9. Many are format strings (`"+%d pts"`). The interface fixes added a few more ("DAY RULE", "NO BITES", "PULL", "Rod level N"). Charm descriptions are a view-side map keyed by the core charm name (`CatchUiText.cpp:7-33`), charm tags another map (`CatchUiPanels.cpp:219-227`). Fish, bait, upgrade, pack and rule-day names and texts come from the core tables through `UCatchSession` (for example `FishName`, `CatchSession.cpp:250-253`).
- Console commands are registered with `FCatchCommand` (`CatchSession.cpp:447` is `catch.NewRun`). Scripted proof runs start the game with `-ExecCmds="catch.NewRun 42,..."`.
- The check gate (`Tools/check_all.py`, check `english-only`) fails on any Cyrillic letter in the game folder `Catch/` in files `.h .cpp .cs .ini .csv .json .txt .md .po`. It does not scan `Docs/`. Do not change the gate.
- Russian terms already used in design documents: `Docs/design/modifiers-v2.data.json` has `nameRu` and `ruleRu` for the designed modifiers; `Docs/design/content-list.md` describes fish, bait and upgrades in Russian prose.
- Coming soon from the core (not in this branch yet): modifier names and rule texts will come from a core query as English templates with `{n}` for a live value. Your text layer must be able to translate such a template before the number goes in.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. One function for all view text, for example `Ui::L(const TCHAR* English)`: it returns the Russian string when the language is Russian and the table has the key, otherwise the English string itself. The English string is the key. Format strings are keys too (`L(TEXT("+%d pts"))`), and the translation keeps the same specifiers in the same order.
2. Every player-facing literal of the view and every name or text that comes from the core goes through that function at the point of drawing. Console and log text stays English and does not go through it.
3. The Russian source table lives outside the game folder: `Docs/design/text-ru.tsv` in your worktree (UTF-8; columns: English key, Russian text, optional note). A generator script `Catch/SourceArt/Text/gen_text_ru.py` (Python 3, standard library only) writes `Catch/Source/Catch/CatchTextRu.inc` with every non-ASCII character as a `\uXXXX` escape, so the game folder holds no Cyrillic letter and the gate stays green. `--check` exits 1 when the file on disk differs from a fresh generation, when a pair has different format specifiers or `{n}` placeholders, or when a key is duplicated.
4. One build flag, `CATCH_WITH_RUSSIAN` (set to 1 in `Catch.Build.cs`), compiles the table and the language commands in. With 0 the game is English only and ignores the launch argument. Both values must compile.
5. How Russian is chosen. A launch argument `-lang=ru` on the command line of the game, and the console command `catch.Lang en|ru` (logs one line), so `-ExecCmds="catch.Lang ru"` works too. The choice is not stored: every start without the argument is English. Nothing on screen offers the choice.
6. Missing translation: show English and remember the key. `catch.TextMissing` logs every key that was asked for in Russian and not found, once each.
7. Russian wording: short, natural, one term per thing. Use this glossary; for the modifiers that exist in `modifiers-v2.data.json` use its `nameRu`.

   | English | Russian |
   | --- | --- |
   | rod / hook / line | удочка / крючок / леска |
   | bait / Worm / bare hook | наживка / Червь / пустой крючок |
   | bait pack / pick | набор наживки / выбрать |
   | chum | прикормка |
   | modifier (charm) / shelf | модификатор / полка |
   | points, pts / coins, c / kg / s | очки, очк. / монеты, мон. / кг / с |
   | norm (target) of the day | норма дня |
   | day / preparation / shop | день / подготовка / магазин |
   | pull / escape / chain | выдернуть / срыв / цепочка |
   | day rule | правило дня |
   | Sharpen Rod / New Rod / Deeper Line / Bigger Shelf | Заточка удочки / Новая удочка / Длинная леска / Большая полка |
   | Sunlight / Twilight / Midnight / Abyss / Trench / Seabed | Солнце / Сумерки / Полночь / Бездна / Жёлоб / Дно |
   | Sprat / Mackerel / Sea Bass / Tuna | Килька / Скумбрия / Окунь / Тунец |
   | Lanternfish / Hatchetfish / Anglerfish / Gulper Eel | Рыба-фонарь / Топорик / Удильщик / Большерот |
   | Oarfish / Giant Squid / Coelacanth / Gold Sunfish | Сельдяной король / Гигантский кальмар / Латимерия / Золотая рыба-луна |
   | run over / new run | забег окончен / новый забег |

   Translate the other fish, baits and texts in the same manner and list every term you chose in the report.
8. Russian text is longer. No text may be clipped, overlap its neighbour or leave its panel in either language at 1920x1080 and 1280x720. Fit by shorter wording first, then by the existing text-fit helpers; do not shrink a font under its readable size. The English layout must not get worse.
9. 3D text in the sea (zone names, padlock labels): if its font has Cyrillic glyphs, translate it; if not, leave those labels English in both languages and say so in the report. Do not import a font.
10. The owner's main rule covers this work too (`responsiveness-is-the-main-rule`: everything on screen answers the player and feels pleasant). Wrapping a text must not remove or weaken any pop, count-up, tilt or timing that text has now. A language change at run time shows on the next frame without a restart.

## 4. Work by blocks

- A. The text layer, the generator, the flag, the launch argument, the two console commands.
- B. Wrap the literals and the core-provided names and texts, file by file. Write the Russian table.
- C. Layout pass in Russian on every screen, both resolutions.
- D. Proof runs, frames, report.

If you run short of session, stop after a finished block with a green build and report exactly what is left.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui`: `10 ok, 0 fail, 1 skip`. The `english-only` check passes untouched.
2. `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` exits 0.
3. The game builds with `CATCH_WITH_RUSSIAN` 1 and 0 (show both build results; leave it at 1).
4. The plain scripted day and the flow run give the same numbers as before your work (the launch message names them). A start without the argument shows English everywhere.
5. A scripted run started with `-lang=ru` through preparation, pack, day, shipping, summary, shop and a rule day ends with `catch.TextMissing` reporting 0 missing keys for what was shown.
6. Frames, looked at by you, in both languages: preparation; a day with a fish on the hook; the full-screen rule card; the countdown; shipping; the summary; the shop; an open bait pack; a fish hover card; run over. And the shop and the summary in Russian at 1280x720.
7. No change in `Catch/Source/CatchCore/**`.

## 6. Verification commands

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_check.sh" /f/UE/EpicJam_ui tl "catch.HudAutoPack 0,"
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_feedback.sh" /f/UE/EpicJam_ui tlf
```

Read the two scripts before you copy them for your own runs; do not edit them in place. Frames land in `Catch/Saved/ue_mcp/`. The editor must be closed while UnrealBuildTool builds. You are the only one who builds the Unreal project.

## 7. Boundaries

- You may change or create: `Catch/Source/Catch/**`, `Catch/SourceArt/Text/**`, `Docs/design/text-ru.tsv`, and your report. Nothing else.
- Do not touch `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Content/**`, `Catch/SourceArt/Sound/**`, `Tools/**`, `CLAUDE.md`, `AGENTS.md`, other files under `Docs/`.
- No Cyrillic letter in any file under `Catch/`. No change to the check gate.
- No menu, no options screen, no language control on screen, no stored language setting. No rule, number or screen beyond this brief.
- Nothing from the browser prototype goes into the game. No downloads, no fonts, no new sounds, no generators.
- Keep what works: card tilt and hover, pack tear, bite and pull feedback, the countdown, the end-of-day count, the sound bed. Do not delete or hide existing elements.
- No commits, no pushes, no packaging. Close the editor if you opened it. Never kill an Unreal process you did not start.

## 8. Report

Write `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-text-language.md` in English and hand back a short summary: what is done per block and where (file, function); the count of keys and of translated keys; the terms you chose beyond the glossary; texts left English and why; the check results with log lines; the list of frames with one line each; every decision you changed; questions.

## 9. Chain of command

You report to the orchestrator, not to the owner. Answer technical questions yourself and list them as revocable readings. A question about what the player should see beyond this brief goes into the report, and you continue with the parts that do not depend on it.
