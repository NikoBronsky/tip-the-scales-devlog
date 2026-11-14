# Brief: two defects of the hook depth drag found by the director in build 18 (14.11.2026 21:47)

Executor: the same Opus executor that built the view and build 18, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base `d690ef2` (build 18, accepted and committed by the orchestrator). You do not commit.

## 1. Goal

The director played build 18 and reported, in his words:

> "крючок прилипает ко дну, если ты его опустил на самый низ, то его невозможно поднять выше. второй момент экран сам не прокручивается когда мы тянем крючок, приходится крутить колесом потом опять тянуть"

Reading of the orchestrator:

1. Defect A: a hook that the player lowered to the very bottom of what the rod can reach cannot be dragged up again. It "sticks".
2. Defect B: while the player drags a hook, the view does not follow. To move a hook further than the visible part of the sea the player must drop the hook, turn the mouse wheel and drag again. He wants the view to scroll by itself during the drag.

Both are view defects. No rule and no number of the core changes; the core stays `694f1b3`.

## 2. What exists (verify before you rely on it)

- The depth drag lives in `Catch/Source/Catch/CatchHud.cpp` around lines 4250-4310: it starts with `DepthAtPixel(Pixel, DragDepth)` and the log line `CATCH_HUD depth drag starts on rod %d at %.0f`, updates `ACatchTackleView::SetDragPreview(DragRod, DragDepth)` and ends with `UCatchSession::SetDepth(Rod, DragDepth)`.
- The mouse wheel moves the view between the surface and the deeper zones (the hint text says so). The orchestrator did not read that code.
- The orchestrator did not reproduce defect A. Candidate causes, none verified: the grab area of the lowest hook lies under the shelf or under the lower edge of the visible sea; the start test of the drag fails when the hook is at the rod's maximum depth; the pixel-to-depth mapping clamps at the bottom so that an upward move is ignored; the view position after a wheel scroll shifts the grab area.
- The video take files (`F:\UE\EpicJam\Tools\video\takes\*.txt`) use the pointer and target commands of the game (`catch.HudCursor`, `target handle`); they must keep working.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. Defect A: find the root cause first and say what it was. The fix must hold for every rod, every zone depth the rod can reach, every view position, with a short and a full shelf.
2. Defect B: while a hook is dragged and the pointer is near the upper or lower edge of the visible sea, the view scrolls in that direction by itself and the hook keeps following the pointer. The speed grows towards the edge. The scroll stops at the same limits as the wheel (never into a locked zone further than the wheel allows). The edge band and the speeds are view feel numbers: choose them, name them in the report. The wheel keeps working during a drag.
3. Small extra, only if it costs under 10 minutes: the Bait Bomb shows no pop-up word when it fires (your open question 1 of build 18). Make it show its tag like the other charms.
4. No other view change.
5. The director is playing build 18 right now (a `Catch` process from `Packaged\v18` runs since 21:44). Do the reading and the code edits now. Before every heavy step (editor build, Live Coding, packaging, a proof game, the gate with `--ue`, `core-test`) check for a running `Catch` game process that you did not start. If one runs, do NOT start the step and do not wait in a loop: finish what needs no build, write the report section with "code ready, not built" and hand back to the orchestrator. The orchestrator resumes you when the game is closed.
6. When no game of the director runs: build, package into `F:\UE\EpicJam_ui\Packaged\v19\Windows` with the same five `.bat` files, and prove on the packaged build. Proof games run with `-RenderOffScreen -unattended -nosplash -nosound -log`, one at a time, `-ExecCmds` quoted.

## 4. Proofs (on the packaged build 19, `-lang=ru -new -coins=300`)

- Defect A: a scripted drag lowers a hook to the rod's maximum depth, releases, then grabs it again and drags it up; the log shows both `depth drag starts` lines and the new depth; a frame after each step. Repeat once with the view scrolled down and once with a full shelf (the shelf must not block the grab).
- Defect B: a scripted drag that holds the pointer at the lower edge: the view scrolls and the hook depth passes the first visible screen; then the same upward. Frames or a short frame sequence, and the log lines.
- The four video take files still stage: run `F:\UE\EpicJam\Tools\video\dry_run.ps1 -Take calm3 -Build <v19>` (it writes only under `F:\UE\EpicJam\Saved\video\dry\`; that one write outside your worktree is allowed) and compare its events with the anchors of `F:\UE\EpicJam\Tools\video\edit_draft3.json` for that take.
- One run without arguments: English, no candidate.

## 5. Done means

The gate with `--ue` has no fail (if `core-test` times out under load, run `Tools\core-test.ps1` alone and report both); build 19 starts with each launch line; the proofs are in the report with frame paths; `git status` shows only files you meant to change; never add `Catch/SourceArt/Fish/atlas.png` or `atlas.json`.

## 6. Boundaries

Write only inside `F:\UE\EpicJam_ui` (and the dry-run output named above). Always use absolute paths. Do not touch `F:\UE\EpicJam_core`. Invent no rule and no core number. Never click, close, move or answer a system dialog. Do not stop any process you did not start: never stop the director's game. Delete nothing; do not touch `Packaged\v14` - `v18`. No commits.

## 7. Report

Add a section "Hook drag fixes (build 19)" to `Docs/briefs/reports/day3-view210-charge-and-links.md`: the root cause of defect A, changes by file, the feel numbers of defect B, the gate, the proofs with frame paths, decisions you changed, open questions, what is not verified. Final message to the orchestrator: the same in short. Time box: 45 minutes of work (waiting for the director's game does not count). You report to the orchestrator, not to the director.
