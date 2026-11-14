# Brief: day one — progression: why the start feels like the late game, and measured variants

Status: written and handed out on 12.11.2026 at 10:50 (owner's clock). Executor: Opus, effort high (the executor of `day1-core-align.md` continues). Work folder: the worktree `F:\UE\EpicJam_core`.

## 1. Goal

The owner played the prototype again and named the main problem of the game. His words:

- «У нас игра супер-пупер крутая получается уже сейчас. Основная проблема в балансе, в открытии технологий. Пока что мы сразу практически чувствуем себя так, как будто бы мы уже на поздних стадиях игры.»
- «в Баллатру ты сначала по одной карточке сначала тебе сложно, потом, если у тебя получается собрать свою комбинацию, то ты уже наслаждаешься вот этим вот огромным количеством денег, рыбы и так далее. А если нет, то нет.»
- «у нас сейчас прям сразу ты заходишь и сразу можешь легко схода делать комбинации. У нас задача не стоит сделать игру суперсложной, но надо, чтобы она хоть какая-то прогрессия чувствовалась.»

Find out with numbers why the start feels like the late game. Then give the owner two or three pacing variants, each measured, so that he can choose. The orchestrator turns your document into the owner's decision page.

## 2. What exists now (verified by the orchestrator on 12.11)

- The worktree is clean at `63e4cb8` (branch `track/core`): the aligned core plus the movement rates of reference section 18. Core tests: `Tools/core-test.ps1`. Runner: `Tools/core-test.ps1 -RunBalance -Runs N -Days D`, three policies, three fee levels.
- Your own measurement, person-like policy, Normal, 20 runs (day / median money before the fee / fee): 1 136 15 | 2 910 30 | 3 2312 70 | 4 2860 150 | 5 3092 320 | 6 4409 700 | 7 5067 1500 | 8 5633 3200 | 9 7459 7000. The money is 30 times the fee on day 2. You named the cause: an early Deeper Line; deep fish pay far more and bare hooks catch them.
- Seen in the game on day 1 with one rod and a Worm: a Sea Bass ate the hooked Mackerel, "CHAIN x1", $23. The chain is there in the first minute.
- Rules in words: `Docs/design/rules-reference.md` (1.6 fee, 9 baits and packs, 10 shop, 11 charms, 12 unlock order, 13 special fish). The first design list: `Docs/design/content-list.md`.
- Owner's recorded decisions that bind this work (`Docs/decisions/game-frame.answers.json`, short form in `CLAUDE.md` section 2): the fee presses every day and the fee table is data; the game must not be easy; content opens in order and gradually (bait, upgrades, charms); the ocean fills gradually; depth opens as the game goes, and the player can also win at the surface because charms lift the income of surface fish; growth as in an incremental game (sharpen one rod, catch size, fish count, rod count, hook count); the heart of the game is upgrades that change the rules, and a cheap fish can bring more than an expensive one; every third day a special fish comes; the run ends when the fee cannot be paid; the game has no end.
- One open question of the owner is still open: is content unlocked inside one run (the current reading) or also between runs.
- New owner decision of 12.11: close the Thrifty Hook loophole (a free portion must not return to the stock on a bait change).

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. Levers are data first: the fee table, prices, unlock days, fish counts by day, offers per shop, start stock, pack contents and odds. A lever that needs a new core rule may enter a variant only if it is small (for example "a species swims from day N", "charm offers start on day N", "hunters strike from day N", "a bare hook does not catch below zone N"). Mark every such lever "needs a core change" with its size in lines.
2. No new mechanic, no new item, no renamed item, no change of what an item does.
3. Experiments live in the runner as named table variants (in `Catch/CoreTests/`). `MakeDefaultTables()` stays equal to the reference, except the Thrifty flag. The owner has not chosen yet.
4. The target shape to test against. It is my reading of the owner's words; the owner confirms it:
   - Days 1-2: one rod, one bait, single fish. After the fee the money buys one thing in the shop, not three. No chain happens unless the player chose something for it.
   - Days 3-5: the build forms: the first charm, a second rod or the next depth, the first chains.
   - Day 6 and later: a build that works earns far above the fee; a build that does not work fails between day 7 and day 9.
   - Not very hard: the person-like policy passes day 5 in at least 80% of the runs on Normal.
5. If the person-like policy plays unlike a person in a way that bends the numbers, fix the policy and say what you fixed. You may add one policy that hunts for combos.

## 4. Work

0. Thrifty Hook: set `Tables::edges.thriftyPortionReturns` off by default; update the test; one line in core-interface section 10.
1. Diagnosis on the current tables. For the person-like policy, days 1-9, Normal: income by source (zone, species, share of chains, share of charms), purchases, money before the fee against the fee, and the day of the first chain, first charm, first Deeper Line, second rod. Name the three to five causes of "late game at once", each with its number.
2. Two or three named pacing variants. Each is a short list of lever values. Run each one: the same tables, the alive share by day, the first-something days. Show Soft, Normal and Hard only where it matters.
3. Your recommendation with the reason. For each variant, say in game language what the player meets on day 1, day 3, day 5 and day 8.
4. The open question about unlocking between runs: say what it would add to each variant and what it costs, in one paragraph.

## 5. Done means

1. `Docs/design/progression-proposal.md`, at most 150 lines, with the tables of work items 1-3.
2. The runner prints every variant by name.
3. `Tools/core-test.ps1` passes. The game target builds from the worktree. `python -X utf8 Tools/check_all.py` is green in the worktree.
4. The default tables changed only in the Thrifty flag.

## 6. Boundaries

- Files you may create or change, all inside `F:\UE\EpicJam_core`: `Catch/CoreTests/**`, `Tools/core-test.ps1`, `Catch/Source/CatchCore/**` (only for work item 0), `Docs/design/core-interface.md` (section 10, one line), `Docs/design/progression-proposal.md`, `Docs/briefs/reports/day1-progression.md`.
- Never write into `F:\UE\EpicJam`. Another executor builds the game there.
- You do not decide the game. You measure and propose; the owner chooses.
- Do not commit, do not push, do not delete files, do not start the Unreal Editor.

## 7. Report

English, `Docs/briefs/reports/day1-progression.md`, at most 25 lines: the causes, the variants in one table, your recommendation, policy fixes, levers that need a core change, questions for the owner.

## 8. Chain of command

You report to the orchestrator, not to the owner. Technical questions: decide, note, continue. Game questions: write them down; the orchestrator asks the owner.

## 9. Round 2 (added by the orchestrator on 12.11.2026 at 10:52)

Round 1 is verified and committed on `track/core`. The diagnosis is strong. The recommended variant does not yet give what the owner asked for. Critique of the orchestrator:

1. The fee still does not press. In steady-build the money is 7 times the fee on days 1-2 and 12-19 times on days 3-5. The owner decided earlier: the fee presses every day. He now says: at first it is hard, one card at a time.
2. The jump only moved. After day 3 the player buys 9.6 things at once, because Twilight multiplies the income by 4.5 and nothing else is needed for it.
3. Depth needs no bait. A bare hook catches deep fish at full price. So bait packs, which are the cards of this game, do not matter. The owner decided: the bait says which fish it catches, and a bare hook catches little.
4. A hidden rule "hunters do not strike before day 3" is weaker than a visible one. The owner decided that the ocean fills gradually and that content opens in order. A hunter species that arrives in the sea on day N is a moment the player sees.

Work of round 2:

A. Add the three gates to the core as data, with defaults that keep the behaviour of today (so the default game does not change): the first day a species swims (per species), the deepest zone in which a bare hook catches, and nothing else. Tests for each gate with a non-default value. A pack must not offer a bait whose fish cannot be met yet; if the core has no such rule, say so and measure without it.
B. New target shape, measured with the person-like policy on Normal:
   - After the fee on day 1 and on day 2 the money buys about one thing (1 to 1.5 purchases). On days 3-4 about two.
   - Money before the fee divided by the fee is between 1.5 and 3 on days 1-5. From day 6 a build that works reaches 5 to 15.
   - The firsts spread out: second rod about day 2, Deeper Line about day 3, the first chain when the first hunter species arrives (day 3), the first charm on day 3 or 4.
   - Alive on day 5: at least 80%. On day 9: between 40% and 70%. The policy buys-cheapest-useful should fail between day 5 and day 7.
C. Variants to build and measure (same tables as round 1):
   - `fee-led`: steady-build unlock days and prices, hunter species from day 3, and a fee table tuned to target B.
   - `bait-gated`: as steady-build, hunter species from day 3, a bare hook catches only in Sunlight, deeper fish need their bait, pack prices and the first packs tuned so that a player can follow.
   - `both`: the two together, tuned to target B.
   You may tune prices, unlock days, the fee table and the pack offer inside a variant. Keep every item and its effect.
D. Say which variant meets target B best, what the player meets on days 1, 3, 5, 8, and what the bot cannot tell us (where a person plays differently).

Outputs: add section 7 "Round 2" to `Docs/design/progression-proposal.md` (at most 70 lines) and a "Round 2" part to the report (at most 12 lines). The same boundaries. `Catch/Source/CatchCore/**` is open for work item A only. Do not commit.
