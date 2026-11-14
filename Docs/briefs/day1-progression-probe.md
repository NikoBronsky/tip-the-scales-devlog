# Brief: day one — a progression probe in the browser prototype

Status: written and handed out on 12.11.2026 at 11:15 (owner's clock). Executor: Opus, effort high. No Unreal Editor and no game code in this task.

## 1. Goal

The owner named the main problem of the game. His words:

- «Основная проблема в балансе, в открытии технологий. Пока что мы сразу практически чувствуем себя так, как будто бы мы уже на поздних стадиях игры.»
- «в Баллатру ты сначала по одной карточке сначала тебе сложно, потом, если у тебя получается собрать свою комбинацию, то ты уже наслаждаешься вот этим вот огромным количеством денег, рыбы и так далее. А если нет, то нет.»

Two rounds of bot runs gave a pacing variant named `both` (`Docs/design/progression-proposal.md`, section 7). A bot cannot tell how it feels. The owner judges by play, and the thing he plays today is the browser prototype. Make a copy of the prototype that plays by the variant `both`, so that the owner can play days 1 to 9 and say whether he feels the progression.

The prototype is a decision aid. Nothing from it goes into the Unreal game, and nothing from this task goes into the repository except your report.

## 2. What exists now (verified by the orchestrator)

- The prototype, version 4: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\catch-prototype.html`. One HTML file, a canvas game, 2,941 lines, all text in Russian. The other files in that folder are older versions. Leave them alone.
- Where its data sits (line numbers are approximate):
  - the fish table near line 145 (`zone`, `price`, `shoal`, `hunter`, `count`);
  - the charm table near line 200;
  - `NEWROD_PRICES`, `SHARPEN_PRICES`, `DEEPER_PRICES`, `SHELF_PRICES` near line 221;
  - the fee levels near line 224: `normal: { fees: [15, 30, 70, 150, 320, 700, 1500, 3200, 7000], mul: 2.15, deeper: [25, 150, 700, 3000, 12000] }`;
  - `PACKS` near line 234 (the small pack costs 6);
  - `SET.level` is "normal" near line 312;
  - the bare hook rule near line 494 (`if (!b) return !!f.T.shoal`);
  - the shop offers near line 1065 (`rollOffers()`), the unlock order of the offers elsewhere;
  - the hover text "Клюёт и на пустой крючок." near line 2399.
- The rules of the prototype in words: `Docs/design/rules-reference.md` (section 9 baits and packs, 10 shop, 12 unlock order). Use it to find your way.
- The variant `both`, exact values (source: `Catch/CoreTests/core_runs.cpp`, functions `SteadyBuild`, `BaitGated`, `Both`):

| Lever | Now (Normal) | Variant `both` |
| --- | --- | --- |
| Hunter species (every fish with `hunter`) | in the sea from day 1 | not in the sea on days 1 and 2; they swim in from the edges on day 3 |
| Bare hook | catches every shoal fish in every zone | catches a shoal fish only when the zone of the fish is Sunlight (zone 0) |
| Fee, days 1-9 | 15, 30, 70, 150, 320, 700, 1500, 3200, 7000 | 20, 30, 60, 100, 300, 600, 1000, 2200, 5000; growth after day 9 unchanged |
| Deeper Line prices | 25, 150, 700, 3000, 12000 | 60, 300, 1500, 6000, 20000 |
| Deeper Line in the shop | from the first shop | from the shop of day 2 |
| Charms in the shop | from their own unlock days | no charm before the shop of day 3; later unlock days stay |
| New Rod prices | 20, 45, 100, 220 | 40, 100, 240, 520 |
| Small pack price | 6 | 4 |
| Pack offer | a bait whose fish has an open zone | also: one of those fish swims today or tomorrow (the shop sells for the next day) |

- How the core reads the two gates, so that you read them the same way. A species whose first day has not come has a wanted count of zero; on its first day the sea fills with it from the edges, like any refill. The zone of a fish for the bare hook gate is its effective zone, after any rule of the day or charm that moves the fish.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. The output is a new file in the same folder: `catch-progression.html`. The original file stays the same, byte for byte.
2. Only the Normal level changes. Soft and Hard keep their tables. If the prototype has a level switch, leave it.
3. "Shop of day N" means what the prototype already means by its unlock days. Do not invent a new meaning.
4. One new piece of feedback, marked NEW in your report: when the hunter species first swim in, show a notice in the way the prototype already shows its new-bait notice. Text: «В море пришли хищники». Nothing else new on screen.
5. A small label that never hides play, in a corner: «Проба прогрессии: плата и наживка». The owner must know which version he plays.
6. Texts must stay true. The hover text "Клюёт и на пустой крючок." shows only for a fish that a bare hook can catch under the gate. If a bait card or a shop text names something that the variant changes, correct it.
7. Do not change the look, the sound, the timing or any other rule. Do not tune anything. The variant is measured as it is.
8. All text for the player stays Russian.

## 4. Work

1. Copy the file. Apply the table of section 2.
2. Check by reading and, if you can do it in ten minutes, by running the script in Node with stubs for the canvas: days 1 and 2 have no hunter fish in the sea; day 3 has them; with two open zones a bare hook gets no bite from a Twilight fish; the fee and the prices match the table; the packs of days 1 and 2 follow the pack rule.
3. Syntax check of the script of the new page (`node --check` on the extracted script, or the same by another way).

## 5. Done means

1. `catch-progression.html` exists. The hash of `catch-prototype.html` is the same before and after.
2. The script parses.
3. The report holds a table: lever, old value, new value, line in the new file.
4. The report says which checks you ran and which you only read.

## 6. Boundaries

- Files you may create or change: `catch-progression.html` in the prototype folder, scratch files in the same scratchpad, and `F:\UE\EpicJam\Docs\briefs\reports\day1-progression-probe.md`. Nothing else in `F:\UE\EpicJam`.
- Do not open the Unreal Editor and do not build. Other executors work there.
- Do not publish the page, do not download anything, do not commit, do not delete files.

## 7. Report

English, `Docs/briefs/reports/day1-progression-probe.md`, at most 20 lines: the table of changes, the checks, what a player will notice first, questions.

## 8. Chain of command

You report to the orchestrator, not to the owner. The orchestrator checks the page in a browser and publishes it for the owner.
