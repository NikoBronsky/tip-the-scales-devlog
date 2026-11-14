# Addendum 2 to brief day3-video-draft: the owner's structure for the video (14.11.2026 13:46)

Executor: the same Opus executor, effort high. The brief and addendum 1 blocks A-D and F still hold. This addendum replaces block E of addendum 1. You do not commit.

## 1. The owner's words (14.11, after watching draft 1)

"Драфт посмотрел, пока что отстой. Техническая проблема - курсор не виден. Художественная - ролик унылый, по сути он рассказывает что итак есть в подсказка. Сделай ролик из трех частей. Первая - расслабояющая рыбалка, закинь удрчку и тяни рыбу. Вторая - улучшаем удочки, рыбу и т.п (лови рыбу больше). Третья - улучшения становятся более глобальными - гигантские рыбы, 8 удочек (или сколько там максимумум?) (БОЛЬШЕ РЫБЫ). Четвертая - безумные улучшения с крючками от верхаи до дна, взрывчатка, гарауны, килька есть рыбу больше себя, на крючке по 20 рыб (погрузись в рыбное безумие)"

The orchestrator's reading (the owner dictates by voice): the video is an escalation in four parts. It must not explain rules the way the hints do. It must show the game growing from calm fishing into madness. The words in brackets are the captions of the parts. The cursor must be visible.

## 2. What the game really has (checked by the orchestrator in the core; do not show more than this)

- Rods: 5 at most (one at the start, four bought: `Tables.cpp:71`, prices 5, 8, 12, 17). Not 8.
- Hooks: a rod gets a hook at levels 3 and 5 (3 hooks), the charm Trotline adds 2 more: 5 hooks per rod at most.
- Fish per hook: the charm Cluster Hook draws up to 4 more fish of a chummed shoal kind to a hook: 5 fish per hook at most, so up to 25 fish on one rod. "20 fish on one hook" does not exist; "25 fish on one rod" does.
- Giant fish: Giants (half as many fish, every fish twice as big), Whopper (once a day a fish ten times larger enters the sea).
- "The sprat eats a fish bigger than itself": Tiny Terror (any fish of another kind may eat the hooked fish, even a smaller one).
- More fish on screen: Big Shoals (chummed shoal kinds have twice as many fish), Warm Current.
- Deep hooks: the upgrade Deeper Line opens zones; Bottom Rig lets the lowest hook go one zone deeper.
- Dynamite and Harpoon exist only with the launch argument `-new` (then: 20 start coins, both stand in the first shop, the rule day of day 3 is Poachers). The owner has not accepted them into the default game yet. He wants them in the video. So: record them, but keep every Dynamite or Harpoon shot as its own segment that can be dropped without re-cutting the rest.
- The full list of charms with their rule texts: `F:\UE\EpicJam_ui\Docs\design\modifiers-v2.data.json` (`ruleEn`).

## 3. The four parts

Total 45-60 seconds. The pace rises: part 1 has one or two long calm shots, every next part cuts faster and has more on screen, part 4 is the fastest and the fullest. No caption explains a rule. Captions: one per part, the owner's words in English (first version, the owner will edit):

| Part | About | Length | What the viewer sees | Caption |
| --- | --- | --- | --- | --- |
| 1 | calm fishing | 8-10 s | one rod, quiet sea; the cursor baits the hook, the line sinks, a fish bites, the cursor pulls, the fish flies into the bucket, the scale tips | Cast a line. Pull a fish. |
| 2 | better rods and fish | 10-12 s | the shop: the cursor buys a rod, sharpens a rod, buys a fish card or chum; then two or three rods with more hooks land bigger pulls | Catch more fish. |
| 3 | global upgrades | 12-14 s | Giants and the Whopper, five rods, deeper zones, big numbers on the scale | MORE FISH. |
| 4 | madness | 14-16 s | five rods with five hooks each, hooks full of fish (Cluster Hook), a small fish eating a bigger one (Tiny Terror), chains, the Dynamite blast, the Harpoon ricochet, fish raining into the bucket | Dive into the fish madness. |

Then the end card with the game name "Tip the Scales" (2-3 s).

## 4. Technical decisions of the orchestrator (revocable; say in the report what you changed)

