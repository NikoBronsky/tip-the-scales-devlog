# Brief: the fish and the sea answer the player

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 02:55 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_look`, new branch `track/look`, base `7aa3ee4` (clean, never built). This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-world-feedback.md`).

## 1. Goal

Make the fish and the sea react to what the player does: fish near a hook, a bite, a strike or a landing flinch; the hooked fish fights; the hunter gulps; the raft dips under the catch; sea life, plants and drifting specks answer too. View only: no fish changes its position in the rules, no rule changes.

Why it matters. The owner (game director) restated his main rule on 13.11 and then widened it (voice input, Russian, verbatim):

- `responsiveness-is-the-main-rule`: «Нам главное сделать отзывчивость, чтобы каждое движение давало прикольную [...] полишинг [...] У нас сейчас с этим очень хорошо, но иногда это как бы многие моменты не реализованы.»
- `responsiveness-covers-everything`: «я имел в виду не только интерфейс [...] Я имел в виду все сразу. То есть вот рыба и так далее, это же не интерфейс, но нужно сделать, чтобы это тоже, чтобы рыбка реагировала. [...] Все должно быть приятным.»

A read-only audit found the main world gap: fish react to nothing the player does. Their motion comes only from the core; the view adds the hook swing, the nibble shake, the tail beat and the size ease.

## 2. What exists now

Read the audit first: `F:\UE\EpicJam\Docs\briefs\reports\day2-feedback-audit.md`, section 4 (Table D, rows D3-D29 with file and function for each moment), section 5.3 (world helpers you can reuse), and gaps G3, G4, G8, G11-G13, G15, G18, G27, G41, G47 in section 6. It was written against this same base commit, so its function names are right here; its line numbers are approximate. Verify each fact before you rely on it.

