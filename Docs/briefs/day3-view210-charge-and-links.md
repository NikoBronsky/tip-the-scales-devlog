# Brief: view 2.10 - the dynamite charge, the link charms, playable build 16 with a Russian launch

Written by the orchestrator on 14.11.2026, 15:28. Executor: Opus, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base `085f565`. You do not commit.

## 1. Goal

The game director redesigned the candidate charm Dynamite today and asked for charms that trigger each other. His last word (14.11, 15:25): "ладно, приостанови дубли. сделай пока механики, дай играбельный билд и команду запуска на русском, потом уже будем запускать ролик." He wants to play the new mechanics tonight in a build that starts in Russian with one command. New rules freeze at 24:00.

The core side is being built right now by another executor in `F:\UE\EpicJam_core` (core 2.10, API 10; not finished, not accepted yet). You build the view side in two blocks: block 1 now, without the new core; block 2 when the orchestrator sends you the accepted core. The result is build 16: `F:\UE\EpicJam_ui\Packaged\v16\Windows`.

Everything new stays a candidate behind the launch argument `-new`. Started without arguments, the game must behave exactly as build 15.

## 2. What exists and what to read first (verify before you rely on it)

- How the view side of the first candidates was built: `F:\UE\EpicJam\Docs\briefs\day3-view-risky-fishing.md` and your predecessor's report `Docs/briefs/reports/day3-view-risky-fishing.md` in your worktree (how the core is merged into the view, how the `-new` switch, the blast effect and sound, the harpoon, the card pictures and shelf icons of Dynamite and Harpoon, the Russian text layer and the `.bat` files were made). Follow its ways.
- The core task and its three addenda (the rules you will show): `F:\UE\EpicJam\Docs\briefs\day3-core210-link-charms.md`, `day3-core210-addendum1.md`, `-addendum2.md`, `-addendum3.md`. Where they disagree, the later file wins.
- The session tables and the `-new` switch: `SessionTables()` in `Catch/Source/Catch/CatchSession.cpp` (about 156). Console commands: `CatchSession.cpp` (about 1126-1345) and the `catch.Hud*` family in `CatchHud.cpp`.
- Build 15 launch files (`F:\UE\EpicJam_ui\Packaged\v14\Windows\*.bat`): `Catch_RU.bat` = `start "" "%~dp0Catch.exe" -lang=ru`; `Catch_RU_new.bat` adds `-new`.

## 3. The rules you show (summary; the core files above are the source)

- **A charge of dynamite.** A source charm throws a charge from its rod at the surface. The charge flies in a straight line at a random downward angle, bounces off the left wall, the right wall, the surface and the seabed, and explodes after a random fuse of 0.8-3.0 s. Every free fish within the blast radius lands (35 % weight). A blast may take no fish. Each kind it took loses 1 chum level for good; the level may go below 0 (level -1: half the fish of that kind; the lowest level: the kind is gone from the sea). The shop sells chum for such a kind again.
- **Sources** (each a charm, each with a chance; a success is a "lucky chance"): Dynamite (a pull that lands a fish, 1 in 5), Powder Keg (each hooked fish landed, 1 in 25), Sore Loser (a fish escapes, 1 in 3), Chain Reaction (each chain link, 1 in 6), Lucky Fuse (any lucky chance that comes true). Horseshoe: +1 to every chance.
- **Chain Shot:** each chain link throws a harpoon from that hook. **Shockwave:** every explosion drives every fish one zone higher until the day ends. **Bait Bomb:** a pull of 3 or more fish throws 2 fish from the bucket back into the sea; every 5 thrown fish of a kind give that kind +1 chum level for good.
- The player never throws a charge by hand.

