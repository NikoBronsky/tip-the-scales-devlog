# Brief: day one, track 1 — the rules core `CatchCore`

Status: drafted before the jam start (12.11.2026, 04:16, owner's clock); refreshed and handed out after the start (12.11.2026, 08:23), when the project existed and built. Executor: Codex or Opus, effort high.

## 1. Goal

Build the rules of the fishing game as a headless C++ core with tests and a balance runner. The core plays a whole run without any picture: preparation, day, shipping, summary, shop, next day, loss. The Unreal layer (track 2, another executor) only draws the core state and plays the core events.

Why it matters to the owner: he played a browser prototype and said «Выглядит практически готово ... можно начать переносить». The core must reproduce those rules. His main note: «все очень хорошо, как будто бы слишком легко», so balance tables must be data and a runner must measure them.

## 2. What exists now

- The project exists since 12.11, 06:10: `Catch/Catch.uproject` with two modules, `Catch` (Unreal layer) and `CatchCore` (this brief). The editor target builds in about one minute, the game packages in about three minutes, and the packaged game starts and logs `Catch started. Rules core API version 1.`
- **Your work folder is a separate git worktree: `F:\UE\EpicJam_core` (branch `track/core`).** Write only there. Never write into `F:\UE\EpicJam`. Reason: track 2 keeps the Unreal Editor open on `F:\UE\EpicJam` with Live Coding, and Live Coding recompiles every changed source file of the project. Half-written core files would break its builds. The orchestrator merges your branch at 11:00.
- Skeleton of your module, in `Catch/Source/CatchCore/`: `CatchCore.Build.cs` (dependency `Core` only); `Private/CatchCoreModule.cpp` (the only file with an engine header); `Public/CatchCore/CoreVersion.h` and `Private/CoreVersion.cpp` with `catchcore::ApiVersion()`. Keep `ApiVersion()`: module `Catch` calls it. Copy the export macro pattern of `CoreVersion.h` (`#ifndef CATCHCORE_API`, `#define CATCHCORE_API`, `#endif`) for every public function and type that module `Catch` will use.
- Unreal puts its shared precompiled header in front of every file of the module. Engine macros are therefore visible in core files. That is the reason for the banned names in decision 2.
- The frozen interface note: `Docs/design/core-interface.md` (v1). Implement it. Its section 9 lists three readings of points the content list does not fix.
- Rules and numbers: `Docs/design/content-list.md`, third edition (Russian; game names in English). Sections: 3 day and shipping, 4 rod cycle and pace, 5 chain, 6 zones, 7 fish, 8 baits and packs, 10 hooks, 11 upgrades, 12 charms, 13 fee and special fish, 16 hook points, 19 unlock order. All numbers are temporary and live in data tables.
- Technical frame: `Docs/plan.md`, section 4, items 1-28. Owner decisions: `Docs/decisions/game-frame.answers.json`.
- Verified on 12.11: MSVC builds plain C++ outside Unreal in about 4 seconds; editor build takes 6-33 seconds (`Tools/build-editor.ps1 -Project <uproject>`).
- The browser prototype is not a source of code. Do not ask for it and do not copy from it. The rules are written here and in the content list.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. Plain C++20 without engine headers. One namespace `catchcore`. The same sources compile as the Unreal module `CatchCore` and as a standalone test program. Only `CatchCore.Build.cs` and one module file touch the engine.
2. Code rules so that both builds pass: no exceptions, no RTTI, no `dynamic_cast`, no iostream in the core, no global mutable state, no file-level names that repeat across files (unity build), no identifiers `check`, `verify`, `ensure`, `min`, `max`. Money is `int64`. Time is `float` seconds.
3. Determinism: one seeded generator owned by the game object. Same seed and same commands give the same event log. Fixed step: `Advance(dt)` splits `dt` into steps of at most 1/60 s.
4. One game object: `catchcore::Game`. Three surfaces: commands in (player actions), state out (const read access), events out (a queue the caller drains each frame). Nothing else leaves the core.
5. All balance is data: `catchcore::Tables` (fish, baits, packs, zones, upgrades, charms, fee tables by level, unlock conditions, day rules, pace numbers). `MakeDefaultTables()` fills it from the content list. No formula for the fee in code: a table by day plus a growth factor after the last row.
6. The core owns the fish as simple entities: kind, position (x along the surface 0-1000, depth 0-3400), home band, state. What the player sees is what can bite. A thousand entities at 60 Hz cost nothing in C++. The Unreal layer mirrors positions and adds the swimming motion in a shader. Fallback if mirroring is slow: the core keeps only fish that interact with hooks and emits their paths.
7. Charms do not get classes of their own. Owned charms compile into one `Mods` struct (multipliers, flags, added values) whenever the shelf changes. The 19 hook points of content list section 16 read `Mods`. The price of a fish runs through one ordered pipeline that records which charms fired.
8. Units: depth and width in abstract world units, as in the content list (zone borders 0, 300, 700, 1200, 1800, 2500, floor 3400). The Unreal layer scales them.
9. Player-facing names in the tables are English, exactly as in the content list.
10. Tests and the runner live outside the module folder, in `Catch/CoreTests/`. Unreal compiles every `.cpp` inside a module folder, so a file with `main` must not be there.
11. The standalone build uses flags close to Unreal: `/std:c++20 /W4 /WX /GR- /EHs-c- /D_HAS_EXCEPTIONS=0 /we4456 /we4457 /we4458 /we4459` (Unreal treats a shadowed name as an error). Use `std::int64_t`; do not define a name `int64`, Unreal has one.
12. Unreal-side compile check from your worktree: the game target. The editor target is locked while any Unreal Editor is open, for every project of this engine. Command: `& "F:\UE_5.9\Engine\Build\BatchFiles\Build.bat" Catch Win64 Development -Project="F:\UE\EpicJam_core\Catch\Catch.uproject" -WaitMutex`. Run it at least after block 3 and at the end.

## 4. Rules to implement

Take every number from the content list. Where this section and the content list disagree, the content list wins; report the difference.

### 4.1 Phases

`Prep` (time stands still; the player loads baits and sets depths) -> command `StartDay` -> `Day` (real time, day length from the tables) -> `Shipping` (the catch leaves the bucket one fish at a time, money arrives) -> `Summary` (fee is due; command `PayFee`; not enough money means `RunOver`) -> `Shop` -> command `NextDay` -> `Prep`. The run starts with one rod, 20 Worm portions and one free Small Pack. The game has no last day.

### 4.2 Rod

States: `Idle`, `Cast`, `Sink`, `Wait`, `Hooked`, `Pull`. A rod casts by itself during the day, sinks to its goal depth (the player may change the goal at any time, in both directions, also from the floor), waits, gets a bite, holds the fish, and is pulled by the command `PullRod`. After landing it casts again. A rod has a level (cycle speed, later escape, extra hooks at levels 3 and 5) and one or more hooks. Each hook carries one bait portion or is bare. A goal depth inside the sixth zone means the floor.

### 4.3 Bite

A free hook attracts the nearest fish in its zone that takes its bait. The fish swims to the hook and bites after a random 1-3 s divided by the bite rate. A bare hook attracts only shoal fish, four times slower. Bait rules, `any` baits, baits that are not used up and Bread's double rate for Sprat are in content list section 8.

### 4.4 Hooked fish, escape, chain

A hooked fish escapes after its hang time (plus one second per rod level above 1). The escape takes the bait. While a fish hangs, a hunter of a higher tier that swims near the hook depth may strike: it eats the hooked fish and is hooked itself. Each eaten fish adds 50% to the price of the final fish. The chain continues while a bigger hunter is near. Hunters of locked zones never strike. The escape is one replaceable function; the charm Barbed Hooks switches it off.

### 4.5 Catch, bucket, shipping

`PullRod` lands every fish on every hook of that rod. The price of each fish is computed at landing time through the price pipeline and stored with the fish in the catch list (the bucket). Emit one landing event per fish. Money does not change during the day. At day end the shipping phase pays the stored prices in catch order; equal fish in a row may ship as one batch when the catch is large; the whole shipping fits in eight seconds; a command skips it.

### 4.6 Shop, packs, upgrades, charms

Content list sections 8, 11, 12 and 19. Shelf of five charm slots, weightless charms, sale for half price, reroll price, fish cards, Deeper Line, Bigger Shelf, bait packs with a pick, Worm Tin. The shop and the packs offer only what is unlocked (section 19). The core reports what became new at each shop visit.

Day one of the jam needs the 19 hook points and the eight step-1 charms of section 19. Put all twenty charms in the table; implement the rest as the hook points allow.

### 4.7 Fee, special fish, levels

Fee by table and level (`Soft`, `Normal`, `Hard`; default `Normal`), content list section 13. Every third day the Gold Sunfish comes with one day rule from the table; the shop shows the rule of the next day on the evening before.

### 4.8 Queries for the Unreal layer

- `FishInfo(kind)`: name, zone, price now, baits that catch it with the portions the player holds, flags (shoal, hunter, floor), known or unknown (locked zone).
- `PullPreview(rod)`: money the player gets if he pulls now, chain length, seconds until escape.
- `FeeOf(day)`, shop contents with prices and lock states, shelf, stock of baits, catch list.

## 5. Events (minimum list; add fields the Unreal layer will need)

`PhaseChanged`, `RodStateChanged`, `HookEnteredWater`, `FishAttracted` (rod, hook, fish, time to bite), `FishHooked` (escape time), `EscapeSoon`, `FishEscaped` (bait lost), `HunterStrikes` (rod, hook, hunter, time to hit), `ChainLink` (eater, eaten, chain length, price now), `PullStarted` (fish count), `FishLanded` (one per fish: catch index, kind, price, charms fired), `BaitUsed`, `HookWentBare`, `DayEnded`, `ShipItem` (kind, count, price, charms fired), `MoneyChanged`, `FeePaid`, `RunOver`, `DayRuleAnnounced`, `ShopOpened`, `Purchase`, `CharmSold`, `CharmFired`, `ZoneUnlocked`, `ContentUnlocked` (what is new), `PackOpened`, `BaitPicked`.

Events carry ids and numbers only. No text, no pointers into core memory.

## 6. Work blocks

1. Module skeleton, `Tables` with defaults, generator, event queue, phases.
2. Fish entities: spawn by zone counts, swim inside the home band, schools as a shared drift; enter from the edges.
3. Rod state machine, bite, escape, chain, pull, price pipeline, catch list.
4. Shipping, fee, loss, special day rules.
5. Shop: upgrades, packs, charms, shelf, fish cards, unlock order.
6. Standalone build script `Tools/core-test.ps1`: finds MSVC (vswhere), compiles the core (`Catch/Source/CatchCore/`, without `CatchCoreModule.cpp`) and the tests (`Catch/CoreTests/`) into `Catch/Saved/CoreTest/`, runs them, returns a non-zero code on failure. It works from any checkout: paths are relative to the script. Add this script to `Tools/check_all.py` as a step.
7. Tests (plain asserts with your own macro name): rod cycle timings; bare hook; bait match for every bait; chain of three links; escape and bait loss; price pipeline order with each implemented charm; one landing event per fish; shipping sum equals the sum of landed prices; fee tables of all three levels; unlock order; loss and a new run; same seed gives the same event log; 10,000 random commands keep the invariants (money never negative, no NaN, counts never negative).
8. Balance runner `core_runs`: plays N runs with a simple buying policy without any picture and prints, per fee level, the share of runs alive after each day and the median money against the fee. Two policies: "saves everything" and "buys the cheapest useful thing". This is a measuring tool; it does not tune numbers.
9. A short interface note for track 2: `Docs/design/core-interface.md` (English): commands, state, events, units. The orchestrator freezes it; later changes go through the orchestrator at 11:00, 14:00 and 19:00.

## 7. Done means

1. `Tools/core-test.ps1` passes in under 20 seconds.
2. The game target builds from your worktree with the core inside (decision 12).
3. `python -X utf8 Tools/check_all.py` is green in your worktree. Do not pass `--ue`: it builds the editor target, which is locked.
4. `core_runs` prints its table for the three fee levels.
5. `Docs/design/core-interface.md` exists and matches the code.

## 8. Boundaries

- You do not invent the game. A rule, number, screen or fish behaviour that is not in the content list: write the question into the report, add a data field with the nearest content-list value, continue.
- Do not touch `Catch/Source/Catch/`, `Catch/Content/`, `Catch/Config/`. Do not start or use the Unreal Editor. Another executor owns them.
- Files you may create or change, all inside `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Tools/core-test.ps1`, `Tools/check_all.py` (one new step), `Docs/design/core-interface.md` (keep it equal to the code), `Docs/briefs/reports/day1-core.md`. Nothing else.
- Do not commit. Do not delete files. Player-facing text is English only.
- Do not run `Tools/build-editor.ps1`. The orchestrator builds the editor target after the merge.

## 9. Report

English, in `Docs/briefs/reports/day1-core.md`, at most 40 lines: what works, test and runner output (last lines), questions about rules, decisions you changed, what track 2 must know.

## 10. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Game questions: write them down; the orchestrator asks the owner.
