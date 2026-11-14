# Addendum 4 to brief day3-video-draft: prepare draft 3 on build 17 without recording takes (14.11.2026 20:16)

Executor: the same Opus executor that built the video pipeline (drafts 1-2, the state note in `Docs/briefs/reports/day3-video-draft.md`), effort high. You write in `F:\UE\EpicJam` only under `Tools/video/` and your report; outputs go to `F:\UE\EpicJam\Saved\video\`. You do not commit.

## 1. Goal

The director paused the takes at 15:25 ("приостанови дубли ... потом уже будем запускать ролик") and has not allowed them again. He is busy now and said: "если есть что делать - делай". So prepare everything of draft 3 that needs no take: when he gives the word, recording and cutting must take one command and about 15 minutes. The plan of draft 3 is unchanged: `Docs/briefs/day3-video-draft-addendum3.md` (one music bed, escalation never steps back, the shop once, several depths, a series of different mad builds).

## 2. What changed since your state note (verify before you rely on it)

- The game build for the video is now build 17: `F:\UE\EpicJam_ui\Packaged\v17\Windows` (not `Packaged\v15`). It has the staging aids of v15 (`catch.CheatRun` and friends honour `-new`, `catch.Tips 0|1`) and a new launch argument `-coins=<n>` (start coins, with or without `-new`).
- The dynamite was redesigned by the director. With `-new`: a charm throws a charge that flies, bounces off the walls and blasts after 0.8-3.0 s (also at the day end); blast radius 160; sources Dynamite (a pull that lands a fish, 1 in 5), Powder Keg (each hooked fish landed, 1 in 25), Sore Loser (a fish escapes, 1 in 3), Chain Reaction (each chain link, 1 in 6), Lucky Fuse (any lucky chance that comes true); Horseshoe adds +1 to every chance; Chain Shot throws a harpoon from each chain link; Shockwave lifts the fish one zone per blast; the Bait Bomb throws bucket fish back. The player never throws a charge by hand, so a blast in a take needs the sources on the shelf and enough pulls. Details: `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view210-charge-and-links.md` (the `-new` shop order, the log lines `CATCH ev Blast`, the proof command lines).
- With `-new` the shop's charm slots show the candidates first in a fixed order; a reroll shows the next ones.
- Your open items: `calm3.txt` (the second shop buy fails for lack of coins), the single music bed in `make_video.ps1` (written, untested), `newbuilds.txt` (its blast shot was staged for the old dynamite).

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. No take: no screen grab, no pointer move, no game window. Everything you run is either ffmpeg on existing files or a proof game with `-RenderOffScreen -unattended -nosplash -nosound -log` and in-game frames (`catch.Shot`, `catch.Record`). One game at a time. Before every game start check for a running `Catch` process that you did not start (the director may play build 17); if there is one, wait and tell the orchestrator.
2. Never open, list or describe the take sets `20261114-142512` and `a3`, or the files `draft2_20261114-142512.*`.
3. Test the single music bed with `make_video.ps1 -SkipRecord` on the set `a2c` into a new output name (for example `Saved/video/bedtest.mp4`); do not overwrite `draft2*.mp4`. Check by measurement: one unbroken bed, no clipping, the fade at the end, the level near the target. The doubled music (the `a2c` takes carry game music) is expected in this test only.
4. Bring the take files of draft 3 to build 17: fix `calm3.txt`; re-stage the blast shot of `newbuilds.txt` for the new dynamite so that several charges fly and blast inside a few seconds (several sources and Horseshoe on the shelf, many rods and pulls); check `builds.txt` (depth ride, deep, wide, chains) still stages on build 17. Validate each take file by an offscreen dry run of its command list and look at the frames yourself. Where a take depends on chance, give it a seed and say so.
5. Point `record_take.ps1` / `make_video.ps1` and the edit file of draft 3 at build 17 by a parameter or one constant, so a later build needs one change.
6. Write the edit file of draft 3 (and the cold-open variant) so that `make_video.ps1` can run it as soon as the takes exist.

## 4. Done means

- The music-bed test file exists and your report gives the measured numbers.
- Each draft 3 take file passed an offscreen dry run on build 17; the report lists per take: what the frames show, the seed, the planned length, and any shot that still depends on luck.
- The report ends with the exact one-line command that records and cuts draft 3, and the conditions it needs (nobody touches the mouse, the game covers the second display).
- `git status` in `F:\UE\EpicJam` shows only `Tools/video/**` and your report as changed by you. The three untracked files in the repository root area (`CatchSession.cpp`, `CatchTackleView.cpp`, `Docs/design/text-ru.tsv`) are not yours: leave them.

## 5. Boundaries

Write only under `F:\UE\EpicJam\Tools\video\`, `F:\UE\EpicJam\Saved\video\` and your report `F:\UE\EpicJam\Docs\briefs\reports\day3-video-draft.md` (add a section "Addendum 4"). Do not touch `F:\UE\EpicJam_ui` or `F:\UE\EpicJam_core` except to run the packaged build 17 and read reports. The video shows only what the build does; nothing from outside the game except the caption font already declared and the game's own music file. No downloads. Never click, close, move or answer a system dialog; change no system setting, no audio device, no volume. Do not stop any process you did not start. Delete nothing. No commits. Always absolute paths. Time box: 60 minutes. Report to the orchestrator.