The short version (paths under `F:\UE\EpicJam_look\Catch\Source\Catch\`):

- `CatchLookFish.cpp`, `FCatchLookFish::Update`: draws every fish from the core state. Per instance it sets colour, size, phase, tail rate (`D_Rate`, with a phase-safe rate change already coded), mode (`D_Mode`: 0 swim, 1 hover silhouette, 2 strike eye, 3 flight). The Main block draws the hooked fish (swing +-4 units at 14 rad/s, rock +-0.22 rad, tail x 2). It does not read the escape time, the rod state or the hunter. `FFrame` already carries `State` and `Tables`.
- `CatchTackleView.cpp`: event handlers `OnFishHooked`, `OnFishLanded`, `OnChainLink`, `OnFishEscaped`, `OnPullStarted`, `OnRodStateChanged`; `OnHunterStrikes`, `OnEscapeSoon`, `OnBaitUsed`, `OnHookWentBare` are empty. Helpers: `SpawnBubbles`, `SpawnSplash`, `SpawnDots(Color)`, `SpawnSurfaceRing`, `BucketHit`, `RaftBob`; per rod `Kick`, `Jerk`, `Squash`, `Flash`. `StepBucket` steps the bucket and the scales; `FCatchLookScales::TakeLanded()` is never read, so the day weight lands on the scales in silence.
- `CatchSeaView.cpp`: `ACatchSeaView::UpdateLook` feeds the fish look; `AddShake(px)` has one caller (`OnChainLink`, 6 px).
- `CatchLookSea.cpp`: `FCatchLookSea::StepLife` (turtle, jellyfish, big shadow), `DrawPlant`, `Tick` (marine snow, plankton lights). All move on time only. `FView` is their one input.
- The fish material takes every value below per instance already. No material or asset change is needed.
- Sound: `FCatchLookSound::Play(Name, Pitch, Volume)`; the files and play rules are in `Catch/SourceArt/Sound/README.md`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

All numbers below are look numbers proposed by the audit. Keep them in one block of named constants per file so the orchestrator can tune them. You may adjust a number when a recorded clip shows it reads badly; say so in the report.

1. Startle list (G3). A view-only list of short-lived startle points (x, depth, radius, age, strength), owned where both the tackle view and the sea view can reach it, with one public call to add a point. Every drawn fish inside a point's radius beats its tail x 2.2 for 0.5 s, fading out, and rocks its pitch +-0.08 rad once. Its position does not change, so picking and hooks stay true. Keep the list small (drop points when they expire; cap it) so the cost per fish stays flat with 1000 fish.
2. Who adds points: a sinking hook, one point every 0.25 s, radius 60 units (D3); a bite, at the hook, radius 150 (D7: the school mates flinch); a landing, at the hook line just under the surface, radius 150 (D14); a strike, at the hunter, radius 120 (D11). Add the calls inside the bodies of the event handlers and, for the sinking hook, in the smallest possible place.
3. The hooked fish fights near the escape (G4, D9): while the hook's escape time is inside `pace.escapeWarning` and escapes are not switched off, the swing grows from 4 to 7 units and from 14 to 22 rad/s, the rock from 0.22 to 0.35 rad, the tail beat x 1.5.
4. The fish thrashes on the pull (G11, D13): swing and tail x 1.5 while its rod is in Pull; a surface ring and 4 splash drops when the pull starts.
5. The prey panics at a strike (G12, D11): while its hook has a hunter, the hooked fish swings as in decision 3; the rod jerks 0.6; shake 3 px.
6. The hunter gulps (G13, D12): when it eats, its drawn size jumps x 1.25 and eases back through the existing size ease; a surface ring. One small `Punch(FishId, Factor)` call on the fish look.
7. The hovered fish notices (G8, D22): tail beat x 1.5 while the cursor is on it (its slot and the hover copy).
8. Camera shake for big moments (G18, D26): 2 px when a fish of tier 3 or more lands; 3 px on a strike (shared with decision 5). No automatic camera moves.
9. The raft dips (G15, D15-D17): one damped dip term on `RaftBob`: 2 units per fish that drops into the bucket, 3 when the catch reaches the target, 2 when the day weight lands.
10. The day weight lands (G27, D17): on `TakeLanded()`: sound `Stamp` at pitch 0.6, 6 grey dots at the pan, the raft dip of decision 9.
11. The Gold Sunfish arrives (G47, D29): on its first drawn frame its bloom pulses from 0.9 to 1.6 alpha and back over 1.5 s. Add the soft `Chord` (volume 0.4) only if the sound class is reachable without new plumbing; otherwise leave the sound out and say so.
12. Sea life, plants and specks (G41, D23-D25), last block: a turtle within 150 units of a splash or a landing turns away and swims x 2 for 1 s; a jellyfish inside a startle point pulses its bell x 2 for 1 s; a plant within 80 units of a startle point bends away by up to 12 units and springs back over 1 s; specks within 40 units of a pulled line or a splash drift sideways and come back. All read the startle list.
13. Debug and proof. A console command `catch.LookStartle x depth [radius]` adds a point by hand. A command `catch.LookDump` (or an extension of an existing dump) logs, for the fish inside any live point: id, kind, tail rate factor; and the current values of the hooked-fish swing for each hook with a fish. Log one line when a point is added (throttle the sinking-hook points to one line per second).

Not in this brief (they wait for other work that edits the same code): everything painted by the HUD (dusk overlay, popups, the depth handle, a click on empty sea), the float and the hook cards inside `DrawRods` (float pop, float settle, bait pop, bare hook flash), real fleeing of fish (a rule: it would change which fish can bite).

## 4. File rules (another executor edits `F:\UE\EpicJam_ui` at the same time; your branch is merged into theirs later)

- Free to change: `CatchLookFish.h/.cpp`, `CatchLookSea.h/.cpp`, `CatchSeaView.h/.cpp`, `CatchLookScales.h/.cpp`.
- `CatchTackleView.h/.cpp`: additions only, kept small and local: new members, new small functions, calls inside the bodies of the event handlers named in section 2, and the changes in `StepBucket` / `BucketHit` for decisions 9 and 10. Do not edit `DrawRods` beyond the one call for the sinking-hook points, and do not reformat or move existing code: every moved line becomes a merge conflict.
- Do not touch: `CatchHud.*`, `CatchUi*.*`, `CatchSession.*` (console commands: register yours in a file you are free to change, in the same way the existing `catch.*` commands are registered), `CatchLookSound.*`, `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Content/**`, `Catch/SourceArt/**`, `Tools/**`, `CLAUDE.md`, `AGENTS.md`, `Docs/**` except your report.

## 5. Work order

Decision 1 and 2 (the mechanism and its four uses), then 3, 5, 6, 4, 7 (the fish), then 9, 10, 8 (raft, weight, shake), then 11, then 12, then 13 if not already done along the way. If you run short of session, stop after a finished decision with a green build and report exactly what is left.

## 6. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_look`: `10 ok, 0 fail, 1 skip`.
2. The plain scripted day is unchanged: 102 events, DaySettled `31 pts, target 10 pts`, coins 8, 0 errors. The view must not change any core result.
3. Log proof from a scripted run: points added at a sinking hook, a bite and a landing; `catch.LookDump` shows fish with a raised tail rate inside a live point and none outside; the hooked-fish swing values rise inside the escape warning; a `Punch` line at a chain link (use a day where eating is on: day 2 or later, `catch.NewRun` then play to day 2, or the existing debug commands; say how you got there).
4. A short clip or a frame sequence for each of: fish flinching near a sinking hook; school mates flinching at a bite; the hooked fish fighting before the escape; a strike with the prey in panic and the gulp; a landing with the raft dip; the day weight landing. Frames of a running game only (an editor frame does not move time). Look at them yourself. Use the project's existing way to record frames from the game (search `Tools/` and the reports under `F:\UE\EpicJam\Docs\briefs\reports\` for `ffmpeg` and for the frame recording commands); FFmpeg is installed.
5. Frame time: a run with the sea full of fish shows no visible cost from the startle list (report the average frame time before and after from the log or the stat you choose).
6. `git diff --stat` shows only the files allowed in section 4.

## 7. Verification commands (run from `F:\UE\EpicJam_look`)

```
python -X utf8 Tools/check_all.py --ue
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_check.sh" /f/UE/EpicJam_look wf "catch.HudAutoPack 0,"
git diff --stat
```

Read `run_check.sh` before you copy it for your own runs; do not edit it in place. Frames land in `Catch/Saved/ue_mcp/`.

Build notes. This worktree has never been built: the first build compiles both game modules and takes a few minutes. Another executor builds the Unreal project in `F:\UE\EpicJam_ui`. `Tools/build-editor.ps1` passes `-WaitMutex`, so the two builds queue up: a build of yours may sit and wait for theirs; wait, do not kill it. Never kill an Unreal or build process you did not start; when you must stop a game run of your own, stop it by its own process id. The editor must stay closed (no editor work is needed). Scripted runs use `-benchmark`, which turns the audio device off; that is expected.

## 8. Boundaries

- No rule, number of the game, screen or text beyond this brief. Nothing that changes what the core computes.
- No new assets, no material edits, no new sound files, no downloads, no generators. Nothing from the browser prototype.
- Keep what works: the bite, the nibble, the hook swing, the flights to the bucket, the bucket and scales reactions. Do not delete or hide existing elements.
- No commits, no pushes, no packaging.

## 9. Report

Write `F:\UE\EpicJam_look\Docs\briefs\reports\day2-world-feedback.md` in English and hand back a short summary: per decision what is done and where (file, function), what is not done; the final numbers where you changed them; the check results with log lines; the list of clips and frames with one line each; the frame time before and after; every line you had to touch in `CatchTackleView.cpp` outside the allowed places; questions.

## 10. Chain of command

You report to the orchestrator, not to the owner. Answer technical questions yourself and list them as revocable readings. A question about what the player should feel or see beyond this brief goes into the report, and you continue with the parts that do not depend on it.
