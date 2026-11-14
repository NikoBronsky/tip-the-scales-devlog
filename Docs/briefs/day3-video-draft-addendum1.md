# Addendum 1 to brief day3-video-draft: make re-recording safe; draft 2 (14.11.2026 13:39)

Executor: the same Opus executor, effort high. Everything in the brief still holds (sections 7 and 9 above all). You do not commit.

## 1. What the orchestrator's check found

The orchestrator ran the one-command path with recording into a separate file:
`make_video.ps1 -Build F:\UE\EpicJam_ui\Packaged\v14\Windows -Edit F:\UE\EpicJam\Saved\video\edit_check.json` (the same edit, output `check_full`).

- Exit code 0, three takes recorded, 0 log errors, 58.5-58.8 grabbed frames per second.
- But `check_full.mp4` is 52.5 s (3150 frames), not 56.7 s. The script printed "10 segments, 56.7 s" and said nothing else.
- Cause: the `chain` take ended at 65.9 s this time. In `Saved/video/takes/chain_events.txt` the first `ChainLink` came at 56.886 s (a Mackerel ate a Mackerel), the pull at 58.084 s, and `{STOP}` 9 s after the first link. `edit.json` cuts 71.6-75.8 s from this take (the BIG CATCH shot), which now lies behind the end of the take.
- So real-time takes do not repeat in time. "Every take is deterministic on seed 42" does not hold for timing. Fixed cut times are valid only for the recording they were made on.
- The re-record replaced the takes that draft 1 was cut from. That is the orchestrator's own mistake: it did not keep them. `Saved/video/draft1.mp4` itself is intact; do not touch it.

## 2. Work

A. Take sets (must). Takes are recorded into `Saved/video/sets/<set>/` (default set name: date and time). A set is never overwritten. The edit file names the set it was cut from (`"set"`). `-SkipRecord` builds from that set; a new recording makes a new set and leaves the older ones alone.

B. Validation (must). Before building: every segment lies inside its take (with the offset), every anchor event is found. Otherwise stop with a non-zero exit code and a table of what is wrong. After building: compare the built duration with the planned one (tolerance 0.1 s) and fail if they differ.

C. Event anchors (should). A segment may be given by an event instead of fixed times: a pattern for a line of `<take>_events.txt`, the occurrence number, and the seconds before and after, for example
`{ "take": "chain", "anchor": "CATCH_MOMENT landing .* CHAIN x", "nth": 1, "before": 4.0, "after": 1.5 }`.
Fixed `in`/`out` stay allowed. Captions stay relative to their segment. In the edit of draft 2 use anchors wherever an event marks the moment.

D. Repeatable takes (test once, then decide). Test whether a fixed time step makes a take repeat while game time and wall time still agree, for example `-benchmark -fps=60` (or the engine's own fixed-step switch) together with `t.MaxFPS 60`. Record the same take twice, compare the event times, and repeat the bucket-landing check of sound against picture. Keep the fixed step only if both hold. Otherwise stay with real time and rely on anchors.

E. Draft 2, cut from a fresh set, with the fixes that need no owner decision:
1. The coin counter must not show test-run money. Stage the shots so that the wallet shows amounts a player could have. If a shot cannot be staged that way, say so in the report.
2. Fewer first-day hint plates in frame, if an existing command or launch option hides them. Do not change game code. Report what exists.
3. The shop shot shows the game's own cursor going to a card before the card is bought (`catch.HudCursor`).
4. The opener. The owner was asked (the chain BIG CATCH or the x4 PULL) and has not answered. Build draft 2 with the chain big catch first: that is the orchestrator's recommendation. Keep the other order as a second edit file (`edit_x4.json`), not rendered.
5. Length stays 45-60 seconds. The Gold Sunfish day is still optional.

Output: `Saved/video/draft2.mp4`, `draft2_sheet.jpg`, `draft2_cuts.jpg`, `draft2_corner.jpg`.

F. At the end run the full one-command path yourself once (a new set and a build). It must either give a video that matches the plan of draft 2, or stop with a clear message. Report which.

## 3. Done means

- A new recording cannot damage an older set or an older video.
- A segment outside its take or a missing anchor stops the build with a clear message and a non-zero exit code.
- `draft2.mp4`: 45-60 s, 1920x1080, 60 fps, sound not silent; the checks of brief section 5 hold; you looked at the sheets.
- The one-command run of block F was done and its result is in the report.
- The gate: `python -X utf8 Tools/check_all.py` gives `10 ok, 0 fail, 1 skip`.

## 4. Report

Append a section "Addendum 1" to `Docs/briefs/reports/day3-video-draft.md`: what changed in the pipeline, the result of the repeatability test with numbers, the shot list of draft 2, what you could not stage, what is not verified. Time box: 45-60 minutes. Final message: paths, duration, shot list, weakest points, what is not verified.
