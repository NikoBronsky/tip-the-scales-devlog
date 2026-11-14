# Report: first draft of the game video (brief day3-video-draft)

Executor: Opus. 14.11.2026. Nothing committed.

## Result

- Video: `Saved/video/draft1.mp4`. 56.7 s, 1920x1080, 60 fps, H.264 yuv420p (CRF 17), AAC 192 kbit/s 48 kHz stereo, faststart, 32.1 MB.
- Contact sheet (a frame every 2 s with time stamps): `Saved/video/draft1_sheet.jpg`.
- Cut check (first and last frame of every segment): `Saved/video/draft1_cuts.jpg`.
- Corner check (bottom right corner once per second, where desktop notifications pop up): `Saved/video/draft1_corner.jpg`.
- Build used: `F:\UE\EpicJam_ui\Packaged\v14\Windows` (build 15, default English game, no launch flags beyond the brief's launch line).

## Pipeline (one command)

- `Tools/video/make_video.ps1 -Build <folder> [-SkipRecord] [-Edit <json>] [-Encoder nvenc|x264]` records every take of the edit file, then cuts, joins, adds captions, levels the sound and writes the video and the three check images. It prints ffprobe and volumedetect at the end.
- `Tools/video/record_take.ps1 -Build <folder> -Take <name>` records one take:
  1. refuses to start when a Catch process already runs;
  2. launches the game full screen on the second display (`r.SetRes.MonitorIndex 0`, `r.SetRes 1920x1080wf`, `t.MaxFPS 60`);
  3. waits for the log line `r.SetRes = "1920x1080wf"`, waits 1.5 s more, and only then starts the ddagrab grab of `output_idx=1` (no desktop frames at the start);
  4. stops the grab (`q` on ffmpeg's stdin) as soon as the log says `CATCH_SOUND recording written`; the take quits about 1 s later, so no desktop frames at the end;
  5. copies the wav and the game log, writes `<take>.json` (wall-clock offset, frames, frames per second) and `<take>_events.txt` (the game's `CATCH ev/cmd/HUD` lines with take time, 0 = sound start). The events file is the tool for choosing cut points.
  It stops only the processes it started. ffmpeg logs go to `Saved/video/takes/<take>-ffmpeg.log`.
- `Tools/video/takes/<name>.txt`: one console command per line, comment lines say what the take shows; `#@ limit <s>`; `{STOP}` = `catch.RecordSound stop <take>`. The recorder adds the setup commands itself (display, cursor, sound start at 3 s).
- `Tools/video/edit.json`: the take list, the segments (take, in, out, note) in order, the captions (text, seg, from, to; times relative to the segment), fade and loudness. Owner remarks become edits of this file; `-SkipRecord` rebuilds in about 1 minute.

## Takes

| Take | What it stages | Real length |
| --- | --- | --- |
| `loop` | Default game, seed 42. Title for 12 s (NVIDIA hint goes away), pack torn, bait picked, start gift (second rod), day 1 with `catch.AutoPull 3`, a forced Sprat card at 27-29.6 s, TARGET REACHED, fish shipped one by one, coin rows, then `catch.BackToTitle` (the title letters fall again: end card). | 51 s |
| `pull5` | `catch.FrameRun 42 Trotline BarbedHooks Worm Shrimp`: the shop, Trotline and Barbed Hooks bought, Sharpen x4, Deeper Line; day 1 with no auto pull; rod 0 holds four fish that never escape; pull at 49 s (x4 PULL, big); auto pull finishes the day; day end. | 64 s |
| `chain` | `catch.FrameRun 42 BarbedHooks FreshScent Worm Shrimp`: charms and Deeper Line in the shop; day 1 at TimeScale 3 (cut away; hunters do not eat on day 1); shop 2: Sharpen x2; prep of day 2: camera rides to depth 520 and back; day 2 with no auto pull; the take pulls rod 0 1.2 s after the second ChainLink: CHAIN x2 on one rod at 61.9 s, the pull of the other at 73.7 s (BIG CATCH, huge). | 81 s |

All three takes are deterministic for seed 42 (two runs of `loop` and of `pull5` gave the same event times to the millisecond).

## Shot list (times in draft1)

| # | Time | Take, in-out | Shows | Caption |
| --- | --- | --- | --- | --- |
| 0 | 0.0-3.5 | pull5 47.8-51.3 | Four fish hang on one rod, one pull lands them, they fly into the bucket (x4 PULL) | none |
| 1 | 3.5-11.9 | loop 15.0-23.4 | START DAY, casts, hooks sink, two bites, two pulls, fish into the bucket, the scales | "Cast, wait for a bite, pull" (4.1-9.1) |
| 2 | 11.9-14.7 | loop 27.0-29.8 | Fish card of a Sprat: zone, size, baits that catch it | "Each fish bites its own bait" (12.1-14.5) |
| 3 | 14.7-21.7 | pull5 12.6-19.6 | Fisher's shop: Trotline and Barbed Hooks bought (cards turn BOUGHT), rod sharpened lv1 -> lv5 | "Charms change the rules" (15.3-21.1) |
| 4 | 21.7-26.2 | pull5 28.5-33.0 | Two rods with three hooks each fill with fish | "More rods, more hooks" (22.0-25.9) |
| 5 | 26.2-31.7 | chain 46.5-52.0 | DAY 2 banner, the view rides down into deeper water (other fish, jellyfish) and back | "Deeper water, other fish" (26.6-31.4) |
| 6 | 31.7-37.4 | chain 58.8-64.5 | A hunter strikes the hooked fish, CHAIN x2, the game hint "Pull now. Or wait..." | "Pull now, or wait?" (32.0-37.1) |
| 7 | 37.4-41.6 | chain 71.6-75.8 | A hunter eats the hooked fish, the pull lands both, BIG CATCH, +23 pts | "The hunter gets hooked too" (38.0-41.3) |
| 8 | 41.6-51.0 | loop 33.6-43.0 | TARGET REACHED, catch shipped fish by fish, DAY 1 OVER, coins to the wallet | "Sell the catch, fish by fish" (42.2-47.6) |
| 9 | 51.0-56.7 | loop 44.8-50.5 | Title TIP THE SCALES (letters fall), fade to black in the last 0.5 s | none |

Captions: Roboto-Bold 58 px, white on a dark plate (0x0b1a26 at 62 %), centred, text top at y = 962 (below the game's own hint plates, above the frame edge), 0.2 s fade in and out. 8 captions, each at most 6 words. No director, cinema or "cut" wording. They describe what the screen shows; "Pull now, or wait?" echoes the game's own hint text.

## Frame rate finding

- Grab kept: `ddagrab=output_idx=1:framerate=60:draw_mouse=0` straight into `h264_nvenc` (`-preset p4 -tune hq -rc constqp -qp 16`), no `hwdownload`. The game runs with `t.MaxFPS 60`.
- Real frames per second in the takes: loop 59.18, pull5 57.71, chain 59.12 (the brief's x264 rehearsal: 56-58). The final 60 fps file therefore repeats about 1-2 frames per second in the loop and chain shots and about 2 per second in the pull5 shots.
- x264 is still in the recorder as a fallback (`-Encoder x264`, also used on its own when nvenc fails to start). I did not record an x264 take this session, so the comparison uses the brief's rehearsal numbers.

## Sound and sync

- Sound is only the game's own output (`catch.RecordSound`), joined by the wall-clock offset (take start 0.92-0.99 s before the sound start). 0.12 s sound fades at every cut, `loudnorm` I = -16, TP = -1.5, a 0.5 s fade at the end.
- volumedetect of draft1: mean -19.0 dB, peak -1.4 dB.
- Measured on the take `loop`: the second fish into the bucket. Log event `FishInBucket #1` at 23.034 s; sound onset in the wav at 23.040-23.050 s; the splash and "+5 pts" first appear in the grabbed picture at about 23.07 s (frame by frame, 60 fps). Picture is about 25-40 ms behind the sound (2 frames or less). For the first fish the sound onset came 83 ms after the event (a quieter onset). I count sync as good for this draft.

## Checks run

- ffprobe draft1: h264 1920x1080, avg_frame_rate 60/1, 3402 frames, duration 56.700 s, size 32,130,563 bytes; aac 48000 Hz stereo 198 kbit/s.
- I looked at the contact sheet, the cut sheet (first and last frame of all 10 segments) and the corner sheet. No desktop, no system dialog, no NVIDIA hint (every segment starts at least 13 s into its take; the hint stays until about 10 s), no Windows pointer (draw_mouse=0), no console, no debug text, no Russian text.
- Every caption crop checked on the sheet; one checked at full size (34 s): readable, inside the frame, not on the game's hint plate.
- `python -X utf8 Tools/check_all.py`: `10 ok, 0 fail, 1 skip`.
- After the takes no Catch or ffmpeg process was left.

## Incidents

- A desktop notification from a messenger app popped up in the bottom right corner of the second display during the first `loop` take (seconds 35-46). It went into the first build of draft1. I re-recorded `loop`, rebuilt draft1, checked the corner sheet (clean) and deleted my temporary frames of it. That is why `make_video.ps1` now writes `<name>_corner.jpg`.
- The real mouse rests over the second display (Windows reports the cursor at x 2666, y 588). The fish card under the mouse ignores `catch.HudCursor`, so the first takes had a fish card stuck at the screen centre. Fix in the recorder: `catch.Hover 999999` forces the card onto a fish id that does not exist (no card); a take shows a card with `catch.Hover <name>` and hides it with `catch.Hover 999999`. I did not move the mouse.

## Weak points of the draft

1. The opener is a "big" x4 PULL without the gold word; the strongest moment (BIG CATCH, gold word) sits at 38-41 s. The fifth hook of the rod never caught a fish, so the x5 PULL (huge) did not happen.
2. Busy frames: the game's own first-day hint plates and the green "this bait takes these fish" glow show in many shots (the glow seems to come from the real mouse over a rod line). The DAY 2 banner covers the first 2 s of the depth shot.
3. The shop shot has no cursor: cards just turn BOUGHT and the rod level jumps. The sale shot is 9.4 s long and mostly a counting panel. No Gold Sunfish.

## Beats skipped

- Beat 7, the Gold Sunfish day: it needs day 3 of a run (two days played first) and a bare hook where the Sunfish swims. Not staged in the time box. A take for it can follow the `charms25`/`core25` scripts of `run_text_proof.py`.
- Beat 5 is covered by rods and hooks (two rods, three hooks each); there is no "rod count grows" shot.

## Section 3 decisions changed

- 1: kept, with h264_nvenc on the GPU frames and `t.MaxFPS 60`. The grab now starts after the game covers the display and stops before it quits (the rehearsal grabbed from 2 s before launch).
- 4: caption position is the bottom (text top at y 962), not the middle of the lower third, because the game's own hint plates sit at y 880-940.
- All others as written.

## Not verified

- One full `make_video.ps1 -Build ...` run with recording in a single command. Each take was recorded by `record_take.ps1` (the same call make_video makes) and the video was built with `-SkipRecord`.
- The video was not played back with sound by a person; only levels and one onset were measured.
- The Windows Security Alert dialog on the primary display was not touched and not looked at.

## Questions for the orchestrator

1. Can the mouse be parked on the primary display before tomorrow's recording, and can the messenger notifications be silenced (Focus assist) by the owner? Both are machine settings I must not change; both make takes cleaner.
2. Opener: keep x4 PULL, or open with the chain BIG CATCH (gold word) and move x4 PULL into the growth shot?
3. Should the Gold Sunfish day get its own take (about 3 minutes of recording) in draft 2?
4. Caption wording is a plain first version for the owner to change.

## Addendum 1 (14.11, blocks A-D; block E withdrawn by the orchestrator, block F not run)

### Pipeline changes

- A, take sets: `record_take.ps1 -Set <name>` writes into `Saved/video/sets/<set>/` and refuses when the take already exists there (json or mkv). The take file is copied into the set. `make_video.ps1` without `-SkipRecord` makes a new set named by date and time, records into it, and writes `<output>_<set>.mp4` unless `-Output` is given. With `-SkipRecord` it builds from the edit file's `"set"` (or `-Set`). An existing output video is never replaced without `-Force` (tested: exit 1, "exists; give another -Output or -Force"). The old `Saved/video/takes/` (the orchestrator's re-record) is now the set `20261114-1340-orchestrator-check`; `draft1.mp4` untouched.
- B, validation: before building, every take must exist in the set with an offset, every anchor must be found, every segment must lie inside its picture and its sound; otherwise a table of the plan and a list of problems, exit code 1, nothing built. After building, the duration must match the plan within 0.1 s, or exit code 1. Tested on `Saved/video/edit_check.json` with the check set: "segment 7 (chain): out 75.8 lies after the take end 65.877", "FAILED: 1 problem(s); nothing built", exit code 1, no file written. The resolved cut list is written to `Saved/video/<output>_plan.txt`.
- C, event anchors: a segment may be `{ "take", "anchor": <regex on a line of <take>_events.txt>, "nth", "before", "after" }`; fixed `in`/`out` still work. Tested on the check set (`Saved/video/build/anchor_test.json`): chain BIG CATCH resolved at 58.367 s, the first FishInBucket of `loop` at 21.917 s; `anchor_test.mp4` 7.5 s planned and built, exit 0.
- Per-take launch arguments (asked by the orchestrator for addendum 2): a take file line `#@ args -new` adds launch arguments; they are logged in `<take>.json` (`launch_args`). Not yet used in a recording.

### D, fixed time step: rejected

Take `Tools/video/takes/run.txt` (a real default run: days 1-2 at TimeScale 3, day 3 the Gold Sunfish day) recorded twice:

| Set | Switch | Result |
| --- | --- | --- |
| `testA` | `-benchmark -fps=60` + `t.MaxFPS 60` | The game ran unthrottled: the whole run (about 120 s of game time) took 31.0 s of wall time. No sound: the log says "LogAudioMixer: Error: Output recording is an audio mixer only feature" (benchmark runs without the audio mixer), so no wav. |
| `testB` | `-UseFixedTimeStep -FPS=60` + `t.MaxFPS 60` | Sound recorded, but the game still ran ahead of the wall clock: day 3 is 35 s long and passed in 9.0 s of wall time (StartDay at 18.30 s, TargetReached "0.0 s saved of 35" at 27.32 s). `t.MaxFPS` does not limit a fixed-step run. |

Game events did repeat between the two fixed-step runs (same fish, same hunters 165 and 237, same points; wall times 0.2-0.4 s apart), but game time and wall time do not agree, so sound and picture would not match game speed. Decision: real time stays; repeatability comes from the anchors. `-FixedStep` remains in `record_take.ps1` as a documented, unused switch. The sound-against-picture landing check was not repeated (no usable fixed-step take).

Side finding from these runs, for part 1 of addendum 2: the real default run (seed 42, no cheat) reaches day 3 with 18 coins; on day 3 a bare hook catches the Gold Sunfish (PrizeEarned, BIG CATCH) and a hunter makes CHAIN x2 twice.

## Addendum 2 (14.11): stopped at the cursor

Decision 4.1 of addendum 2 says: if the game cursor cannot be made visible with existing commands, stop and report. It cannot. I stopped before recording any take for draft 2.

### The cursor finding

- The game draws no cursor of its own. `ACatchPlayerController` sets `bShowMouseCursor = true` (`CatchPlayerController.cpp:10`): the visible cursor in the game is the Windows pointer, with no custom shape (no cursor setting in `Catch/Config/*.ini`).
- `catch.HudCursor x y [seconds]` (`CatchHud.cpp:1473` `SetFakeCursor`, read by `CursorRef` at `CatchHud.cpp:1457`) moves an invisible point. The HUD uses it instead of the mouse for hover states: shop tips (`CatchHudShopTip.cpp:104`), the cursor light on panels (`Ui::Fx::CursorLight`, `CatchHudMenu.cpp:524`), lit rims of cards and buttons. Nothing draws an arrow or a hand there, and no click is shown.
- The fish card under the cursor does not even use this point: it reads the real mouse (`MousePixel` from `NativeOnMouseMove`, `CatchHud.cpp:3815`).
- `catch.HudClick` acts at a point without moving anything visible.
- The grab hides the Windows pointer (`draw_mouse=0`), as the brief demanded.

### Ways forward (the orchestrator decides; none started)

1. Show the real Windows pointer (`ddagrab draw_mouse=1`) and let the recorder move it on the second display (`user32 SetCursorPos` along a path), with the action fired by `catch.HudClick` when it arrives. No game code. The fish cards and hovers would then follow a real mouse. Costs: it moves the owner's mouse pointer during takes, which the brief does not cover. It shows the Windows arrow, which brief section 5 forbids. Clicks are not visible.
2. A small game change: the HUD draws a cursor sprite (a hand, with a press state) at the fake cursor point, plus a console switch. Needs a code executor and a new build; then the recorder only needs `catch.HudCursor` paths.
3. Not allowed by decision 4.1: a cursor drawn by ffmpeg.

### The hint-plate finding (for part 1)

- No command or launch option hides the hints. `MakeHintView` (`CatchHud.cpp:4669`) shows the hint bar in the day only on days 1 and 2. In every preparation it shows "Pick a bait and a depth. Press START DAY." (on day 1 the longer text). Teaching tips ("Pull now...", "Chain!...", "The fish got away...") show once per session, the first time each happens.
- Way without code: shoot part 1 on day 3 or later of a real run (days 1-2 at TimeScale 3, cut away; see the `run` take above). Then the day itself has no hint plate, but the first bite of the session still brings the "Pull now. Or wait..." tip for 5.5 s. A pull before that tip is mostly hidden by the take's own timing.

### Not done

- No draft 2 (neither `draft2.mp4` nor `draft2_coldopen.mp4`), no `edit_coldopen.json` or `edit_no_candidates.json`, no part 4 frame-rate measure, no block F run: all wait for the cursor decision.
- `Tools/video/edit.json` still describes draft 1 and names no set (its takes are gone); `-SkipRecord` on it stops with "no set".

### Checks

- `python -X utf8 Tools/check_all.py`: `10 ok, 0 fail, 1 skip`.
- Both scripts parse without errors. After every take no Catch or ffmpeg process was left.

## Addendum 2, continued (14.11, after the orchestrator chose cursor option 1: the real Windows pointer)

### Result

- `Saved/video/draft2.mp4`: 48.3 s, 1920x1080, 60 fps, AAC 48 kHz, mean -17.9 dB. Sheets `draft2_sheet.jpg`, `draft2_cuts.jpg`, `draft2_corner.jpg`, plan `draft2_plan.txt`.
- `Saved/video/draft2_coldopen.mp4`: 50.1 s, the same video after a 1.8 s flash of the madness (mean -18.3 dB), with its sheets.
- `Tools/video/edit_no_candidates.json`: 48.5 s, not rendered (part 4 without the -new take).
- All three edit files name the set `a2c` and use event anchors only.
- The draft2 files were rebuilt once more with one change: the two day 4 pulls of part 3 now play in time order.

### Pipeline changes

- `Tools/video/pointer.ps1` (new): the visible hand. `record_take.ps1` starts it once the game covers display 2; it reads the game log and mirrors every `catch.HudCursor x y s` the take fires with the real pointer: `SetCursorPos` only, an eased path with a slight arc, about 60 steps per second, clamped to display 2 (x 1920..3839, y -5..1074). No buttons, no keys, no SendInput: every action stays a console command (`catch.HudClick`, `catch.Buy`, ...). `catch.HudCursor 5000 <y> <s>` means "the rod whose hooked fish `catch.HudClick fish` will pull" (the pointer follows FishHooked and PullStarted events). Points off the viewport (the old park 2200 1300) leave the pointer where it is.
- `record_take.ps1`: grabs with `draw_mouse=1`; saves the pointer place before a take and puts it back after; the hand reports glides and "touched" (the pointer found where the script did not put it; one step of lag during a glide is tolerated); `<take>.json` gains `pointer_glides` and `pointer_touched`, and a warning says "record it again" when touched. It refuses a take whose command line is longer than 15800 characters (the game reads at most 16384; a longer take started without its commands: set `a2c`, files `madness_plain-failed-cmdline.*`).
- `make_video.ps1`: an end-card segment `{ "card": "Tip the Scales", "duration": 2.5 }` (the game name in Roboto-Bold on the sea colour, silent, no take); a caption may take `"x"` (an ffmpeg expression), used to keep parts 3-4 captions right of the shelf.
- Take files: `calm`, `grow`, `madness_plain`, `madness` (each explains its staging at the top), `probe` (pointer test and handle places, not for the video). Every take of set `a2c` reports `touched 0`.

### Shot list of draft2.mp4 (set a2c; times in the video)

| Video | Take, anchor | What the viewer sees | Caption |
| --- | --- | --- | --- |
| 0.0-8.8 | calm, 2nd fish click on day 3 | Part 1. Day 3 of a real run (seed 42, 17 coins, two rods: the start gift card 0 is a rod). The pointer rests, glides to the hooked fish and pulls; the fish flies into the bucket. | Cast a line. Pull a fish. (centre) |
| 8.8-14.9 | grow, NEW ROD bought | Part 2. Shop after day 2 on 13 earned coins: the pointer buys NEW ROD (8), SHARPEN (2), a chum card (3). | Catch more fish. |
| 14.9-19.6 | grow, 1st fish click on day 3 | Part 2. Three rods; the pointer pulls a bite; the Gold Sunfish lands on the next rod (PRIZE). | (same) |
| 19.6-22.0 | madness_plain, DEEPER LINE bought | Part 3. Shop after day 3 of the cheat run; the shelf already holds six charms; the pointer buys DEEPER LINE. Wallet 1808 -> 1798. | MORE FISH. (right) |
| 22.0-28.0 | madness_plain, 1st and 2nd handle click | Part 3. Day 4: five rods, five hooks each, Giants and Whopper on the shelf; the pointer pulls a full rod, then the next (CHAIN x2). | (same) |
| 28.0-31.8 | madness_plain, 4th handle click | Part 3. x4 PULL, TARGET REACHED. | (same) |
| 31.8-34.0 | madness_plain, BIGGER SHELF bought | Part 4. Shop after day 4: Cluster Hook, Tiny Terror, Food Chain added; the pointer buys BIGGER SHELF. | Dive into the fish madness. (right) |
| 34.0-37.0 | madness_plain, 6th handle click | Part 4. Day 5, the full shelf (nine charms): x10 PULL, a Tuna eats a hooked fish (CHAIN x4). Wallet 1743. | (same) |
| 37.0-40.0 | madness (-new), day 4 x5 PULL | Part 4. Dynamite and Harpoon on the shelf: one pull, the blast and the harpoon, x5 PULL. Real -new run, 3 coins, four rods. | (same) |
| 40.0-42.8 | madness_plain, 7th handle click | Part 4. x16 PULL. | (same) |
| 42.8-45.8 | madness_plain, 8th handle click | Part 4. CHAIN x5, fish rain into the bucket. | (same) |
| 45.8-48.3 | card | Tip the Scales | - |

Part lengths: 8.8 / 10.8 / 12.2 / 14.0 s, end card 2.5 s. `draft2_coldopen.mp4` adds 1.8 s of the -new x5 PULL first; the rest is the same.

### How the states were staged

- Parts 1-2: a real default run, seed 42, no cheat. Days 1-2 at TimeScale 3 with the auto pull (cut away). `calm`: day 3 at real speed, bare hooks (the day rule), pointer pulls 2.4-3.4 s after a bite, auto pull 1.2 s before an escape. `grow`: the same run, but the shop after day 2 buys on camera; day 3 with three rods. In `calm` the run ends after day 3 (38 of 44 pts); in `grow` day 3 passes (44 of 44).
- Parts 3-4 (`madness_plain`): `catch.CheatRun 42 !timed SeaChest BigShoals Trotline ~BarbedHooks ~Giants ~Whopper ~ClusterHook ~TinyTerror ~FoodChain` (+2000 coins; timed mode so a day runs its full time and every pull counts). Days 1-3 at TimeScale 5, cut away. Off camera: Bigger Shelf, rods 2-5, sharpening to level 5 (three hooks; Trotline adds two), Deeper Line, chum; Barbed Hooks, Giants, Whopper after day 3; Cluster Hook, Tiny Terror, Food Chain after day 4 (`catch.Buy charm <name>` with rerolls). On camera: DEEPER LINE, BIGGER SHELF (level 2), chum. Pulls: `catch.HudClick handle <rod>` after the pointer arrives (Barbed Hooks: no escapes).
- Shelf places (section 5a, point 4): five base + Bigger Shelf twice (+2) + Sea Chest (+3 at 30 coins or more) = 10 places, nine charms used.
- `madness` (-new): see below.

### What I could not stage, and why

- Dynamite and Harpoon in the same build as the rest (section 5a). The cheat runs (`CheatRun`, `FrameRun`) build their tables with `MakeDefaultTables()`, without the `-new` candidate switch (`CatchSession.cpp:599, 728`); the shop never offers a candidate then (`Game::CharmOpen` -> `CharmInGame`, `Profile.cpp:130-139`). So `madness` is a real `-new` run: Dynamite and Harpoon after day 1 on 20 start coins, four rods by day 4, 3 coins in the wallet, a sparse sea (each blast removes 20 %). It shows both charms firing in the same pulls, but not with Cluster Hook, Tiny Terror and five hooks. A code change (the cheat runs use `SessionTables()`) would allow the combined shot.
- The view riding down to the deeper zones (part 3): it happens in the preparation, which shows the hint plate "Pick a bait and a depth. Press START DAY." Cut out; the deeper zones show only in the depth gauge.
- Part 1 with one rod: the real run needs the start-gift rod to pass days 1-2.
- The wallet in parts 3-4 is 1743-1808 (cheat coins, not round); in the -new shot it is 3.

### Hint plates

- Part 1 (day 3) has no day-hint bar. The teaching tip "Pull before it slips away." shows at the top for the first 1.5 s of part 1 (first bite at real speed in that session). Parts 2-4 show no hint bar; the part 2 day 3 shot shows the Gold Sunfish rule box and PRIZE words (game events, not hints).

### Frame rate of the grab (part 4)

- madness_plain day 5 (the fullest shots, 126.9-146.4 s of the take): 58.9 frames per second; day 4: 59.2; madness day 4: 59.0. The longest gap between grabbed frames is about 117 ms in each, once or a few times per take: short hitches. The game does not drop under 60 for long; the cut video is 60 fps (frames repeated in gaps).

### Block F (one-command run)

`make_video.ps1 -Build F:\UE\EpicJam_ui\Packaged\v14\Windows -Edit Tools\video\edit.json` (before the last order fix) recorded the new set `20261114-142512` (calm, grow, madness_plain, madness; each touched 0, 59.0-59.3 grabbed frames per second, 0 log errors), resolved every anchor and built `Saved/video/draft2_20261114-142512.mp4`: 48.3 s planned = 48.3 s built, exit code 0. The shots land on the same kind of moments (the anchors hold in a new recording). But its contact sheet shows a desktop notification in the bottom-right corner during the part 2 shop (about 8-13 s): that video and set must not be used for the owner. I did not open its corner sheet.

### Not verified

- After the last order fix (the two day 4 pulls in time order) both drafts were rebuilt and checked by plan, duration and sound; I did not look again at every cut. I looked at the sheets and cuts of the build before it.
- `draft2_corner.jpg` and `draft2_coldopen_corner.jpg` were not opened (rule: no frames with notifications); the 2 s sheets of set `a2c` show none. Look at the corner sheets before showing the owner.
- The sound against the pointer: the hand reacts to the log, about 0.05-0.2 s after the command; actions are timed 0.3-1 s after the glide ends, so the pointer arrives first, but a click is not visible (Windows draws no press).
- The pointer shows the Windows arrow, which brief section 5 did not want; the orchestrator chose it.

### Decisions for the orchestrator

1. Accept the -new shot as it is (a real run, modest), or order the code change so `CheatRun` honours `-new`, then record `madness` again with the full build plus Dynamite and Harpoon.
2. Cold open or not (`draft2.mp4` vs `draft2_coldopen.mp4`): for the owner.
3. The end card is drawn by ffmpeg (the game name only). The game's own title screen would need one more take (BackToTitle at the end of a take).


## Addendum 3 (draft 3): paused by the director

The director stopped the takes on 14.11 at about 15:28. Draft 3 is not built. No video was made from partial takes.

### What is ready

- `Tools/video/pointer.ps1`: the visible Windows pointer. It follows `catch.HudCursor` lines in the game log with eased glides inside display 2. `x = 5000` means "the handle of the rod that holds a fish". It reports glides and touches.
- `Tools/video/record_take.ps1`: grabs with the pointer, saves and restores the pointer place, refuses a command line over 15800 characters, and sets `catch.Tips 0` and `catch.MusicVolume 0` at the start and again at 1 s and 12.2 s. The music setting is not stored: `catch.MusicVolume` is not saved, and no menu step was used. The build's `Saved` folder has no music entry.
- `Tools/video/make_video.ps1`: card segments (end card) and caption `x`. The new `music` block in the edit file puts one unbroken bed under the whole video: `S_Music.wav` from `start` for the full length, the effects at `effects_gain_db`, `amix`, then `loudnorm` to the target, and a fade-out of the bed over the card. Per-segment effect fades come from `fade` (0.04 s planned). This mix is **not tested** yet.
- Take files in `Tools/video/takes/` (made by a generator script in the scratchpad, which may be lost; the take files themselves are complete):
  - `calm3.txt`: a real run, seed 42, gift card 1, day 1 with one rod, a 5 s shop with the pointer buying, day 2 with two rods. Known fault: the second buy (Sharpen) fails with NotEnoughMoney (7 coins after day 1). Change the shop to New Rod plus a 1-2 coin item, or use gift card 0.
  - `builds.txt` (about 10k characters, no `-new`): four FrameRun builds, each filmed on day 1: A depths (3 rods at 150/520/1000, camera ride 0 → 2950 → 0), B deep (one rod with Trotline, Extra Edge, Bottom Rig, Old Faithful, Deeper Line x5, line to 3300, camera down and back with the pull), C wide (Auto-Reel, Rod Rack, chum, 5 rods level 1), D chains (Tiny Terror, Fresh Scent, Food Chain, Apex Hunter, 5 rods level 3, pull row).
  - `newbuilds.txt` (`#@ args -new`, about 11.9k characters): E harpoons (Harpoon, Big Shoals, 4 rods level 3), F blasts (old Dynamite, Cluster Hook, Big Shoals, 5 rods), G finale (Sea Chest, Dynamite, Harpoon, Trotline, Cluster Hook, Big Shoals, Tiny Terror, Food Chain, 5 rods level 5, Bigger Shelf x2, pull row of 8).

### Set `a3`: no take is usable

| Take | State | Pointer |
| --- | --- | --- |
| `calm3` | complete, 71 s, sound 70.5 s | touched 812 times in 0-34 s (somebody moved the mouse, first on display 1). Discard. |
| `builds` | complete, 164 s, sound 160.4 s | touched 1299 times. Discard. |
| `newbuilds` | stopped by me at 20 s on the director's request, no wav | touched 402 times. Discard. |

The touches came from a person at the computer, not from the game: the game has no code that moves the Windows pointer. Takes with the visible pointer need a free mouse for the whole take (calm3 70 s, builds 165 s, newbuilds about 200 s).

### State and what is left

- No game, grab or pointer process of mine runs. The pointer was put back each time.
- Nobody opened the frames or corner sheets of set `a3`. Before set `a3` is used, look at its corner sheets.
- Left: record `calm3` (shop fix first), `builds` and `newbuilds` on the newer build with the mouse free; check the touches, the corner sheets and the event anchors; write `edit.json` for draft 3 (order: calm, shop, depth ride, wide, deep, chains, harpoons, blasts, finale, then the card; 2.5-4 s for each build shot; music block); render `draft3.mp4` and `draft3_coldopen.mp4`; check the levels around each cut; run the gate.
- On a newer build, check first: the console names of the charms, `-new` in CheatRun and FrameRun, `catch.Tips 0`, and the new dynamite. Take files call charms by name, so a renamed charm will make a `Buy` fail. The `CATCH_` events in the take log show this.
- `edit.json` still describes draft 2 (set `a2c`).

## Addendum 4: draft 3 prepared on build 17, no take recorded

Brief `Docs/briefs/day3-video-draft-addendum4.md`. No take, no grab, no pointer move, no game window. Every game ran off
screen (`-RenderOffScreen -unattended -nosplash -nosound -log`), one at a time, after a check for a running `Catch`
process (there was none each time). I did not open the sets `20261114-142512` and `a3`.

### State checked after the reboot

My files under `Tools/video` were all there (the take files, scripts, edits); the take generator in the scratchpad also
survived. It now lives in `Tools/video/takes/gen_draft3.ps1`.

### One constant for the build

`Tools/video/video_build.txt` (first line: `F:\UE\EpicJam_ui\Packaged\v17\Windows`). `record_take.ps1`,
`make_video.ps1` and `dry_run.ps1` read it when `-Build` is not given. A newer build needs one line changed.

### Music bed test (`Saved/video/bedtest.mp4`, set `a2c`, edit `Tools/video/edit_bedtest.json`)

The single bed works. Measured on the finished file (`Tools/video/audio_check.py`, ffmpeg `ebur128`):

- Integrated loudness -15.7 LUFS (target -16), LRA 2.5 LU, sample peak -0.98 dBFS, 0 clipped samples.
- 48.3 s, 13 segments, effect fades 0.04 s. The bed is one input from 0 to the end (one `atrim` of `S_Music.wav`), so it
  cannot restart at a cut.
- At each of the 12 cuts, the 10 ms levels from -60 ms to +60 ms stay between -32 and -12 dB: no gap to silence (the bed
  carries the cut). The largest jumps (11-12.5 dB at 28.0 s and 34.0 s) come 10-20 ms after the cut, inside the 40 ms
  fade-in, as a loud take sound starts. No click shape (no single-window spike).
- The quietest 100 ms before the card: -28.4 dB; median -21 dB.
- Fade over the card: -24 dB at 45.8 s, then -29, -35, -43 dB by 47.8 s.
- Expected in this test only: the `a2c` takes carry game music, so the music is doubled here.

`S_Music.wav` tempo: 75 bpm (0.8 s beat), first beat at 0.035 s (onset autocorrelation). Draft 3 starts the bed at
0.035 s and puts every cut on the 0.8 s grid.

### Offscreen dry runs (`Tools/video/dry_run.ps1`, frames in `Saved/video/dry/<take>/`)

`dry_run.ps1 -Take <name> [-Every <s>]` runs the take's own command list off screen, adds `catch.Shot` frames, and
writes `<take>_events.txt` and a sheet. I looked at the sheets and at single frames.

| Take | Seed | Length | What the frames and events show |
| --- | --- | --- | --- |
| `calm3` | 42, `-coins=8` | ~70 s | Day 1 with one rod (Sharp Rod gift), 3 pointer pulls, 24 pts, coins 16. The one shop: Buy New Rod Ok, Sharpen rod 2 Ok twice (frame `t0460`: ROD 2 lv 3, coins 5), NEXT DAY. Day 2 with two rods. |
| `builds` | 42 | ~175 s | A depths: three rods at 150/520/1000, the view rides to the seabed (frame `t0270`, the Whopper in Twilight). B deep: one rod lv 7/8, five Flounders on the seabed, then one pull x5 (124 pts each) and the ride up (frame `t0780`). C wide: five rods lv 1, a self-pull every 0.3-1 s on all five rods. D chains on DAY 2: CHAIN x5, x5, x5, x7, x3 in the pull row. |
| `newbuilds` | 42, `-new` | ~92 s | E harpoons: four pulls of x3-x5, one harpoon each. F blasts (new dynamite): shelf 10/10 (Dynamite, Powder Keg, Sore Loser, Chain Reaction, Lucky Fuse, Horseshoe, Sea Chest, Tiny Terror, Big Shoals, Cluster Hook); in the second pull row 12 charges thrown and 7 blasts in 4 s, pulls up to x26 (frame `t0840`). |
| `finale` | 42, `-new` | ~50 s | Shelf 15/14: Dynamite, Lucky Fuse, Horseshoe, Harpoon, Chain Shot, Sea Chest, Rod Stand, Big Shoals, Tiny Terror, Cluster Hook, Fresh Scent, Barbed Hooks, Trotline, Food Chain, Apex Hunter; five rods lv 5 with five hooks. In the chosen 4 s: 6 charges thrown, 5 blasts, 6 harpoons, pulls x6-x10 (x21 shortly after). |

What I changed in the take files (all through `gen_draft3.ps1`):

1. `calm3`: `#@ args -coins=8` (8 start coins), three buys: New Rod (5), Sharpen of the new rod twice (2 + 4). The
   pointer goes to NEW ROD (408, 842), then the new rod's SHARPEN (408, 832), then NEXT DAY. The shop shot is 4 s.
2. `newbuilds`: the old blast shot is replaced by the new dynamite build F (above). It has no Barbed Hooks, so
   escapes feed Sore Loser. Two pull rows, 0.8 s apart per rod.
3. `finale` is a new take file (`-new`). `newbuilds` could not hold three builds of this size under 15800 characters.
4. Chains on day 2: the core lets no fish eat a hooked fish on day 1 (`eatFromDay` 2). The first D dry run had no chain
   at all. Build D now plays day 1 at core speed x8 off camera (`catch.TimeScale 8`), lands fish twice mid-day for the
   target (108 of 22 pts), buys three Worm Tins in the day 2 shop, and is filmed on day 2.
5. Every rod takes the Worm (`catch.Bait r Worm`). `Bait r next` had left rods 1-4 on a bare hook. Cheat shops buy two
   Worm Tins.
6. The pointer goes to the handle height the game reports (`target handle at x y`): y 610 for level 3 rods, 807 for
   the finale.
7. A tried finale on day 2 (for chains) failed: no bait was left and day 1's blasts had emptied the sea (0 pts, run
   over). The finale stays on day 1, so it has no chains; the chains shot carries them.

### Edit files of draft 3

`Tools/video/edit_draft3.json` (46.4 s) and `Tools/video/edit_draft3_coldopen.json` (48.0 s, a 1.6 s flash from a later
finale moment in front). Every anchor was found in the dry run events. Cuts on the 0.8 s beat grid.

| # | Take | Shot | Length | Busiest event (dry run) |
| --- | --- | --- | --- | --- |
| 0 | calm3 | part 1: day 1, one rod, the pointer pulls | 8.0 | a pull of 2 into the bucket |
| 1 | calm3 | part 2: the one shop, 3 buys | 4.0 | NEW ROD, SHARPEN x2, NEXT DAY |
| 2 | calm3 | part 2: day 2, two rods | 4.0 | a pull |
| 3 | builds A | part 3: ride from the surface to the seabed | 5.6 | three depths, the Whopper |
| 4 | builds C | wide: 5 rods lv 1, Auto-Reel | 3.2 | 4 self-pulls |
| 5 | builds B | deep: one rod, 5 hooks on the seabed | 4.0 | x5 Flounder pull, ride up |
| 6 | newbuilds E | harpoons, 4 rods | 3.2 | x5 + harpoons |
| 7 | builds D | chains, 5 rods, day 2 | 4.0 | CHAIN x5, CHAIN x7 |
| 8 | newbuilds F | blasts, 5 rods | 4.0 | 12 charges, 7 blasts, x26 |
| 9 | finale | everything, 15 charms | 4.0 | 6 charges, 5 blasts, 6 harpoons |
| 10 | card | Tip the Scales | 2.4 | the bed fades out |

Mix: the bed `S_Music.wav` from 0.035 s, 0 dB; effects 0 dB, 0.04 s fades; `loudnorm` to -16 LUFS; bed fade 2.4 s over the
card. The balance of bed and effects was measured only with the doubled `a2c` sound. The first real draft 3 needs a
listen, then perhaps `effects_gain_db`.

`make_video.ps1` now also refuses a take whose `pointer_touched` is above 0. With `-Set` in record mode it records into that
set name, so the cold open can cut the same set.

### The command (when the director allows takes)

In PowerShell, from any folder:

```
$s='d3-'+(Get-Date -Format MMdd-HHmm); & F:\UE\EpicJam\Tools\video\make_video.ps1 -Edit F:\UE\EpicJam\Tools\video\edit_draft3.json -Set $s -Output draft3; if ($?) { & F:\UE\EpicJam\Tools\video\make_video.ps1 -SkipRecord -Edit F:\UE\EpicJam\Tools\video\edit_draft3_coldopen.json -Set $s -Output draft3_coldopen }
```

It records `calm3`, `builds`, `newbuilds` and `finale` into `Saved/video/sets/d3-<date>/`, then cuts `draft3.mp4` and
`draft3_coldopen.mp4`. Time: about 8 min of takes and 3-4 min of cutting.

Conditions:

- Nobody touches the mouse for the whole run. A touched take stops the cut.
- The game covers the second display, and no other `Catch` runs.
- No notification and no firewall prompt during the takes. Look at the corner sheets after the run.

### Not verified, depends on luck

- No real take was recorded, so no real sound, no pointer, no real-time timing. The dry runs used the same command lists and
  game time, but rendered off screen.
- Luck: the charges (F, finale), the chains (D) and the harpoon targets depend on chance and on frame timing (seed 42 fixes
  the run, not the frame order). The edit cuts relative to pulls, so a weaker roll gives a weaker shot, not a broken cut.
  Other shots repeat closely.
- The `calm3` shop needs at least 11 coins after day 1. The dry run had 16.
- The candidate charms show red TEMP tags on their shelf pictures (the temporary art of build 16/17). They will be visible
  in the blast and finale shots.
- "TARGET REACHED" banners can show in the madness shots; Rush/Calm labels show the timed mode of the cheat runs.
- The finale has no chains (day 1).
