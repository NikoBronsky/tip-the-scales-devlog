# Brief: view, part 4 - the free charm after each day, rule-day marks, the texts of core 2.4

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 16:30 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message (the fish pictures and core 2.4 are merged in it). This brief lives only in the main checkout.

## 1. Goal

After every passed day the player takes one charm of three for free, or skips. The view shows this choice in a window, lets the player look at the shop before choosing, and puts the taken charm on the shelf. The view also shows which fish give nothing on a rule day, and carries the new texts and numbers of core 2.4.

Why it matters. The owner (game director):

- Decision `fast-growth-approved`, 13.11: «В принципе, запускай следующий блок.» The block, in the orchestrator's words that he approved: after each passed day the player takes one charm of three for free; the reward and the shop offer only charms that can already work for the player; targets rise more steeply; the rule day "10 kg or more gives no points" reads the fish kind.
- Decision `tons-of-fish-means-big-points`, 13.11: «Когда я говорю тонны рыбы я не имею в виду буквально, я скорее про большое количество очков». The player must see the points grow fast; big numbers must read well.
- The frame of the game: the upgrades that change the rules are the heart; every action answers with motion, sound and an effect.
- He judges every frame against a browser prototype he liked; a placeholder look must be labelled as such in the report.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\`.

- The core interface of this block: `Docs/briefs/reports/day2-core24.md`, section "For the view" (commands `TakeReward(index)`, `SkipReward()`; queries `RewardOffers()`, `DayRuleScores(kind, finalWeightTenths)`; events `RewardOffered`, `RewardTaken`; `State::rewardPending`; `StartDay` returns Locked while a reward is pending; `TakeReward` returns NoRoom on a full shelf for a charm that needs a slot; the start gift Deeper Line now carries bait; ApiVersion 5).
- The pick-one-of-N window of part 2: the start gift (`PICK YOUR START`, `One is free.`, `Click a card to take it.`), files `CatchHudFlow.cpp`, `CatchUiFlow.cpp/.h`; the cheat `catch.Gift <i>`; the scheduler `catch.On <Event>@<delay> <cmd>`. Report: `Docs/briefs/reports/day2-view-core23.md`.
- The day-end flow: the summary counts in order, the total flies to the coin counter, then the shop opens (`catch.HudToShop`).
- The shop: charm cards, the shelf, hover cards (`CatchHudShopTip.cpp`, `Ui::PaintShopTip`), the hint `Click a charm on the shelf to sell it.`, the `SELL +N` popover, the `FISH TABLE` button.
- The sea hover card of a fish and the fish table (`Ui::PaintFishPicture` draws the fish picture since part 3; report `Docs/briefs/reports/day2-view-fish-art.md`).
- The text layer and the Russian testing table: `Catch/SourceArt/Text/gen_text_ru.py` (`--check`), `run_text_proof.py` with the modes plain, flow, fail, big, depth, screens, title. The game's word for a modifier is "charm".
- Big numbers: thin-space thousands up to 9 999 999, `catch.HudBigNumbers`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. When the window opens. `RewardOffered` arrives after `DaySettled` and before `ShopOpened`. The view first plays the whole day summary. Then the shop opens with the reward window on top of it. A failed day has no reward and no window.
2. The window reuses the pick window of the start gift: three cards (or fewer if the core gives fewer), each with the charm picture or class colour as the shop shows it, name, live rule text, rarity, and the tag `FREE`. Title `PICK A CHARM`, line `One is free.` The weightless variant carries the same mark as in the shop. The hover card of the shop works on these cards.
3. Two buttons under the cards: `SKIP` (calls `SkipReward`, after one confirming second click or a clear two-step press so a stray click does not lose the reward) and `LOOK AT THE SHOP` (hides the window, the reward stays pending).
4. While a reward is pending and the window is hidden, the shop shows a pulsing button `FREE CHARM` near the shelf or the charm row. It reopens the window. `NEXT DAY` with a pending reward does not leave the shop: it reopens the window with the line `Take one or skip.` So the choice is always made in the shop, where selling works.
5. Full shelf. A card whose charm needs a slot shows `SHELF FULL` and a click on it does not take it; the window shows the line `Sell a charm on the shelf to make room, or skip.` After a sale the cards become clickable again. A weightless charm can be taken with a full shelf.
6. Taking a card: the card flies to its shelf slot, the shelf slot answers (the same motion and sound family as a purchase), the other cards leave, the window closes. `RewardTaken` drives it. Read the day length and the hooks again after `RewardTaken`.
7. The fit group of an offer (`Offer::fitGroup`) is a diagnostic. Do not show it to the player.
8. Rule-day marks. When `DayRuleScores(kind)` is false for today's rule, the player must see it before fishing and while fishing: the sea hover card of such a fish and its row in the fish table carry a mark `0 pts today`; a fish of such a kind on the hook shows `0 pts` on the rod card as the core reports it. Use the query, do not copy the rule into the view.
9. The start gift Deeper Line: the new card text comes from the core; show the bait icon with the portion count on the card (`GiftData::bait`, `baitPortions`: 15 Squid Strip today; the `GiftTaken` event carries the bait kind). Read the count from the core, do not copy the number.
10. Texts. Every changed charm text comes from the core. Regenerate the Russian testing table; add Russian rows for every new view text. English stays the default; no Cyrillic under `Catch/`.
11. Numbers. Targets now reach tens of thousands by day 15 and hundreds of thousands later. Check the target, the scales, the summary and the shop line `Target tomorrow` with such numbers.
12. Cheats and proofs. Add `catch.Reward <i>` (take the card i) and `catch.Reward skip`. Every proof mode and cheat run that goes past a day end must take or skip the reward, or the next `StartDay` is Locked. Add a proof mode `reward`: a day, the summary, the window, hover, look at the shop, reopen, take; a second run with a full shelf (the mark, a sale, the take); a run that skips.

## 4. Work by blocks (in this order; extend the report after each block)

1. The reward window after the summary: cards, take, skip, the flight to the shelf.
2. Look at the shop, the `FREE CHARM` button, `NEXT DAY` with a pending reward, the full shelf.
3. Rule-day marks; the Deeper Line gift card with bait.
4. Texts, Russian rows, big numbers, the proof modes and cheats.
5. Proof frames and checks.

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- The proofs pass with 0 errors and 0 missing keys: `run_text_proof.py plain en 1920`, `flow en 1920`, `reward en 1920`, `screens ru 1280`.
- A scripted run reaches day 4 through three rewards (take, take, skip) with no Locked `StartDay`.
- Frames you looked at yourself, 1920x1080: the window with three cards; a hover card on a reward card; the hidden window with the `FREE CHARM` button in the shop; the full-shelf state; a card in flight to the shelf; the shelf after the take; the mark `0 pts today` on the sea hover card and in the fish table on a rule day; the Deeper Line gift card with bait; the day 4 target on the scales. The window and the full-shelf state in Russian. One frame at 1280x720.
- The report lists every new player text in English and in Russian.

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py reward en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`. Do not kill a process you did not start. If the gate reports a time-out of the core tests while the machine is busy, run `Tools\core-test.ps1` alone and report both results.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, and your report file. Use absolute paths.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, `Catch/Source/Catch/Catch.Build.cs`, `Catch/Config/**`, the level, the assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game. If the core lacks something the view needs, write the question into the report and build the rest.
- No downloads, no generated pictures, no fonts, no new sounds: reuse the sounds the game has.
- Do not delete or hide old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/`. No director, cinema or "cut" wordplay.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-reward.md`, in English. Create it right after reading, with one status line per block, and extend it after each block. Sections: what changed by block, new texts (English and Russian), cheats and proof modes, frames with paths, checks with their output lines, not checked, placeholders that remain, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
