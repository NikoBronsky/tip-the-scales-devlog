# Brief: view part 9b - the game on core 2.5: the prize for the Gold Sunfish, the bare hook, six new charms

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. The base is the commit the orchestrator names in the launch message: view part 9a plus the merge of core 2.5 (`track/core`). The merge may leave the view code not compiling against the new core: making it compile is your first block.

## 1. Goal

Core 2.5 changes when the player gets a free charm and adds six rare charms. The view must show these rules so that a player understands them without reading a manual, and each new charm must be seen in the world, not only in a text.

Owner decisions (file `Docs/decisions/game-frame.answers.json` in the main checkout `F:\UE\EpicJam`; Russian words verbatim, dictated by voice):

- `free-charm-for-special-fish`: «По поводу бесплатных модификаторов их можно, но надо за какие-то условия. Я думаю, вот за рыбу это будет ок.» The free charm window no longer opens after every day; it opens only when the Gold Sunfish was landed that day.
- `sunfish-bare-hook-only`: «пускай рыба луна клюет только на пустой крючок.»
- `crazy-rod-charms-approved`: Trotline, Extra Edge, Bottom Rig. Owner's wish: «я просто хочу добавлять немножко безумия, чтобы игрок иногда мог делать какие-то ... супер длинные [удочки], которые там будут до дна доставать».
- `silly-charms-approved`: Upside Down, Tiny Terror, Whopper. Owner's wish: «чтобы выглядело немножко даже глупо ... достаточно редко, чтобы игрок мог чувствовать это как достижение.»
- `charm-rarity-frames`: frames by rarity (done in part 8; the gold frame now meets its first Legendary charms).

## 2. What exists now (verify before you rely on it)

- The core report with every interface change: `Docs/briefs/reports/day3-core25-fish-reward.md` (in the worktree after the merge; section "Interface changes (for the view)"). In short: API 6; `ShopData::rewardMode` {None, EveryDay, PrizeFish} replaces `dailyReward`; `FishData::bareHookOnly`, `FishData::prize`; event `PrizeEarned` (once per day, right after the `FishLanded` of the Sunfish); `State::prizeEarned`, `State::zonesMirrored`; `RodData::extraHooks`, `levelLimit`, `maxGoal`; `HookData::deepestZone`, `extra`; `FishEntity::sizeFactor` (Whopper 10); `FishDetails::bareHookOnly`, `prize`, `sizeFactor`, `baits` empty for the Sunfish, `eatenBy` lists every other kind while Tiny Terror is on the shelf; Whopper sends `CharmFired` with the new fish id when it enters; `UpgradeData::limit`.
- The six charms (id, name, class, rarity, rule):
  - `trotline`, Trotline, Rod, Legendary, "Every rod has 2 more hooks."
  - `extra_edge`, Extra Edge, Rod, Rare, "Rods can be sharpened 3 levels past the limit."
  - `bottom_rig`, Bottom Rig, Rod, Rare, "The lowest hook of each rod can go one zone deeper than your line."
  - `upside_down`, Upside Down, Sea, Legendary, "The zones swap places: deep fish swim near the surface, surface fish go deep."
  - `tiny_terror`, Tiny Terror, Fish, Legendary, "Any fish of another kind may eat the hooked fish, even a smaller one."
  - `whopper`, Whopper, Fish, Rare, "Once a day a fish ten times larger enters the sea. It weighs x10."
