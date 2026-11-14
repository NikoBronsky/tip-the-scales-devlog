# Brief: day one, step 5 — the playable version on the aligned core, hover card, water rim

Status: written and handed out on 12.11.2026 at 10:35 (owner's clock). Executor: Opus, effort high (the executor of `day1-playable.md` continues).

## 1. Goal

The first playable version now runs on the rules core that equals the play the owner approved. Make every changed flow of the core correct on screen. Then add two things the owner already decided: the card that shows when the cursor is on a fish, and the bright rim at the water line.

Owner's decisions behind this step:
- «Игрок наводит курсор на рыбу и видит, что это за рыба и на какую наживку она ловится.»
- Water look, decision `water-look` of 12.11: «A со светлой кромкой».

## 2. What exists now (verified by the orchestrator on 12.11 at 10:30)

- `main` is at `48da98e`. It holds your first playable version (`e89f4a0`) and the aligned core (merge `2d7545b`). Your report: `Docs/briefs/reports/day1-playable.md`. I checked it: gate with the editor build is green, a `-game` run plays by console commands, I viewed your frames. The fish fix is correct against `T_TempFish.png`.
- The editor target builds with the new core without an edit in `Catch/Source/Catch/`. A `-game` run on the new core plays one full day: fish swim in the whole sea from the first frame of the preparation, the day end lands the hooked fish, shipping, summary.
- The core changed flows. Read these first: `Docs/design/core-interface.md` section 10 (the part "Flows that changed") and `Docs/briefs/reports/day1-core-align.md`. The rules in words: `Docs/design/rules-reference.md` (sections 1, 3-6, 9-10, 14 and the new section 18).
- Seen in my run: at the start the pack is not open; `catch.Pick 2` returned NothingToDo and the panel shows "Open Small Pack". `catch.Dump` prints the hang time as 6.0; the reference says 7 s (the core executor checks its side now).
- Another executor works in the worktree `F:\UE\EpicJam_core` on a small data follow-up of the core. It does not touch `F:\UE\EpicJam`. I merge its result when you are not building.
- I record the motion clips for the owner myself. You do not record them again.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. The Unreal layer still decides nothing. Every number and every rule comes from the core state, the core events or `Tables`.
2. Fish outside the sea (the core gives x from about -68 to 1068 while a fish enters or leaves): never draw a fish outside the water block. Fade or clip it at the side walls.
3. The bucket answers `FishInBucket`, not `FishLanded`. The flight takes the time the core gives.
4. Words on screen: English, from the rules reference where it gives them. Your temporary words (`Bare hook`, `Bait:`, `Catch`, `Fee`, `Day N`, `Upgrades`, `Bait Packs`, `Charms`, `Fish Cards`, `Reroll`, `NEW`) stay for now.
5. The hover card is plain UMG, same style as the rod cards. No new art.
6. The victory screen after day 9 is a plain panel with the words of reference section 14 and one button. It only answers the `Victory` event.

## 4. Work

Block A — changed flows. Go through every line of "Flows that changed" and every new event and state field. For each one: make the picture and the HUD right, keep one answer function per event. At least:
1. Fish that enter and leave at the walls (decision 2). The sea persists across days: no jump at the day end.
2. Landing: a landed fish leaves `state.fish`; the flight uses the landed entry; the bucket answers `FishInBucket`; a Bait Knife Sprat shows as bait gained, not money.
3. Day end: the `Shipping` phase starts with the day-end pulls and flights; `DayEnded` fires when the shipping itself starts; a `ShipItem` can be a batch (count above 1, `catchIndices`); the summary opens 0.8 s after the last item; a skip is remembered.
4. Packs: a bought pack opens at once; a pack can be opened and a bait picked during the day; `StartDay` is refused while a pack is open, and the HUD shows why.
5. Rod: the goal depth is clamped by the core (the handle follows the core value); a bait change in the water applies at once; a pull of an empty rod is valid; Auto-Reel shows its countdown.
6. Hunter: show the approach from the state (the striking fish swims to the hook; HUNTER NEAR on the rod card).
7. Dusk and the day rule: one visible sign each, from the state.
8. `Victory`: decision 6. `SetFeeLevel`: keep it reachable by `catch.NewRun ... Soft|Normal|Hard` only.
9. `catch.Dump` prints the values the core really uses.

Block B — hover card. The cursor on a swimming fish shows a card: name, price at its depth now, the baits that catch it (and "bare hook" if it does). Data: `FishInfoOf`. Pick the nearest mirrored fish within a small screen radius. The card follows the fish, hides when the cursor leaves. It also works during the preparation. Add `catch.Hover <fish id>` so that an agent can show the card without a mouse.

Block C — water rim. Keep water A. Add the bright line along the water line of the front cut, as variant B had it. Two frames, before and after, at the surface camera.

## 5. Done means

1. `python -X utf8 Tools/check_all.py --ue` is green with the editor closed.
2. A `-game` run with `-benchmark -fps=30` plays three days by console commands and by simulated mouse: money in `catch.Dump` equals the HUD after every day.
3. Frames in `Catch/Saved/ue_mcp/`: `int_1_prep.png`, `int_2_day_pack_open.png`, `int_3_hunter.png`, `int_4_day_end_flights.png`, `int_5_ship_batch.png` (use `catch.TimeScale` and a rich seed, or say why a batch did not happen), `int_6_hover.png`, `int_7_rim_before.png`, `int_8_rim_after.png`.
4. No fish is visible outside the water block in any frame.
5. The report lists every changed flow with "done" or the reason.

## 6. Boundaries

- Files you may create or change: `Catch/Source/Catch/**`, `Catch/Content/**`, `Catch/SourceArt/**`, `Docs/briefs/reports/day1-integrate.md`. Not `Catch/Source/CatchCore/**`, not `Catch/CoreTests/**`, not the documents.
- You are the only executor in the editor. Close the editor before you hand back, with everything saved.
- You do not invent the game. No new rule, number or screen beyond this brief. A missing value: write the question, do the rest.
- Do not commit, do not push, do not delete files.

## 7. Report

English, `Docs/briefs/reports/day1-integrate.md`, at most 30 lines: the table of flows; what a player sees new; how you checked; frames; values the picture needed and the core does not give; questions.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Game questions: write them down; the orchestrator asks the owner.
