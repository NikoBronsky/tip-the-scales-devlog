# Brief: view for core 2.11 - Spare Tackle and a shelf with copies, build 20 (14.11.2026 22:23)

Executor: the same Opus executor that built builds 17 - 19, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. You do not commit. You start only after the orchestrator has committed build 19.

## 1. Goal

The director decided tonight (decisions `charm-allows-duplicates`, `duplicate-copy-works-alone`, `spare-tackle-picture`):

> "Ещё должна быть карточка которая позволяет делает модификаторы не уникальными (т.е. могут появляться уже купленные модификаторы)"

> "Да, все ок. ... Просто сделай его редким. ... Понятное дело, что он половине модификаторов ничего не даст."

> "второй вариант - ящик со снастями"

The core has the charm (core 2.11.0). The game does not show it yet. Build 20 makes it playable behind the candidate switches `-new` and `-newmix`: the charm has its picture, name, rule text and short tag; the shelf, the sale and the pop-ups work when one kind of charm stands on the shelf twice. The default game (no switch) must not change. Code freeze is 15.11 12:00; a small, safe change beats a complete one.

## 2. What exists (the orchestrator states what it verified; verify the rest before you rely on it)

- Core 2.11.0 is commit `d342b5f` on `track/core`, API stays 10. Verified by the orchestrator: gate and core tests of the core branch, the diff of `Shop.cpp` and `Tables.h`. The orchestrator merges it into `track/ui` with `git merge --no-commit d342b5f` BEFORE it resumes you, so you start on a worktree where the core files equal `d342b5f` and the merge is staged, not committed. Check this first: `git -C F:\UE\EpicJam_ui diff d342b5f -- Catch/Source/CatchCore Catch/CoreTests` must print nothing. If it prints something, stop and report. (Done by the orchestrator at 22:32: the merge had no conflict, the two folders equal `d342b5f`; `Docs/design/modifiers-v2.data.json` differs from the core's file only in the Russian rule texts that the view branch reworded earlier, and it holds the `spare_tackle` row.)
- The charm: id `spare_tackle`, `charm::SpareTackle`, index 69 (so `charm::Count` grew by one), English name "Spare Tackle", Russian name "Запасная снасть", class sea, rarity rare, price 8, no needs, candidate. Rule, English: "The shop can offer charms you already own. Each copy works on its own." The Russian rule text is in `Docs/design/modifiers-v2.data.json` (`ruleRu`).
- The core executor's notes for the view are section 14 of `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core210-link-charms.md` ("Core 2.11: Spare Tackle"). Read it first. In short, not verified by the orchestrator in the view code: arrays sized by `charm::Count` grow by one; the shelf may hold one kind twice, so a shelf item must be found by its slot, not by its kind (`SellCharm(slot)`, `CharmSold.slot`, `CharmTextOf(kind, slot)`, `CharmFired`, `LuckyChance`); `FishLanded.charms` may name one kind twice; `ChargeThrown` carries no slot; a shop offer may be a kind that is already on the shelf; 43 of 69 charms have `CharmData::repeatable`.
- The picture the director chose: `F:\UE\EpicJam\Saved\art\spare_tackle\cut\spare_tackle_b.png` (488 x 559, an open teal tackle box with two hooks and two lures). The orchestrator copied it to `Catch/SourceArt/Items/spare_tackle.png` and added its row to `Catch/SourceArt/Items/items.json` (done at 22:32; 89 items, none temporary, no charm without a picture). You rebuild the atlas the same way as for build 18 (`make_item_atlas.py`, then the headless `build_item_art.py`). Do not run `make_temp_charm_art.py`.
- Build 19 (your own work): the launch switches `-new`, `-newmix`, `-newfirst=<...>`, `-coins=<n>`, and six launch files.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. Texts. Short tag for pop-ups: English `SPARE`, Russian `ЗАПАС` (Russian only in the testing layer, `Docs/design/text-ru.tsv` of this worktree, as for the other candidates). Name and rule text come from the charm data. No other new player-facing words except decision 3.
2. Shelf by slot. Every place of the view that maps a shelf chip to a charm must work with two chips of one kind: the hover text of a chip shows the text of that copy; selling the second chip sells the second copy and the first one stays in its place; a pop-up of a fired charm points at the chip that fired when the event names the slot, and at the first chip of that kind when the event names only the kind (say which events are which in the report); the lucky-chance display counts every copy as the core does. Where you find a place that silently assumes "one chip per kind" and cannot be made safe in the time, list it in the report instead of guessing.
3. Shop offer of a held charm. The offer card shows one small mark so that the player sees it is a second copy: English `OWNED`, Russian testing layer `ЕСТЬ`. Use an existing tag or badge style of the shop card; no new panel, no new screen. This is the only new visual element of this build.
4. Launch file for the director: `Catch_RU_new_spare.bat` = the Russian layer, the candidate switch, 300 coins, and Spare Tackle as an early offer of the first shop. Use the existing `-newfirst` mechanism if it fits; if it does not, add the smallest testing-only argument that does, and name it in the report. The file is a testing aid and never goes into the submission archive.
5. Defect C of build 19 (a small dark-blue patch that flickers on a hooked fish, only when several fish hang on the hooks). The orchestrator looked at your crop sheet: a hooked Lanternfish hangs upright on the hook, and in the last frames it is still drawn inside the head of the Gulper Eel that took that hook. The game rule is that the hunter eats the fish on the hook and is hooked itself, so an eaten fish must not stay on screen. Check one thing, at most 15 minutes, after everything else is proven: is the prey of a chain still drawn after the hunter has taken the hook (for how many frames)? If yes, remove it at the moment of the bite (a short fade is fine) and prove it with a frame sequence. If the prey is already removed at once, change nothing and say so. Do not hold build 20 for it. UPDATE 22:37: the director looked at the crop and answered "Нет, другое". The orchestrator sent the executor the director's own screenshot, a 6x zoom, the measured facts (a flat saturated dark indigo patch of about 20 x 35 px with horizontal banding on the head of the middle of three hooked Gulper Eels in the darkest zone) and two unproven readings (two fish cards at one hook with an unstable draw order; texture coordinates that leave the atlas cell). The allowance is 25 minutes; the result is either a reproduced and fixed patch or a list of the scenes that were run, plus one frame from the darkest zone with three hooked eels.
6a. Defect D, the open proof of build 19: one frame sequence of a blast that takes a fish longer than 100 units, from the blast to the bucket, so that the shrinking during the flight is seen on a big fish (the director's complaint was about big fish). Use a cheat run that puts big fish near the hook.
6b. Defect A of build 19: the press on a free depth handle now starts the drag before the hidden buttons. Say in the report whether a shelf chip that lies under a handle can still be hovered and sold, and how (for example after the hook is moved away).
6. Package into `F:\UE\EpicJam_ui\Packaged\v20\Windows`, with the six launch files of build 19 plus the new one.

## 4. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui` has no fail.
2. Proof on the packaged build 20, windowless (`-RenderOffScreen -unattended -nosplash -nosound`), frames in the build's `Catch\Saved\ue_mcp\`:
   - a shop frame with the Spare Tackle offer: picture, name, rule text, price 8, in English and in the Russian layer;
   - after Spare Tackle is bought: a shop frame with an offer of a charm that is already on the shelf, with the `OWNED` mark;
   - a shelf frame with two chips of one kind, and the hover text of the second chip;
   - the second copy sold, the first copy still on the shelf (frame plus the log line of `CharmSold` with its slot);
   - one day played with two copies of one chance charm on the shelf, with the log lines that show each copy rolling on its own;
   - the default game (no switch): the log shows that Spare Tackle is never offered in the first five shops, and the day plays as in build 19.
3. Build 20 starts and exits with code 0 in your own windowless smoke run. Size of the build without `Saved` and `.pdb` in the report.
4. `git status` shows only files you meant to change, plus the staged core merge. `Catch/SourceArt/Fish/atlas.png` and `atlas.json` stay untracked and untouched.

## 5. Verification commands

```
python -X utf8 F:\UE\EpicJam_ui\Tools\check_all.py --ue
powershell -File F:\UE\EpicJam_ui\Tools\package-win64.ps1 -Project F:\UE\EpicJam_ui\Catch\Catch.uproject -OutDir F:\UE\EpicJam_ui\Packaged\v20
```

Proof game pattern (no window, no sound, no focus taken):

```
Catch.exe -RenderOffScreen -unattended -nosplash -nosound -ResX=1920 -ResY=1080 -log -abslog=<log> -ExecCmds="catch.After 6 catch.Shot <name>,catch.After 9 quit" -lang=ru -new -coins=300
```

## 6. Boundaries

- Write only inside `F:\UE\EpicJam_ui`. Always absolute paths. Do not touch `F:\UE\EpicJam`, `F:\UE\EpicJam_core`, `F:\UE\EpicJam_submit`.
- Do not edit core files (`Catch/Source/CatchCore`, `Catch/CoreTests`). If the view needs something the core does not give, write it down as a question.
- Invent no rule and no number. The charm, its rarity, its price and what a copy does are decided. A rule question that this brief does not answer goes into the report; take the smaller reading.
- The default game must not change: every candidate, Spare Tackle included, stays behind the switches.
- All player-facing text is English; Russian exists only in the testing layer behind `-lang=ru`. Do not change the gate's `english-only` check. No Cyrillic under `Catch/` beyond what the gate allows.
- Before you compile, package, or start a game: check for a running `Catch` process (`Get-CimInstance Win32_Process -Filter "Name='Catch.exe'"`, read its command line). A game without `-RenderOffScreen` is the director's game: do not build, do not start anything, do not stop it; finish the code, report "code ready, not built" and hand back.
- One game instance at a time. Never stop a process you did not start. Never click or answer a system dialog.
- Delete nothing. No commits. No downloads. Nothing from the browser prototype goes into the game.

## 7. Report

Add a section "Spare Tackle and a shelf with copies (build 20)" to `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view210-charge-and-links.md`: what you found in the view that assumed one chip per kind, what changed by file, which events name a slot and which name only a kind, the new words, the launch file and its arguments, the proofs with frame names and log lines, what is not verified, open questions. Final message to the orchestrator: the same in short, blockers first. Time box: 60 minutes of work. You report to the orchestrator, not to the director; the orchestrator answers technical questions and carries director-level ones to the director.