- Russian names and rules (for `Docs/design/text-ru.tsv`): «Перемёт» / «Каждая удочка получает +2 крючка.»; «Сверх предела» / «Удочку можно заточить на 3 уровня выше предела.»; «Донная снасть» / «Нижний крючок каждой удочки достаёт на одну зону глубже открытой.»; «Вверх дном» / «Зоны меняются местами: глубинная рыба плавает у поверхности, поверхностная уходит на глубину.»; «Мелкая, но злая» / «Любая рыба другого вида может съесть рыбу на крючке, даже более мелкая.»; «Рыба-гора» / «Раз в день в море заходит рыба в десять раз больше обычной. Она весит x10.»
- The view guesses that a hook accepts the Gold Sunfish on any bait: `CatchHud.cpp:4197`. The term `bare` says: "A hook with no bait. It catches only shoal fish of the Sunlight zone, and four times slower." (`CatchUiTerms.cpp`, `Glossary[]`); the term `sunfish` and the forecast plate describe the Sunfish and its prize of points and coins.
- The free charm window (the reward window) and its button exist since view part 3-4 (`CATCH_HUD reward window`, `FreeCharm` press); today it opens after every day.
- Item pictures: `Catch/SourceArt/Items/*.png` with `items.json`, one atlas texture, the table `CatchItemArt.inc`. The orchestrator delivers six new pictures (`trotline.png`, `extra_edge.png`, `bottom_rig.png`, `upside_down.png`, `tiny_terror.png`, `whopper.png`) and their `items.json` rows into that folder before you start; the report of view part 5 (`Docs/briefs/reports/day2-view-item-art.md`) says how the atlas is rebuilt.
- Part 8 report (`Docs/briefs/reports/day3-view-term-colours.md`): the cheat `catch.FrameRun <seed> <charm|bait>...`, proof mode `classes`, rarity frames. Part 9a report (`Docs/briefs/reports/day3-view-feel-highlights.md`): red and green outlines, the day-end count, the luck sign.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. The Sunfish and the bare hook. Every place that tells the player about the Gold Sunfish says the condition in plain words, from the core flag `bareHookOnly`, never from a guess: the forecast plate on the days before and on its day, the fish hover card, the fish table, the term `sunfish`, the term `bare` (it now also names the Sunfish and its normal bite speed). The green outline of part 9a follows the core: a baited hook never lights the Sunfish, a bare hook does.
2. The prize. The forecast plate names the prize when the core says `prize`: a free charm. When `PrizeEarned` fires: an answer in the world at once (the words, a burst, a sound, the plate marks the prize as earned). At the day end the free charm window opens as the prize, with a heading that says so. On a day without the prize no window and no button appear, and nothing looks broken or empty.
3. The six charms get their pictures, Russian rows and term marks like the other 53.
4. Each charm is seen in the world, from core data only:
   - Trotline: the two extra hooks of each rod hang on the line (they are the lowest hooks), with bait slots on the rod card; when the charm comes or goes the hooks appear or leave with motion.
   - Bottom Rig: the lowest hook may hang in the next closed zone (`deepestZone`, `maxGoal`); the depth drag allows it, the closed zone shows that this one hook may enter.
   - Extra Edge: the rod card and the Sharpen tile show the higher level limit (`levelLimit`), the levels past the old limit look special.
   - Upside Down: a clear sign that the sea is upside down today (`zonesMirrored`): the day plate says it, the zone labels show which fish live where today. The sea itself already follows the core.
   - Tiny Terror: nothing new in the world beyond the red outlines of part 9a, which must now light every other kind (check it).
   - Whopper: the fish with `sizeFactor` 10 is drawn ten times larger and stays readable (outline, hover, bite at the hook, the flight to the bucket, the scales); an announcement when it enters (`CharmFired`); its hover card says x10.
5. The gold frame of Legendary charms is checked on the shop card, the reward card, the hover tip and the shelf chip.
6. No mode choice in this part: the timed mode comes later as its own part.

## 4. Work by blocks

Order: compile on core 2.5 and run the old proofs; decisions 1 and 2; decision 3; decision 4 (Whopper and Trotline first); decision 5. Extend the report after each block.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes; proof runs in English and Russian with 0 errors and 0 missing keys.
- Frames from a running game, looked at by you: the forecast plate with the bare hook condition and the prize; a Sunfish day with one bare hook, the Sunfish landed, the prize answer, the prize window at the day end; a day end without the prize; each of the six charms on a shop card with its picture and frame; Trotline rods; a Bottom Rig hook in a closed zone; a rod past the old level limit; an Upside Down day; a Whopper in the sea, on the hook and on the scales.
- The report lists every changed text before and after and every decision you changed.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py classes en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms ru 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

The owner may play the packaged game on this machine. Run game proofs one at a time, as few as you need, grouped near the end of a block. Never leave a proof game running. If the orchestrator writes "pause", stop every build, test and game at once and go on with reading and editing only, until "resume". Do not kill a process you did not start; do not touch `Packaged/`.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, the atlas asset the item art tool rebuilds, `Docs/design/text-ru.tsv`, and your report file.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, charm rule texts, `Catch/Source/Catch/Catch.Build.cs`, config files, the level, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings. If the core is wrong or lacks data, write the exact case into the report.
- Do not invent rules or numbers of the game. The view shows what the core says.
- No downloads, no generated pictures (the six pictures come from the orchestrator), no fonts, no new sound files. No deleting or hiding of old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The Russian word for a charm is «модификатор».
- Nothing from the browser prototype goes into the game. Do not copy the look, names or sounds of any existing game.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-core25.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
