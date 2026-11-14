# Brief: day one, step 3 — the first playable version

Status: written and handed out on 12.11.2026 at 09:25 (owner's clock). Executor: Opus, effort high. This executor is the only one who writes in the Unreal Editor and in `Catch/Source/Catch/`.

## 1. Goal

Make the game playable for the first time: the Unreal layer shows the rules core and lets a person play the loop with the mouse. Preparation, day, catch flies to the bucket, shipping, summary with the fee, next day, loss, new run.

Why it matters to the owner: he approved the rules by playing a prototype and wants them in Unreal: «можно начать переносить». He also said «Главное — ощущение и полировка. Каждое действие игрока отвечает красиво: движением, звуком, эффектом». This step gives every core event a place to answer from. The answers are simple today; later passes make them beautiful without new plumbing.

## 2. What exists now (verified by the orchestrator on 12.11)

- The rules core is merged: `Catch/Source/CatchCore/`. Include `CatchCore/Game.h`. Read `Docs/design/core-interface.md` completely: sections 1-9 are the frozen interface, section 10 gives the exact C++ names, the fields of state and of every event. The core plays a whole run without a picture; its tests pass (`Tools/core-test.ps1`).
- The scene exists: level `/Game/Maps/Sea` (startup level), `ACatchSeaView` (`Catch/Source/Catch/CatchSeaView.h/.cpp`: temporary fish, water A/B, camera travel along depth, frame and bench commands), `CatchWorld.h` (the only place that turns game units into centimetres; screen-left is world +X, always convert through `XToWorld`). Report of that step: `Docs/briefs/reports/day1-scene.md`.
- Finding in the packaged game: for the first two seconds every fish stood in one column in the middle of the sea. Cause: the instance transforms hold X = 0 and only the material moves a fish to its place; the material was not ready yet. From now on the instance transform carries the real position.
- Rules and their signs for the player: `Docs/design/content-list.md` (Russian; game names and screen words in English): section 3 day and shipping, 4 rod cycle and the words over a rod, 5 chain and the signs of the choice, 9 how the player loads bait, 10 hooks, 15 response table (event -> motion, sound, effect).
- Technical frame: `Docs/plan.md` section 4. Editor link: `python -X utf8 Tools/ue_mcp.py`; ready calls: `Docs/research/ue59-editor-link-capabilities.md` (read blocks C, D and F before you start).
- The Unreal Editor is closed now. Start it yourself. Live Coding locks the editor target for every project of this engine: build with `Tools/build-editor.ps1 -Project F:\UE\EpicJam\Catch\Catch.uproject` only while no editor is open. An editor restart costs about one minute.
- In parallel another executor aligns numbers and small rules inside the core, in a separate folder (`F:\UE\EpicJam_core`). `Catch/Source/CatchCore` in your folder does not change while you work. Names of the interface stay. Never copy a number of the rules into your code: read it from the state, the events or `GetTables()`.
- The owner has not chosen the look yet (water A or B, sky, darkness of the deep, fish size). Do not rework the look. Keep water A. Keep one scale factor for the fish size in one place.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. One owner of the core in the Unreal layer: a world subsystem (working name `UCatchSession`). It holds `catchcore::Game`, sends commands, calls `Advance` once per frame, drains the events and hands each event to the presenters through one C++ delegate. Presenters read `GetState()` for everything that lasts and use events for everything that happens once. Frame order: core-interface section 2.
2. The Unreal layer decides nothing. No price, timer, chance or rule is computed outside the core. If the picture needs a value the core does not give, write it into the report; do not approximate it.
3. Every player action is a console command first, and the mouse calls the same function: `catch.NewRun [seed] [Soft|Normal|Hard]`, `catch.StartDay`, `catch.Pull <rod>`, `catch.Depth <rod> <depth>`, `catch.Bait <rod> <bait name|none>`, `catch.OpenPack <i>`, `catch.Pick <i>`, `catch.Skip`, `catch.Pay`, `catch.NextDay`, `catch.Buy <upgrade|card|charm|pack|worms> <i> [rod]`, `catch.Sell <slot>`, `catch.Reroll`, plus `catch.Dump` (one-screen text of the state into the log) and `catch.TimeScale <x>`. This is how you and later agents play the game without hands.
4. Fish: `ACatchSeaView` mirrors the fish of the core state. One instance per `FishId`; the transform is the real position every frame (`x`, `depth`, `direction`); the material keeps only the body wiggle. Kind, state and a phase go to per-instance data. Kinds differ by tint and by the size column of content list section 7 through one small view table; no new textures today. Rebuild the mirror when the core rebuilds its population (new day, new run). The temporary shader-driven schools stay available only for the bench commands.
5. Tackle: one presenter (working name `ACatchTackleView`) draws, for every rod of the state: a rod placeholder on the raft, the line, each hook, the bait on it, the fish on it, the hunter that comes, and the goal-depth handle. Placeholders are engine shapes. Rods and hooks stay visible at every depth. Each `FishLanded` event starts one flight of one fish along its own arc into a bucket placeholder on the raft; the bucket rocks on every arrival.
6. Interface: UMG, layout in a widget asset, behaviour in a C++ parent class (plan item 9). One HUD widget:
   - top strip of rod cards, always visible: state word (`CAST`, `SINK`, `WAIT`, `FISH ON`, `PULL`), time until escape as a ring or bar that turns red in the last two seconds with `SLIPPING!`, the pull preview (`$6` or `$21 CHAIN x2`), `HUNTER NEAR`, the bait name with portions left. A click on a card pulls that rod. A click on the bait counter picks the next bait the player holds (and none);
   - counters: money, day, time left, fee of today, catch count and catch value;
   - phase panels: preparation (`START DAY`; unopened packs: open, show the choices, pick), summary (catch, fee, `PAY`), run over (`NEW RUN`), shop (`NEXT DAY`, and the plain list of block 6).
   All text is English, taken from the content list or from `Tables` names.
7. Mouse: click a rod card to pull; drag the goal-depth handle of a rod up or down, also from the seabed; mouse wheel moves the camera along depth through the existing camera travel; click or Space skips the shipping.
8. Response table: one function per core event type in the presenters, named after the event, even where today's answer is only a small motion or a log line. This is the slot list for the polish passes.
9. Temporary sound: short tones made now by a Python script (numpy), kept under `Catch/SourceArt/TempSound/` with the script, imported under `/Game/Audio/Temp`. One sound per event of block 7. Nothing from outside.
10. No save files. No settings screen. No tutorial.

## 4. Work blocks (in this order: a playable loop first, comfort later)

1. Session subsystem, the console commands and `catch.Dump`. Check: play one whole day with commands only and read the money in the log.
2. Fish mirror from the core state (decision 4). Check: fish stand at their places in the first frame of a `-game` run.
3. Tackle presenter: rods, lines, hooks, bait, hooked fish, hunter approach, pull, flights to the bucket (decision 5).
4. HUD: counters, rod strip, mouse actions (decisions 6 and 7).
5. Day end: shipping (each `ShipItem` shows a coin number and the money counts up), summary, fee, run over, new run, next day.
6. Plain shop, only if blocks 1-5 work: lists from `state.shop` as buttons with name and price (upgrades, packs, charms, fish cards, Worm Tin), the shelf with sell buttons, reroll. No layout work; the real shop is the next step.
7. Temporary sounds on: cast, hook enters water, bite, chain link, escape, pull, fish lands in the bucket, coin of shipping, fee paid, day start, run over.
8. Play it yourself: two full days by console commands and at least one day by simulated mouse input. Take frames from the running game: preparation, a day with a fish on a hook, a pull with flights to the bucket, shipping, summary, shop, run over. Look at every frame.

## 5. Done means

1. A person can play with the mouse only, from the first screen: load bait, start the day, pull rods, see every fish fly to the bucket, watch the shipping, pay the fee, go to the next day, lose, start a new run.
2. Your own two-day run by console commands ends with the same money in `catch.Dump` as in the interface.
3. Fish stand at real positions from the first frame; no column in the middle.
4. `Tools/build-editor.ps1` builds with the editor closed; the game starts with `UnrealEditor.exe <uproject> -game`; `python -X utf8 Tools/check_all.py` is green.
5. The frames of block 8 are in a folder that git ignores and are listed in the report.

## 6. Boundaries

- You do not invent the game: no rule, no number of play, no new screen beyond this brief. The words on screen come from the content list. A missing decision goes into the report as a question; build what needs no decision.
- Do not touch `Catch/Source/CatchCore/`, `Catch/CoreTests/`, `Tools/`, `Docs/` (except your report) and the folder `F:\UE\EpicJam_core`.
- Every asset is made by you now, by script or from engine shapes. Nothing from outside the project and nothing from any earlier prototype. List each asset and how you made it.
- Do not commit. Do not delete files of other steps. Save everything before the editor closes. Leave the editor closed when you finish, with nothing unsaved.
- To save tokens, inspect work-in-progress frames at 1280x720 or smaller.

## 7. Report

English, in `Docs/briefs/reports/day1-playable.md`, at most 45 lines: what a player can do now; what is simple or missing; the commands; the frame list with real paths; values the picture needed and the core does not give; decisions you changed; every asset and how you made it; questions for the owner about feel.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Questions about feel: write them down; the orchestrator shows the owner frames and the build.