1. The cursor. In every shot where the player acts, the game's own cursor is visible and moves like a hand: it glides to the bait, the rod, the fish or the shop card (`catch.HudCursor x y seconds`), and the action command fires when it arrives. Never park it off screen in a shown shot. Find how the HUD draws this cursor and whether it shows a click; report it. If the game cursor cannot be made visible with existing commands, stop and report: do not draw a fake cursor in ffmpeg.
2. Shown footage runs at real speed. `catch.TimeScale` is allowed only in stretches that are cut away.
3. Late-game states may be staged with the run commands (`catch.FrameRun`, `catch.CheatRun`, `catch.Buy`, money). Parts 1 and 2 must show small amounts in the wallet that a player could have. Parts 3 and 4 may show a rich wallet, but not a round test number.
4. The game's first-day hint plates must not fill parts 2-4. For part 1 find a way without them (a later day, a command, a launch option). No game code changes. Report what exists.
5. Takes for Dynamite and Harpoon use `-new` through the per-take launch arguments. All other takes use the default game.
6. Two edit files from the same set: `edit.json` in the owner's order (calm first) gives `draft2.mp4`; `edit_coldopen.json` is the same video with a flash of the madness of 1.5-2 seconds before part 1 and gives `draft2_coldopen.mp4`. The second is the orchestrator's counter-proposal for the owner to compare; render both.
7. A third edit file, not rendered: `edit_no_candidates.json`, part 4 without the Dynamite and Harpoon segments, still 45-60 seconds.
8. Check the grab frame rate in the fullest shots of part 4 and report it. If the game drops under 60 frames per second there, say so.

## 5. Done means

- `Saved/video/draft2.mp4` and `Saved/video/draft2_coldopen.mp4`: 45-60 s, 1920x1080, 60 fps, sound not silent, with their sheets (`_sheet`, `_cuts`, `_corner`).
- The four parts are there in the owner's order, each with its one caption; the cursor is visible wherever the player acts; no rule-explaining captions.
- Nothing is shown that the build cannot do. Every Dynamite and Harpoon shot is a separate segment.
- The checks of brief section 5 and of addendum 1 section 3 hold. You looked at the sheets and at the first and last frame of each cut.
- The gate is green.

## 5a. Correction by the owner (14.11, after reading the plan of part 4)

"Нет, это не модификаторы безумия, это должна быть комбинация модификаторов"

The orchestrator's reading: the madness of part 4 is not a parade of single "mad" charms shown one after another. It is one build: several charms on the shelf at once, working together in the same shots. This changes sections 3 and 4:

- The video is the story of one build that grows. What the cursor buys in part 2 stays on the shelf in part 3; part 3 adds to it; in part 4 the shelf is full and everything fires together. The shelf with its charms is in frame, so the viewer sees that the madness comes from the set.
- Part 4 is staged as one run with one full shelf: five rods, five hooks each, Cluster Hook, Big Shoals, Tiny Terror and the chain charms, and - in the take with `-new` - Dynamite and Harpoon in the same pulls as the rest. A blast that throws clusters of fish into the bucket while a small fish eats a bigger one on the next rod is the picture to look for.
- This replaces "every Dynamite or Harpoon shot is its own segment" (section 2, section 4 points 5 and 7, section 5). Instead record part 4 twice from the same staging: the take `madness` with `-new` and both charms on the shelf, and the take `madness_plain` without them (the default game, the rest of the build the same). `edit.json` and `edit_coldopen.json` use `madness`; `edit_no_candidates.json` uses `madness_plain`.
- If the shelf has too few places for the build, use what the game gives for more places (the upgrade Bigger Shelf, the charms Rod Stand and Sea Chest) and say what you used.

## 6. Report

Append a section "Addendum 2" to `Docs/briefs/reports/day3-video-draft.md`: the shot list of both files with times, how each state was staged (commands), the cursor finding, the hint-plate finding, the frame rate in part 4, what you could not stage and why, what is not verified. Time box: about 90 minutes; if a shot costs more than 15 minutes of trying, replace it with the nearest thing the build does and say so. Final message: paths, durations, the shot list, the weakest points, what is not verified, what the orchestrator must decide.
