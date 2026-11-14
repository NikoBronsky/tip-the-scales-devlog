# Brief: first draft of the game video (day 3, 14.11.2026 13:05)

Executor: Opus, effort high. You report to the orchestrator, not to the game director. You do not commit.

## 1. Goal

Make the first draft of the jam video for the game "Tip the Scales": 45-60 seconds, 1920x1080, with the game's own sound, cut from real-time takes of the packaged build, with short English captions. Build it as a pipeline that one command can run again on tomorrow's final build.

Why: the jam requires a video of 30-60 seconds at 1920x1080 or more. Owner decision of 14.11: "Ролик снимает агент, ГД смотрит и принимает ролик". The owner returns in 2-3 hours and should find a draft to judge. The earlier the owner sees a draft, the more rounds of his remarks fit before the deadline.

## 2. What exists now (verified by the orchestrator today)

- Build to record: `F:\UE\EpicJam_ui\Packaged\v14\Windows\Catch.exe` (build 15, English, the default game: no player profile, every charm open). The game log: `F:\UE\EpicJam_ui\Packaged\v14\Windows\Catch\Saved\Logs\Catch.log`, UTC time stamps with milliseconds, event lines `CATCH ev ...` and command lines `CATCH cmd ...`.
- The machine has two displays, both 1920x1080. The primary one has a Windows Security Alert dialog on it (see section 7: never touch it). The second display is free. In the engine's monitor list the second display is index 0 (log line `LogWindows: resolution: 1920x1080, work area: (1920, -5) ... device: '\\.\DISPLAY1'`).
- This launch line puts the game full screen on the second display (verified by a frame):
  `Catch.exe -fullscreen -ResX=1920 -ResY=1080 -nosplash -unattended -log -ExecCmds="r.SetRes.MonitorIndex 0,r.SetRes 1920x1080wf,<your commands>"`
- ffmpeg 9.0.2 is on PATH (ffprobe next to it). This input grabs the second display through the GPU and stamps frames with the wall clock (verified: about 56-58 real frames per second, the game itself runs at about 120):
  `-f lavfi -use_wallclock_as_timestamps 1 -t <seconds> -i "ddagrab=output_idx=1:framerate=60:draw_mouse=0,hwdownload,format=bgra" -copyts -c:v libx264 -preset ultrafast -crf 14 -pix_fmt yuv420p take.mkv`
  `gdigrab` reached only 34 frames per second: do not use it. ffmpeg cannot hear the machine's sound output.
