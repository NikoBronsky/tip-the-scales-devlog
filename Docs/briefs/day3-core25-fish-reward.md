# Brief: core 2.5 - the free charm is a prize for the Gold Sunfish; the Sunfish takes only a bare hook

Executor: Opus, effort high. Written by the orchestrator on 14.11.2026 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_core`, branch `track/core`, base `1abeb14` (core 2.4). This brief lives only in the main checkout `F:\UE\EpicJam`.

## 1. Goal

Today the player gets a free charm (a choice of three) after every passed day. The owner finds it too easy. New rule: the free charm is a prize for a condition. The first condition is landing the Gold Sunfish, and the Sunfish takes only a hook with no bait.

Owner (game director), 14.11.2026, playtest of build 10, voice input:

- Decision `free-charm-for-special-fish`: «По поводу бесплатных модификаторов их можно, но надо за какие-то условия. Я думаю, вот за рыбу это будет ок.» Earlier the same night: «как вариант давать бесплатный модификатор за рыбу».
- Decision `sunfish-bare-hook-only`: «Просто рыба луна, пускай рыба луна клюет только на пустой крючок.»
- Decision `start-gift-instead-of-free-charm`: at the run start the player keeps the existing gift choice (new rod, sharper rod, longer line) and gets no free charm there: «вместо бесплатных модификаторов давать эти штуки».
- Earlier decision `daily-free-charm-feels-too-easy`, 13.11: «мне дают его каждый день по новому модификатору. Как будто это слишком легко.»

Other special fish with other conditions are an idea of the owner, not a decision. Do not add them. Build the prize so that a second prize fish can be added later as data.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_core\Catch\Source\CatchCore\`.

- The reward: `Private/Shop.cpp:117` (`if (!tables_.shop.dailyReward) return;`, then the `RewardOffered` event at :121), called from the day end `Private/Days.cpp:265` (`RefreshShop(true, tables_.shop.dailyReward)`). Flag `dailyReward` and `rewardChoices`: `Public/CatchCore/Tables.h:337-338`. Events `RewardOffered`, `RewardTaken`: `Public/CatchCore/Game.h:30-31`.
- The Gold Sunfish: enters on every third day (`Private/Days.cpp:323`, `EnterGold` in `Private/Fish.cpp:108`); points = half of the day target (`Private/Prices.cpp:130`, `Private/Tables.cpp:144`), 3 coins; nothing eats it.
- The bare hook: it catches only shoal fish of the Sunlight zone (`Private/Rods.cpp:152-153`), four times slower (`Private/Rods.cpp:184-185`, `bareDelayMultiplier`).
- The targets were fitted in core 2.4 with a free charm after every day. Report with the goals and the bot tables: `F:\UE\EpicJam_core\Docs\briefs\reports\day2-core24.md` (or the same file in `F:\UE\EpicJam_ui`). Balance program: `Tools/core-test.ps1 -RunBalance -Runs N -Days D`.
- The description of the core: `F:\UE\EpicJam\Docs\design\core-interface.md` (read only; the orchestrator updates it from your report).

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. The reward mode is data: none, after every day (the old rule, kept for tests and for a quick switch back), or for a prize fish. Default: for a prize fish. Do not delete the old path or its tests.
2. In the prize mode the reward is offered at the end of a passed day only if the player landed the Gold Sunfish on that day. Same three choices, same fit filter, same take and skip, same place in the day-end order as today, so the view shows it without a change. A day without the prize offers nothing.
3. The existing prize of the Sunfish (half of the target in points and 3 coins) stays. The free charm is added to it.
4. The Gold Sunfish takes only a bare hook: a hook with no bait. No bait draws it, also not a bait that takes any fish. On a bare hook it bites with its own normal delay; the four-times slow-down of bare hooks does not apply to it. Shoal fish keep the old bare hook rule.
5. Give the view the facts it needs as data, not as text guessed in the view: a fish kind says that it takes only a bare hook, and the state says whether the prize of the day was earned. Name the new fields and events in the report (section "Interface changes").
6. Read every charm that names the bare hook or the Sunfish (Bare Steel, Storm Glass and any other). Keep their rules true. If a charm text becomes wrong or a rule clashes, do not invent a fix: report it as a question.
7. Bots. The bots that play well keep one hook bare on a Sunfish day until the Sunfish lands. Keep one bot that ignores the condition (the unaware player). Report the share of Sunfish days on which each bot lands the Sunfish.
8. Refit of the targets (data only). Run the balance program in the new mode and compare with the goals of the 2.4 report. Refit the target table so that the pass rates return to the same goals. These numbers are the orchestrator's technical balance; show the table before and after. Do not change prices, charm numbers or fish numbers.

## 4. Work by blocks

1. The reward mode and the prize condition, with tests. 2. The bare hook rule of the Sunfish, with tests. 3. The data for the view. 4. The charm review (decision 6). 5. Bots. 6. The refit and the tables. 7. The report.

## 5. Done means

- `powershell -NoProfile -ExecutionPolicy Bypass -File Tools\core-test.ps1` passes, with new tests: no reward on a day without the Sunfish; a reward after a day with the Sunfish landed; no reward when the Sunfish swam but was not landed; the Sunfish ignores every baited hook; the Sunfish takes a bare hook; the old daily mode still passes its tests; determinism and capacity as before.
- `python -X utf8 Tools/check_all.py --ue` from `F:\UE\EpicJam_core` prints `10 ok, 0 fail, 1 skip` (the view module must still compile against the core).
- The report holds: interface changes; the bot tables before and after the refit (pass rate by day, Sunfish landed share, charms on the shelf by day); the target table before and after; the charm review; open questions.

## 6. Verification commands (run from `F:\UE\EpicJam_core`)

```
powershell -NoProfile -ExecutionPolicy Bypass -File Tools\core-test.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File Tools\core-test.ps1 -RunBalance -Runs 300 -Days 12
python -X utf8 Tools/check_all.py --ue
```

Builds use `-WaitMutex`: another executor builds the view worktree at the same time, so a build may wait. If the gate reports a time-out of the core tests while the machine is busy, run `Tools\core-test.ps1` alone and report both results. Do not kill a process you did not start.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_core`: `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, design data files that the core build reads, and your report file. If the view module stops compiling because of a renamed field, make the smallest fix in `Catch/Source/Catch/**` and list it.
- Do not change the view otherwise, `Catch/Config/**`, `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, `Docs/design/core-interface.md`, the check gate, permission settings.
- Do not invent rules of the game beyond this brief. A question goes into the report.
- No deleting or hiding of old work. No commits, no pushes. All player text is English.

## 8. Report

File: `F:\UE\EpicJam_core\Docs\briefs\reports\day3-core25-fish-reward.md`, in English. Create it right after reading, extend it after each block. Final message to the orchestrator: at most 25 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; do not wait for an answer if other work remains.
