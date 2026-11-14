# Brief: the view on core 2.2 — 53 modifiers on the shelf and in the shop, the strike chance on screen, the run opens in the shop

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, 03:40 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. State when you start: the orchestrator has merged `track/core` (core 2.2) into the worktree **without committing the merge** (`git status` says "All conflicts fixed but you are still merging"). The view does not compile against the new core yet; making it compile and showing the new things is this brief. Do not commit, do not abort the merge. This brief lives only in the main checkout (`F:\UE\EpicJam\Docs\briefs\day2-view-core22.md`).

## 1. Goal

Adapt the view module `Catch` to core 2.2 and show what the new core brings: 53 modifiers with rarity, live numbers and counters; the strike chance of a larger fish as a number the player sees; the lucky hit; the shop before day 1; the new day rule; a rule switched off by a modifier; coins paid by modifiers at the end of a day. Every new moment answers the player with motion, sound and an effect.

Why it matters. The owner (game director) lost a run on day 15 and found one obvious winning path. His decisions of 13.11 (voice input, Russian, verbatim):

- `drive-and-build-variety`: «Дело в механиках, в том, что как они переумножаются [...] чтобы у нас механики были какие-то такие безумные и при этом ну, позволяли игроку, давали поле для деятельности.»
- `chain-chance-is-a-stat`: «визуально выглядит практически всегда рыба хватается за цепочку. И мы это никак не улучшаем, не ухудшаем. [...] мне хочется, как в рыбалке, чтобы игрок на что-то влиял и понимал бы стратегию, то есть чтобы у него был какой-то азарт.»
- `run-starts-with-shop`: «Забег начинается с магазина. До дня один у игрока есть несколько монет. [...] Давай сделаем попроще.»
- `responsiveness-is-the-main-rule` and `responsiveness-covers-everything`: «Нам главное сделать отзывчивость, чтобы каждое движение давало прикольную [...] полишинг», «Все должно быть приятным.»

## 2. What exists now (verify before you rely on it)

Read first, both in your worktree after the merge:

- `Docs/briefs/reports/day2-core-charms.md`: sections 5 (readings), 6 (interface changes) and 7 (breaks for the view). Its line numbers are from an older view; find the places by name.
- `Docs/briefs/reports/day2-hud-fixes.md`: what the interface executor before you changed (rule card, rule marks, countdown, end-of-day count with the coin flight, `catch.HudClick` / `HudPress` / `HudRelease`, `Ui::PaintFishPicture`).

Read in the main checkout: `F:\UE\EpicJam\Docs\briefs\reports\day2-feedback-audit.md`, section 7 (the answers proposed for the core changes) and section 5.2 (interface helpers that exist).

