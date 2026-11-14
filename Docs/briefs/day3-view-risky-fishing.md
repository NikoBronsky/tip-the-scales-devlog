# Brief: view part 11 - risky fishing candidates shown in the game

Written by the orchestrator on 14.11.2026. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. Core 2.9 (API 9) is merged into this worktree before you start; the two new item pictures are already in `Catch/SourceArt/Items/`.

## 1. Goal

The game director asked on 14.11 (Russian, dictated by voice, he is away for several hours) for new kinds of fishing to look at: «варианты типа гарпунов ... или взрывчаткой ... динамит какой-нибудь, который достаёт много рыбы, но при этом уменьшает их количество. Или там боссов ... типа вредитель ... или браконьерская ловля, что за каждую выловленную рыбу у тебя уменьшается количество рыбы, и она там тощает, худеет ... Я бы такие варианты посмотрел.»

Core 2.9 holds four candidates, off by default: the charms Dynamite and Harpoon, the rule days Poachers and Pest. Make them playable and good to look at behind one launch argument, so the director can try them tonight. The default game (no argument) must stay exactly as it is now. Every action of the player must answer with motion, sound and an effect: a blast must feel like a blast.

## 2. What exists now (verify before you rely on it)

- Core 2.9 report with every interface addition: `Docs/briefs/reports/day3-core29-risky-fishing.md` (in this worktree after the merge). Read it first. In short:
  - `Tables::candidates`, `ShopData::testOfferOrder`, `PaceData::testRuleOrder`; `charm::Dynamite`, `charm::Harpoon`; `DayRule::Poachers`, `DayRule::Pest`; `Game::CharmInGame(kind)`.
  - `EventType::Blast`: rod, `x` = rod x, `count`, `otherFishIds` = the fish taken, `value` = the sea factor after. `EventType::HarpoonThrown`: rod, `x` = rod x, `seconds` = depth of hook 0, `count`, `otherFishIds` = the hit fish in hit order. Each is followed by one `CharmFired`, then by the `FishLanded` events of its fish. They come after `PullStarted` of that pull. The extra fish land at the START of the pull, from where they swim, not from a hook; the hooked fish land later, when the rod arrives.
  - `State::seaFactor` (1 each day, lower after blasts), `State::dayWeightFactor` (1 except on a Pest day), `RodData::pullExtras`.
  - Numbers in `PaceData`: `dynamiteRadius` 130, `harpoonReach` 260, `harpoonHits` 3, `harpoonFactors` {1, 1.5, 2}.
  - `charm::Count` is now 61. Every place of the view that counts or lists charms up to `charm::Count` (the title block "CHARMS n / 59", the run-over blocks, the task list, any table sized by the count, the Russian generator's count of modifiers) must skip a kind with `!Game->CharmInGame(kind)`, so the default game still says 59.
  - A balance pass of the core (2.9.1) runs while you work: the rule texts and numbers of the four candidates will change (for example blasted fish at half weight, a blast only on a pull that lands a hooked fish, a sparser sea on the Poachers day, a stronger Pest). The events and fields stay. Never copy a rule text or a number into the view: read them from the core. The orchestrator merges 2.9.1 into this worktree and tells you.
- The session builds its game in `Catch/Source/Catch/CatchSession.cpp:57-60` (`MakeDefaultTables()`); launch arguments are read there and in `LoadProfile` (`-tasks`, `-unlockall`, `-freshprofile`), the language in `CatchUiText.cpp:175` (`-lang=`). The agent cheat runs (`CatchSession.cpp`, the other `MakeDefaultTables()` calls) keep the default tables.
- Core events reach the view in `CatchHud.cpp` (about line 2700-2780), `CatchTackleView.cpp` (about 850-865), `CatchSeaView.cpp` (about 575), `CatchLookSound.cpp` (about 200-225), `CatchSession.cpp` (about 956).
- Big moments: `CatchMoments.h/.cpp` (steps by pull size, chain, fish share of the target). Signs, shakes and gold words exist; reuse them.
- Item pictures: `Catch/SourceArt/Items/items.json` + `<id>.png` -> `make_item_atlas.py` -> `build_item_art.py` (headless editor run, see its header) -> `CatchItemArt.inc`. The rows `dynamite` and `harpoon` and their PNG files are delivered by the orchestrator.
- Sounds are made by synthesis: `Catch/SourceArt/Sound/make_sounds.py` (numpy, fixed seed) and its README; find how the existing files reach `Content` and follow the same path.
- Russian test language: `Docs/design/text-ru.tsv` + `Catch/SourceArt/Text/gen_text_ru.py` (`--check`). Charm names and rules come from the data file.
- Proof runs: `Catch/SourceArt/Text/run_text_proof.py <mode> <en|ru> <1920|1280>`; agent console commands `catch.*`.
- Conventions of this track: your predecessor's reports `Docs/briefs/reports/day3-view-big-moments.md` and `day3-view-unlocks.md`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. **Launch argument `-new`** (a testing aid like `-tasks`; it works in a packaged game). With it the session builds its game from tables with `candidates = true`, `shop.testOfferOrder = {Dynamite, Harpoon}`, `pace.testRuleOrder = {Poachers, Pest}` and `pace.initialMoney = 20` (so the first shop can buy both charms). With `-newfirst=pest` the rule order is {Pest, Poachers}. One log line `CATCH_NEW ...` says what is on. Without `-new` nothing changes: no candidate charm, no candidate rule, the same start coins, no new log line.
2. **Extra fish fly from where they swam.** A `FishLanded` of a fish listed in a `Blast` or `HarpoonThrown` event of this frame starts its flight to the bucket at the fish's own last position in the sea (your own record of that fish), not at a hook. A fish you have no record of flies from the hook as today.
3. **Blast** (`Blast` event). At every hook of that rod that is in the water: a bright flash; a ring that grows to the real reach (`pace.dynamiteRadius` in sea units) in about 0.35 s, so the player sees how far the blast takes fish; a burst of bubbles; a short shake of the picture (at most 0.25 s, reuse the shake of the big moments); the water surface above answers if an existing effect allows it. A new synthesized sound `Blast` (a low thump with a noise burst, about 0.6 s, added to `make_sounds.py`). The fish then fly to the bucket as landed fish do (decision 2).
4. **Sea chip.** While `State::seaFactor < 1` a small chip near the day header shows `SEA -20%` (the lost share, whole percent) and pulses when it changes. It leaves when the day ends.
5. **Harpoon** (`HarpoonThrown` event). A steel line that shoots from (rod x, depth of hook 0) to the first fish and on to the next ones, about 0.07 s per segment; a spark at every hit; at hit 2 and 3 a small gold label with the factor (`x1.5`, `x2`, read from `pace.harpoonFactors`); the line fades within 0.3 s. A new synthesized sound `Harpoon` (a short twang; one tick per hit, pitch rising with the hit number, by the existing pitch-per-step rule). Then the fish fly to the bucket (decision 2).
6. **Rule day Poachers.** The rule text comes from the core and must show wherever a rule shows (day header, the forecast two days ahead, the announcement, the shop line). On that day a chip `FISH LEFT n` shows the number of free fish in the sea (states Swim and ToHook) and counts down live.
7. **Rule day Pest.** The same places for the text. Free fish are drawn thinner: body height x (0.5 + 0.5 x `dayWeightFactor`), eased, so the sea visibly starves as the player lands fish. A chip `FISH WEIGHT -12%` shows the lost share while the factor is below 1.
8. **Big moments.** A pull with extras has a large pull size. Check what the moment code shows for a blast of 5 or more fish (the extras land at the pull start, the hooked fish later) and make the pull sign count the whole pull once, not twice. Say in the report what the player sees.
9. **Russian rows** (testing aid). Use these, add rows for any other new text and list them. The two rule-day texts will change with core 2.9.1; the orchestrator then gives you the final pair, so keep them in one place.
   - `Poachers: no new fish swim in today.` -> `Браконьеры: сегодня новая рыба не приходит.`
   - `Pest: each fish you land makes the next ones 3% lighter, down to half weight.` -> `Вредитель: каждая пойманная рыба делает следующих на 3% легче, но не легче половины веса.`
   - `SEA -%d%%` -> `МОРЕ -%d%%`; `FISH LEFT %d` -> `РЫБЫ ОСТАЛОСЬ %d`; `FISH WEIGHT -%d%%` -> `ВЕС РЫБЫ -%d%%`.
10. **Proof mode `new`** in `run_text_proof.py`: launches with `-new`; a scripted run that buys Dynamite and Harpoon in the first shop, pulls on day 2 (frames: the blast at its start and middle, the harpoon line, the sea chip, the bucket), reaches day 3 (Poachers: header, chip) and, in a second launch with `-newfirst=pest`, a Pest day after several landings (thin fish, chip). Also a frame of the shop with the two cards and of the shelf. The last launch of the mode has no argument and shows that the default game logs no `CATCH_NEW` and offers no candidate.
11. **Sound recording for the video (last block, only if it is small).** A console command `catch.RecordSound start` / `catch.RecordSound stop <name>` that writes the game's own master output to `Saved/ue_mcp/<name>.wav` with the engine's audio mixer recording (look it up in the engine source under `F:\UE_5.9\Engine\Source`; do not guess the API). The orchestrator needs it to record the jam video without a screen recorder's microphone. If it needs more than about 40 lines or a risky module change, skip it and say so.

## 4. Work by blocks

1. `-new`, the atlas with the two pictures (`make_item_atlas.py`, `build_item_art.py`, build), the Russian rows; the game runs with the candidates: the cards in the shop with pictures, the rule texts in every place.
2. Decision 2, the blast, the harpoon, the two sounds, the sea chip, the big-moment check.
3. The two rule days: chips, thin fish.
4. The proof mode, frames in English and Russian, the old proofs.
5. The sound recording command (optional).

Extend the report after each block. Order of value if time runs short: blocks 1, 2, 4, 3, 5. Stop with a clean, building state rather than a half-working block.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`; `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes; the proofs below run with 0 errors and 0 missing keys.
- Frames from a running game, looked at by you, English and Russian: the shop with both cards; a blast (start and middle) with the ring; the harpoon line with the factor labels; the sea chip; a Poachers day; a Pest day with thin fish; the default title and first shop unchanged.
- Without `-new`: no `CATCH_NEW` line, no candidate offered, the frames of the `plain` proof look as before.
- The report lists: every file changed, every new text with its Russian row, the new sounds (how they are made), what the player sees for a blast of 5+ fish, every decision you changed, what you did not verify, questions.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
python -X utf8 Catch/SourceArt/Text/run_text_proof.py rush en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py new en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py new ru 1920
```

Run game proofs one at a time, grouped near the end of a block. Never leave a proof game or an editor process running. If the orchestrator writes "pause", stop every build, test and game at once until "resume". Do not kill a process you did not start; do not touch `Packaged/`. Use absolute paths in every command; never write into `F:\UE\EpicJam`, `F:\UE\EpicJam_core` or `F:\UE\EpicJam_submit`. PowerShell 5.1: no `2>&1` on native programs; write output to a file with `*>` and read the file.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**` (the two new synthesized sounds included), the `Content` assets that the existing scripts import for the atlas and the sounds, `Docs/design/text-ru.tsv`, and your report file. `Catch/Source/Catch/Catch.Build.cs` only for decision 11 and only by one module name; say so.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Docs/design/modifiers-v2.data.json`, rule texts or numbers of the core, config files, the level, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings. A core problem goes into the report.
- Do not invent rules or numbers of the game. No new screen, no settings entry. No downloads, no generated pictures, no fonts. Sounds only by the project's own synthesis script. No deleting or hiding of old work. No commits, no pushes.
- `Catch/SourceArt/Fish/atlas.png` and `atlas.json` are untracked on purpose; leave them alone.
- All player text is English; no Cyrillic under `Catch/` except the generated escapes. The theme rule of the jam entry: no director, cinema or "cut" wordplay in the game. Age 13+: a blast is water, bubbles and light, no gore.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view-risky-fishing.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 20 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. The game director is away. Write questions into the report; do not wait for an answer if other work remains.