## 4. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. **Block 1 is built against a view-only preview**, so it does not wait for the core: debug console commands that drive the new visuals with fake data and never touch the core state: `catch.DebugCharge [rod]` (a fake charge with a random path, bounces and fuse, then the explosion with a few fake fish flying to the bucket), `catch.DebugChum <kind> <-1|1|gone>`, `catch.DebugBaitBomb`, `catch.DebugLift`. They are staging aids like `catch.Tips`; they stay in the build.
2. **The charge**: a small bundle of dynamite sticks with a burning fuse (a bright spark and a short trail of bubbles), drawn in the sea plane at the charge's (x, depth), turning as it flies. A bounce: a small puff and a knock. The explosion at the charge's point: reuse the blast flash, ring, bubbles and sound of build 15, moved from the rod to the point, a little stronger; a blast that takes no fish is the same explosion with no fish flying. Taken fish fly from the blast point into the bucket by the existing flight of the extra fish. It must read at a glance on a busy screen and at every depth.
3. **Chum changes outside the shop** are shown where they happen with a floating label and a short sound: `-1 chum: <Kind>` / `+1 chum: <Kind>` / `<Kind> left the sea` (Russian layer: `-1 прикормка: <вид>` / `+1 прикормка: <вид>` / `<вид> ушёл из моря`). These three are the only new free-standing player texts; use the game's existing label style.
4. **The shop** shows chum levels below zero (`lv -1 -> 0`) and never a price below the base price. A kind that left the sea is still offered.
5. **Bait Bomb**: the thrown fish fly from the bucket back into the sea (the bucket flight reversed), a splash, the bucket count and the points go down visibly; a level gain shows the label of decision 3 and the new fish swim in.
6. **Shockwave**: after an explosion a wave runs upward through the sea; the fish positions come from the core (they move up by themselves if the core moves them).
7. **Glossary**: the cards underline the term `charge of dynamite` and explain it with the text the core report gives (one text for all sources). Show the chance that holds now on the chance cards (with Horseshoe: `2 chances in 5`), by the way the core offers.
8. **Card pictures and shelf icons** for the new charms (Lucky Fuse, Powder Keg, Sore Loser, Chain Reaction, Horseshoe, Chain Shot, Shockwave, Bait Bomb): make them the way your predecessor made Dynamite and Harpoon, if that way needs no paid generator, no account and no browser of the director. Otherwise draw simple pictures by script in the same frame. Every temporary picture carries a small visible mark `TEMP` and is listed in the report. Nothing from outside the project; no downloads.
9. **Trying the candidates quickly.** With `-new` the charm slots of every shop visit show the candidates that are not on the shelf, in a fixed order, and a reroll shows the next ones. A new launch argument `-coins=<n>` sets the start coins (a testing aid; without it `-new` keeps its 20 coins). The rule days of `-new` stay as they are.
10. **Launch files of build 16** (next to `Catch.exe`): `Catch_RU.bat` (`-lang=ru`), `Catch_RU_new.bat` (`-lang=ru -new`), `Catch_RU_new_rich.bat` (`-lang=ru -new -coins=300`). Keep the other `.bat` files of build 15.
11. **The director is working at this computer.** A game window that opens takes his keyboard focus. Start every proof game without a window: add `-RenderOffScreen` to the launch line (with `-unattended -nosplash -log`), and check on your first run that `catch.Shot` frames are valid that way. If offscreen rendering does not give valid frames, stop and tell the orchestrator before you open any window. One game at a time; start and stop only your own processes.

## 5. Work

**Block 1 (now):** decisions 1-6 and 8 with the preview commands; build with `python -X utf8 Tools/check_all.py --ue`; proofs as frame sequences from the running game (`catch.Record` or timed `catch.Shot`), which you look at yourself: the charge in flight, a bounce, the explosion at two depths, an empty blast, the labels, the Bait Bomb throw, the shop with a negative level. Then report block 1 to the orchestrator and wait for the core.

**Block 2 (after the orchestrator's message with the accepted core commit):** merge the core, API 10; wire the real events and the state list of charges to the visuals; texts and glossary in English and Russian from the data file; decisions 7, 9, 10; build; package into `F:\UE\EpicJam_ui\Packaged\v16` (do not touch `v14` and `v15`); proofs on the packaged build with `-lang=ru -new -coins=300`: buy sources and Horseshoe, play a day, show a real charge, a real level loss, a real Bait Bomb, a real lift; and one run without arguments that shows no candidate.

## 6. Done means

- Block 1: the gate with `--ue` has no fail; the proof frames exist and you looked at them; the report lists what is real and what is preview.
- Block 2: the gate with `--ue` has no fail; build 16 starts with each of the three `.bat` lines; the proofs of block 2 are in the report; started without arguments the game shows no candidate; the default English text check passes; no Cyrillic under `Catch/`.
- `git status` shows only files you meant to change. Never add `Catch/SourceArt/Fish/atlas.png` or `atlas.json`.

## 7. Boundaries

- Write only inside `F:\UE\EpicJam_ui`. Do not touch `F:\UE\EpicJam_core` (another executor works there) or `F:\UE\EpicJam` (read only).
- Invent no rule and no number. A rule question goes into the report; build what needs no answer.
- Never click, close, move or answer a system dialog; change no system or security setting. Do not grab the desktop; use in-game frames.
- Do not stop any process you did not start. Delete nothing. No commits.

## 8. Report

`Docs/briefs/reports/day3-view210-charge-and-links.md` in your worktree, in English: per block what changed by file, the proofs with frame paths, every temporary picture, every decision you changed, open questions, what is not verified. Final message of each block to the orchestrator: the same in short. Time box: block 1 about 60 minutes, block 2 about 75 minutes. You report to the orchestrator, not to the director.