Facts (paths under `F:\UE\EpicJam_ui\Catch\Source\Catch\`, lines of the state before the merge):

- Core 2.2 queries: `Game::CharmTextOf(kind, shelfSlot)` -> `CharmDetails {kind, id, name, rule, category, rarity, hasLive, live, counter, price, weightlessPrice}` (`rule` has `{n}` filled); `StrikeChanceOf(rod, hook)` (0..1; 0 before `pace.eatFromDay`, for the Gold Sunfish and with no eater in the open zones; 1 when a hunter is coming); `RodNoEscape(rod)`; `RodAutoReel(rod)`; `ActiveRule()`; `State::firstShop`, `State::dayRuleOff`; `ShelfItem::counter`; `Mods::zoneShiftOf` and `countFactor` per fish kind.
- Core 2.2 events: `LuckyChance` (rod, hook, charm, slot, `value`), `MoneyReason::Charm`, `DaySettled.charmCoins`, `CharmFired` for a counter (slot, `oldValue`, `newValue`). The first shop: `PhaseChanged` to Shop, then `ShopOpened` with day 0. `NextDay` from it leads to Prep of day 1.
- Charm enum: 53 kinds; Thrifty Hook is gone, so every kind after Bait Knife moved down by one; `charm::` names of the kept kinds still exist.
- `CatchSession.h:34` exposes `Mods()` only; the new queries have no wrappers. `CatchSession.cpp` logs events in a switch (new event type and money reason are missing there).
- `CatchUiText.cpp:5` `Ui::CharmRuleText(CoreName)` and `CatchUiPanels.cpp:280` `Ui::CharmTag(CoreName)` are maps keyed by the 20 old names. `CharmTag` already falls back to the longest word of the name in capitals. Callers: `CatchHud.cpp:686`, `:1504`, `:1551`, `:3659`.
- `CatchHud.cpp:413` reads `S->Mods().noEscape` (now "any rod"); `:434-441` finds Auto-Reel by a shelf scan; `:3387` reads `Mods().zoneShift` (now always 0; per kind in `zoneShiftOf`); `:2473` calls `ResetForNewRun()` on Prep of day 1 (now after the first shop); `:496` `TodayRule()` returns `St.dayRule`; `:755` the day banner reads `St.dayRule`.
- `CatchTackleView.cpp:476` and `:1099` read `Mods().noEscape`.
- `CatchUiPanels.cpp:954` prints "after day %d" in the shop header.
- `CatchLookSound.cpp` `OnEvent` has no case for `LuckyChance`.
- The rule icon function `PaintRuleIcon` has no case for `DayRule::OnlySmallFish`.
- Bait cards already show rarity by a border colour (`RarityBorder`) and a glint.
- Scripted runs: `catch.NewRun 42` used to land in the preparation of day 1. It now lands in the shop before day 1; `catch.NextDay` leads to the preparation. The two proof scripts in the orchestrator's scratchpad (section 6) still assume the old start.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

Block A, compile and behave as before:

1. Session wrappers for the new queries; log lines for the new event type and money reason.
2. Charm name, class, rarity and rule text come from `CharmTextOf(kind, slot)` everywhere (shop card, shelf chip tooltip, sell view, popups). Remove the old rule map. The chip word: keep `CharmTag` with its fallback; drop the dead "Thrifty Hook" row; you may add tags for new names where the fallback reads badly.
3. Per rod: the escape ring, the escape warning and the rod card use `RodNoEscape(rod)`; the Auto-Reel line of the rod card uses `RodAutoReel(rod)`. Per kind: the zone of a fish in the info card uses `zoneShiftOf[kind]`.
4. `TodayRule()` and every rule mark use `ActiveRule()`. When `dayRuleOff` is true on a rule day, the rule box and the rule card show the rule text struck through with the line "RULE OFF" and the name of the modifier that did it (Storm Glass); no rule marks are drawn that day.
5. The first shop. `ResetForNewRun()` moves to the start of the run (the `PhaseChanged` to Shop with `firstShop`, or `ShopOpened` with day 0). The shop header reads "before day 1" there instead of "after day 0". The NEXT DAY button reads "TO DAY 1" there. The free starting pack keeps its present behaviour (it opens by itself where the game is waiting for it; say in the report where it opens now). The day 1 banner and tips still show in the preparation of day 1. `catch.HudAutoPack` keeps working for scripted runs.
6. New reference numbers. Make your own copies of the two proof scripts with `catch.NextDay` after `catch.NewRun 42`, run each twice, and report the numbers of the plain day and of the flow run (events, the `DaySettled` line, coins, errors). They must be equal between the two runs. The orchestrator adopts them.

Block B, the new things on screen (existing material only: helpers, sounds and colours that are already in the view):

7. Rarity. The charm card border takes the rarity colour and the glint that bait cards use; the tooltip of a shelf chip names the rarity in one word ("Common", "Uncommon", "Rare").
8. Live number and counter. The rule text already carries the live number. A charm with a counter shows the counter as a small number on its shelf chip. When `CharmFired` reports a counter change, the number pops (scale 1.4 to 1) and `Note` plays a semitone higher per step (at most 12 steps); a reset drops it to its start with `Bloop` at pitch 1.3.
9. Dynamic shelf slots (Rod Stand, Sea Chest, Bigger Shelf). The slot row follows the core's slot count; when it changes the slots slide instead of jumping. A charm that stands beyond the slot count stands on a red dashed slot. A refused purchase answers as refusals do now (card shake, `Deny`).
10. Strike chance. For every hook with a fish and `StrikeChanceOf > 0`: a thin red arc just outside the gold hang ring, its length = the chance. The rod card's third line reads "STRIKE 40%" (when Auto-Reel is on, both facts must stay readable). The fish hover card of a hooked fish adds the line "Strike chance: 40%". The numbers come from the query every frame; round to whole percent.
11. The lucky hit (`LuckyChance`). The biggest beat of a day: `Chord` plus `Link` at pitch x 2; camera shake 5 px (`ACatchSeaView::AddShake`); gold dots at the hook; a large gold popup at the hook: "LUCKY x3" (the multiplier from `value`); the chip of the charm that won wobbles; a chip whose counter grew at the same moment pops as in decision 8.
12. A modifier row fires at its place (audit G10). Today the only answer is the chip wobble at the bottom. Add one generic answer: a small popup with the chip word at the place the row acted on (the bucket for a landed fish, the hook for a chain link or an escape, the rod tip for a pull, the chip itself for shop and day-end rows), in the class colour, at most one popup per charm per 0.4 s. Use what the events carry now; where an event carries no place, keep the chip wobble only and list the event in the report.
13. Coins from modifiers. `MoneyChanged` with `MoneyReason::Charm` during the day: coins fly one by one from the chip to the coin counter (the existing coin flight). In the summary: one row "Modifiers" with `charmCoins`, counted up like the other rows, shown only when it is not 0; these coins join the coin flight of the summary.
14. `OnlySmallFish`. An icon in `PaintRuleIcon` (a large fish shape, crossed). The rule card and rule box show the core text. A hooked or hovered fish of 10 kg or more shows "now 0 pts" as the mirror rule does for light fish.
15. `DayRuleAnnounced`: when a rule day first enters the forecast, the forecast plate drops in with the existing back-out ease and `Chord` at volume 0.5.

Text. All new player-facing text is English and written as plain literals like the code around it (a later brief wraps every literal into the text layer). The new texts are exactly: "before day 1", "TO DAY 1", "RULE OFF", "Common", "Uncommon", "Rare", "STRIKE %d%%", "Strike chance: %d%%", "LUCKY x%d", "Modifiers", "now 0 pts" (exists). If you need another text, keep it short, list it in the report.

Not in this brief: the text layer and Russian; the fish and sea reactions (another executor); greying heavy fish in the sea on an `OnlySmallFish` day; icons; any change in the core.

## 4. File rules (another executor edits `F:\UE\EpicJam_look` at the same time; that branch is merged into this one later)

- Free to change: `CatchHud.*`, `CatchUi*.*`, `CatchSession.*`, `CatchPlayerController.*`, `CatchLookSound.*`.
- `CatchTackleView.*`: small local changes only: the two `noEscape` reads, one new handler for the lucky hit with its dispatch line, the strike arc next to the hang ring in `DrawRods`. Do not move or reformat code there. The other executor adds calls inside the bodies of `OnFishHooked`, `OnFishLanded`, `OnHunterStrikes`, `OnChainLink`, `OnPullStarted`, `OnEscapeSoon`, `OnBaitUsed`, `OnHookWentBare`, `StepBucket`, `BucketHit`: stay out of those bodies.
- Do not touch: `CatchLookFish.*`, `CatchLookSea.*`, `CatchSeaView.*` (call its existing public functions only), `CatchLookScales.*`, `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Content/**`, `Catch/SourceArt/**`, `Tools/**`, `CLAUDE.md`, `AGENTS.md`, `Docs/**` except your report.

## 5. Work order and stops

Block A first, to a green build and a clean scripted day (decision 6). Then block B in this order: 10, 11, 7, 8, 13, 14, 4 (frames), 9, 12, 15. If you run short of session, stop after a finished decision with a green build and report exactly what is left.

## 6. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui`: `10 ok, 0 fail, 1 skip`.
2. The plain day and the flow run, each run twice with equal numbers and 0 errors (decision 6).
3. A scripted mouse run through the first shop: a purchase by `catch.HudClick`, the starting pack, "TO DAY 1", the preparation of day 1, START DAY.
4. Log proof: a strike chance value for a hooked fish on day 2 or later; a `LuckyChance` event with its popup (use a run where Lucky Bite is on the shelf; if the view has no command to put a charm on the shelf, add a debug command `catch.GiveCharm <id>` in `CatchSession` that is logged as a cheat, and say so); a counter change on a chip; a `MoneyReason::Charm` coin flight.
5. Frames from a running game, looked at by you, at 1920x1080: the first shop; a shop with an uncommon and a rare charm card; the shelf with five charms, one with a counter; a hooked fish with the strike arc and the rod card line; the lucky hit; the rule card and rule box of `OnlySmallFish`; a rule switched off by Storm Glass; the summary with the "Modifiers" row. And at 1280x720: the first shop, the shelf, the rod card with the strike line.
6. No text clipped or overlapping in those frames. What worked before still works: card tilt and hover, pack pick only after the cards landed, click on a hooked fish pulls, the countdown, the end-of-day count, the sound bed.
7. `git status` shows your changes only in the files of section 4 (plus the files the merge brought).

## 7. Verification commands (from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_check.sh" /f/UE/EpicJam_ui v22 "catch.HudAutoPack 0,"
bash "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/run_feedback.sh" /f/UE/EpicJam_ui v22f
```

Read the two scripts, then copy them into `F:\UE\EpicJam_ui\Catch\Saved\` under new names and adapt the copies (decision 6); do not edit the originals. Frames land in `Catch/Saved/ue_mcp/`.

Build notes. Another executor builds the Unreal project in `F:\UE\EpicJam_look`; `Tools/build-editor.ps1` passes `-WaitMutex`, so a build of yours may wait for theirs. Wait; never kill a build or an Unreal process you did not start; stop your own game runs by their own process id. A game run binds port 8000; when two game runs overlap, the later one logs two "HttpListener unable to bind" errors: rerun, it is not your bug. Scripted runs use `-benchmark`, which turns the audio device off. Do not open the editor.

## 8. Boundaries

- No rule, number, screen or text beyond this brief. Nothing in the core.
- No new assets, sounds, fonts, downloads or generators. Nothing from the browser prototype.
- Do not delete or hide existing elements. Keep what works.
- No commits, no pushes, no packaging, no `git merge --abort`, no `git reset`, no `git stash`.

## 9. Report

Write `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-core22.md` in English and hand back a short summary: per decision done / not done with file and function; the new reference numbers; the texts you added; events that carry no place (decision 12); the check results with log lines; the list of frames with one line each; every decision you changed; every line touched in the other executor's places; questions.

## 10. Chain of command

You report to the orchestrator, not to the owner. Answer technical questions yourself and list them as revocable readings. A question about what the player should see beyond this brief goes into the report, and you continue with the parts that do not depend on it.