- Sound: the console command `catch.RecordSound start`, later `catch.RecordSound stop <name>`, writes the game's own master output to `...\v14\Windows\Catch\Saved\ue_mcp\<name>.wav` (48 kHz stereo, at most 600 seconds). The log line `CATCH_SOUND recording the master output` carries the wall-clock time of the sound start. Verified: the file is not silent (mean -30 dB, peak about -10 dB); the owner listened to a joined test file and said the sound is fine.
- Joining picture and sound by wall clock works: `offset = (log time of the sound start) - (start_time of the take from ffprobe)`; `ffmpeg -ss <offset> -i take.mkv -i sound.wav ...`. The working rehearsal script, read it first and build on it: `F:\UE\EpicJam\Saved\orchestrator_scripts\video_rehearsal.ps1`.
- The NVIDIA overlay shows a green hint in the top right corner for the first seconds after each launch. It must not be in the video: measure how long it stays and start each scene after it is gone.
- Console commands that drive the game: `F:\UE\EpicJam_ui\Catch\Source\Catch\CatchSession.cpp:1126-1345` (`catch.NewRun <seed>`, `catch.StartDay`, `catch.Pull`, `catch.Depth`, `catch.Bait`, `catch.OpenPack`, `catch.Pick`, `catch.Gift`, `catch.Reward`, `catch.Buy`, `catch.Sell`, `catch.Reroll`, `catch.NextDay`, `catch.AutoPull`, `catch.TimeScale`, `catch.CheatRun`, `catch.BiteRun`, `catch.FrameRun`, `catch.RuleRun`, `catch.RuleDays`, `catch.After <seconds> <command>`, `catch.On <event>@<seconds> <command>`, `catch.Dump`, `catch.Shot`, `catch.Record`). The HUD has its own commands (`catch.Hud...`, for example `catch.HudToShop`, `catch.HudAutoPack`) in `F:\UE\EpicJam_ui\Catch\Source\Catch\CatchHud*.cpp`. Read the help strings there. Working scripted flows (title, gift, day, reward, shop, rule days) are in `F:\UE\EpicJam_ui\Catch\SourceArt\Text\run_text_proof.py` and in `F:\UE\EpicJam\Saved\orchestrator_scripts\run_packaged_v25.sh`.
- Caption font: `F:\UE_5.9\Engine\Content\Slate\Fonts\Roboto-Bold.ttf` exists (Apache licence, the engine's own font).
- What the game is, in the owner's words: `F:\UE\EpicJam\CLAUDE.md` section 2. Read it before you choose shots.

## 3. Technical decisions of the orchestrator (each one revocable: change it if it breaks something, and say so in the report)

1. Real-time takes, screen grab of the second display, the game's own sound, joined by wall clock. Main risk: 56-58 grabbed frames per second give repeated frames. Try `t.MaxFPS 60` in the game and, if ffmpeg has it, `h264_nvenc` straight from the GPU frames; keep what gives the steadier picture and report the number of repeated frames.
2. Sound in the video is only what the game plays. No music or effects from outside. You may level the loudness (`loudnorm`, about -16 LUFS) and add fades of 0.1-0.2 seconds at cuts.
3. Each take is a text file with its console commands. One script records a take, one script builds the whole video. The build folder is a parameter.
4. Captions: Roboto-Bold through `drawtext`, white on a soft dark plate or with a shadow, in the lower third, at most 6 words each, at most 8 captions in the video. They describe what the screen shows. The owner will change the wording; write a plain first version.
5. The default English game only: no `-lang`, no `-new`, no `-tasks`. Dynamite, Harpoon and the rule days Poachers and Pest are not accepted by the owner and must not appear.
6. Final file: H.264, yuv420p, 1920x1080, 60 fps, CRF 16-18, AAC 192 kbit/s, `-movflags +faststart`.
7. 3 to 6 takes. Hard cuts or crossfades of at most 0.25 seconds.

## 4. Work by blocks

Block A - pipeline (files in the main checkout `F:\UE\EpicJam`):
- `Tools/video/record_take.ps1 -Build <folder> -Take <name>`: starts the grab, launches the game with the take's commands, waits for the game to quit, writes `Saved/video/takes/<name>.mkv`, copies the wav, writes the wall-clock offset next to them. It stops only processes it started itself.
- `Tools/video/takes/<name>.txt`: one console command per line for each take, with a comment line that says what the take shows.
- `Tools/video/make_video.ps1 -Build <folder> [-SkipRecord]`: records every take (unless `-SkipRecord`), cuts, joins, adds captions, writes `Saved/video/draft1.mp4` and a contact sheet `Saved/video/draft1_sheet.jpg` (one frame every 2 seconds in a grid, with time stamps).
- `Tools/video/edit.json` (or a similar plain file): the cut list (take, in, out), the captions (text, from, to) and the order. The owner's remarks will become edits of this file.

Block B - the shots. Required beats; choose the order yourself, the first 3 seconds must be a strong moment, not a quiet title:
1. The core loop: bait on the hook, the hook sinks, a fish bites, the pull, the fish flies into the bucket, the scale answers.
2. The choice "pull now or wait": a chain, where a predator eats the hooked fish and is caught itself.
3. The shop between days: charms on offer, one is bought, the shelf fills.
4. A charm that visibly changes play (pick charms whose effect the viewer can see without reading).
5. Growth: more rods or hooks, a bigger catch than at the start.
6. Depth: the view rides down to a deeper zone with other fish.
7. The special fish day (Gold Sunfish) if you can stage it.
8. The end of a day: the catch is sold fish by fish, coins pop. Then an end card with the game name "Tip the Scales" (the title screen if it shows the name, else a caption).
A beat that you cannot stage with the existing commands in about 10 minutes: skip it and list it in the report. Do not change game code.

Block C - checks and the report (section 5, 6, 8).

Time box: aim to hand over draft 1 after about 60-75 minutes. A complete plain draft beats a polished half.

## 5. Done means

- `F:\UE\EpicJam\Saved\video\draft1.mp4`: 45-60 seconds, 1920x1080, 60 fps, H.264 and AAC, sound not silent, under 200 MB.
- `F:\UE\EpicJam\Saved\video\draft1_sheet.jpg` exists and you looked at it.
- No frame shows the desktop, a system dialog, the NVIDIA hint, the Windows mouse pointer, debug text, the console, or Russian text. You checked the contact sheet and the first and last frame of every cut.
- Every caption is readable, stays inside the frame with a margin, and does not cover the main action.
- Sound and picture: for at least one cut you measured the gap between a visible event (its `CATCH ev` log time) and its sound in the wav, or you say plainly that you did not measure it.
- One command rebuilds the video from a build folder.
- The gate is green.

## 6. Verification commands

- `ffprobe -v error -count_frames -select_streams v:0 -show_entries format=duration,size -show_entries stream=nb_read_frames,width,height,avg_frame_rate -of default=nw=1 F:\UE\EpicJam\Saved\video\draft1.mp4`
- `ffmpeg -hide_banner -i <file> -af volumedetect -f null -` (mean and peak level).
- `python -X utf8 Tools/check_all.py` from `F:\UE\EpicJam`: expected `10 ok, 0 fail, 1 skip`.

## 7. Boundaries

- Never click, close, move or answer the Windows Security Alert dialog or any other system dialog. Never change Windows, firewall, NVIDIA or audio settings. Never install or download anything. No network.
- Grab only the second display (`output_idx=1`) and only while the game covers it. Do not open or describe frames that show the desktop instead of the game; cut them away.
- One game instance at a time. Never stop a process that you did not start. The takes play sound on the machine: that is allowed, the owner is away.
- No C++ changes, no engine builds, no packaging. Do not write anything under `Catch/` in any worktree, except what the game itself writes into the packaged folder's `Catch\Saved`.
- Do not touch `Docs/journal.md`, `DEVLOG.md`, `CLAUDE.md`, `AGENTS.md`, `Docs/decisions/`, `Docs/design/`. Your files: `Tools/video/**`, `Docs/briefs/reports/day3-video-draft.md`, and anything under `Saved/video/` (not in git).
- Theme rule of the game: no director, no cinema, no wordplay on "cut" in captions. No logos. Age 13+. English only.
- Nothing in the video may come from outside the game except the caption font named above.
- Do not invent game rules or numbers in captions: describe only what the build does.
- Do not commit. Do not push.

## 8. Report

Write `F:\UE\EpicJam\Docs\briefs\reports\day3-video-draft.md` in English: the shot list with times, the captions, the take files and what each stages, the frame-rate finding (repeated frames, which grab settings you kept), the sound-sync number or "not measured", the checks you ran with their output, what is weak in the draft, the beats you skipped and why, the decisions of section 3 that you changed, your questions. In your final message give: the path of the video and the contact sheet, the duration, the three weakest points, and anything the orchestrator must decide.

## 9. Chain of command

Questions go into the report; the orchestrator answers them. A technical question does not stop the work: choose, note it as revocable, go on. Only the orchestrator talks to the owner.
