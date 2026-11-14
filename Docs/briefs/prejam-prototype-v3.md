# Brief: browser prototype, third version (owner's play feedback)

Date: 2026-11-12, 04:07 (owner's clock). Written by the orchestrator. Executor: Opus, effort high.

## 1. Goal

The owner played the second prototype and said: «Поиграл очень здорово. Выглядит практически готово.» He named five weak points. Fix them in the same throwaway prototype, so that he can judge the answers when he wakes up. The prototype is a decision aid. Nothing from it goes into the Unreal game.

The owner's words, one per work block:

- A: «в самом низу я смог наживку поместить очень низко, но не смог переместить крючок выше. Я по нему нажимаю, он не тянется.»
- B: «Когда мы вытаскиваем сразу 5 рыб, у нас в ведерко летит 1 рыбка. Хотелось бы, чтобы все 5 красиво летели, и самое главное, чтобы оно взаимодействовало с ведерком ... звук, как оно падает в ведерко. Ведерко должно при этом как-то шататься, плескаться водичка.»
- C: «Хотелось бы, когда я еще не начал день, чтобы я мог навести мышку на рыбу и понять ... что вот это вот такая-то рыба, она ловится на то-то и на то-то.»
- D: «как будто бы слишком легко. Я дошел до 20 тысяч, а плачу при этом 194. То есть должна быть какая-то прогрессия ... может быть, уровни сложности имеет смысл менять».
- E: «открывать их по порядку, чтобы мы виды наживки и так далее открывали это все постепенно.»

## 2. What exists now (verified by the orchestrator at 04:05)

File: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\catch-prototype.html`, 2665 lines. All canvas text is Russian since 03:57. Test: `...\scratchpad\proto-test\smoke.js`, which ends with `SMOKE OK`. A backup of the current file lies next to it as `catch-prototype.v2d-ru.html`. Do not edit backups.

- Line 110: `WORLD_D = 3450, FLOOR = 3400`. Lines 128-135: six zones; the sixth («Дно») spans 2500 to 3400.
- Line 433: `goalDepth(r)` snaps the goal to `FLOOR - 10` whenever the depth lies anywhere in the sixth zone. Line 1180: `handleOf(r)` draws the drag handle at `goalDepth(r)`. Line 1294: a depth drag writes `dr.depth` from the pointer. Result: inside the sixth zone, 900 px tall, the handle stays on the floor while the pointer moves. This is bug A.
- Line 840: `land(r)`; line 834 builds one flying fish `fl`. Lines 1299-1312: `crateXY`, `crateCount`, landing sets `G.crateBump = 1`. Line 1812: `drawCrate`. Line 2151: the catch icon in the HUD.
- Line 397: `feeOf(day)` is `15 * 1.5^(day-1)`. Line 213: `NEWROD_PRICES`, `SHARPEN_PRICES`, `DEEPER_PRICES = [25, 70, 160, 380, 900]`, `SHELF_PRICES`.
- Line 1000: `rollOffers`. Line 1010: `openShop`. Line 1098: `rollPack(n)`. Lines 161-174: `BAITS` with `catches` or `any`. Lines 182-203: `CHARMS`.
- Lines 250-272: `fitText`, `wrapFit`, `plural`. Use them for all new text.
- Lines 90-98: the Russian settings box of the page with `opt-win`, `opt-sound`, `btn-skip`, `btn-new`. Line 2640: `window.CatchProto` debug hook with `settings: SET`.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. Edit in place. Keep ids, data keys and the existing debug hook. Add to the hook, do not rename.
2. All new canvas text is Russian, by the glossary and the style rules of `F:\UE\EpicJam\Docs\briefs\prejam-prototype-v2-russian.md`, sections 4 and 5. The smoke test rejects Latin words on the canvas.
3. All numbers below are temporary. Put them in tables at the top of the script, next to the existing price tables.
4. The catch container is now called «ведро» everywhere (canvas and page text). Replace the word «ящик» for the catch container. The bait box keeps its name «ЯЩИК НАЖИВКИ».

## 4. Work blocks

### A. Hook that does not go up (bug)

- While the player drags the depth handle, the handle and the goal marker follow the pointer exactly, in every zone. No snap during the drag.
- The snap to the floor stays for the simulation, on release: a goal inside the sixth zone means the floor.
- While the pointer is inside the sixth zone during a drag, show the label «ДНО» at the marker, so the player knows the hook will go to the floor.
- Check the same path during the day for a hook that already lies on the floor: the player grabs it and drags it up; the hook must follow.
- Test in `smoke.js`: put a rod on the floor, drag from the handle 200 px up with steps, assert the handle follows the pointer; continue above 2500 and assert the goal equals the pointer depth.

### B. Every fish flies to the bucket; the bucket answers

- Every landed fish gets its own flight: every hook, every extra fish of «Гроздь», every link result of a chain. Stagger the starts by 70 ms. Vary the arcs a little.
- Draw the container on the raft as a wooden bucket with water in it. The fish count still shows.
- On each landing: the bucket gets an impulse (a damped spring: tilt up to 7 degrees, squash), 4-6 water drops fly up, the water line in the bucket rocks, a short watery plop plays. The pitch rises with each fish of the same haul and resets after one second without landings. Keep at least 40 ms between plops.
- The catch counter in the HUD grows per landing, with a small bump.
- The day-end shipping takes the fish out of the bucket; the bucket rocks there too.
- Test: a haul of five fish creates five flights and five landings; the crate sum is unchanged against the old rule.

### C. Fish info under the cursor

- In preparation and during the day: the fish under the cursor gets a white outline. Other fish of the same kind get a soft glow, so the player sees which fish is which.
- A card appears near the cursor, inside the canvas, never under the rod strip:
  - name; zone name; price now (with Fish Card level and price charms, if a pure function gives it; else the base price);
  - «Ловит:» and the baits that catch it. Baits the player holds: white, with the portion count. Baits he does not hold: grey. Include baits with `any` when their rule covers the fish;
  - for a shoal fish: «Клюёт и на пустой крючок.»;
  - for a hunter: «Хищник. Съедает рыбу мельче себя на крючке и попадается сам.»;
  - for a floor fish: «Живёт на дне.»
- A fish of a locked zone (a shadow) shows: «Неизвестная рыба.» and «Купите «Длиннее леска», чтобы открыть зону <имя>.»
- No card while the player drags, while a menu or a pack is open, or while the cursor is over the rod strip, the bait box or the depth gauge.
- Hit test: the nearest fish within `max(len * scale * 0.6, 22)` px.
- Add to the hook: `CatchProto.hover(x, y)` returns the card data (or null) for canvas coordinates. Test it for a shoal fish, a hunter and a shadow.

### D. Fee levels (the game was too easy)

Facts from the owner's run: about $20,000 in hand at a fee of $194. Cause found by the orchestrator: the fee grows by 1.5 per day; income grows by about 2.3 per day while zones open; depth costs almost nothing.

- Add a setting «Плата» with three levels to the page settings box (radio buttons) and to `SET.level`: `"soft"`, `"normal"`, `"hard"`. Default `"normal"`. A change works at once for the next fee and for the prices shown.

| Level | Page label | Fee by day | Deeper Line prices |
| --- | --- | --- | --- |
| soft | Мягкая (как было) | `15 * 1.5^(day-1)` | 25, 70, 160, 380, 900 |
| normal | Обычная | 15, 30, 70, 150, 320, 700, 1500, 3200, 7000, then x2.15 per day | 25, 150, 700, 3000, 12000 |
| hard | Жёсткая | 15, 38, 95, 235, 590, 1450, 3650, 9150, 22900, then x2.5 per day | 25, 150, 700, 3000, 12000 |

- Round fees after day 9 to two significant digits.
- The HUD already shows the fee of tonight. Add the fee of tomorrow to the day summary and to the shop block «ЗАВТРА»: «Плата завтра: $N».
- Add to the hook: `CatchProto.fee(day)`.
- `smoke.js`: run the twelve-day play on `soft`. Assert the tables of `normal` and `hard`. Then run the same bot on `normal` and on `hard` for up to 15 days and print the day on which it fails, or "survived". Do not assert survival there. Put both lines in your report.

### E. The shop opens step by step

Reading of the orchestrator; the owner may correct it. The run starts small and each shop visit may show something new.

| Thing | Opens |
| --- | --- |
| Заточить, Новая удочка, Малый набор, Банка червей, Длиннее леска | first shop (after day 1) |
| Charm offers | two offers in the first shop; three from the shop after day 2 |
| Charms, step 1: `barbed`, `thrifty`, `knife`, `lucky`, `fever`, `shoals`, `warm`, `longday` | first shop |
| Charms, step 2: `faithful`, `bighaul`, `chum`, `scent`, `same`, `foodchain`, `dusk` | shop after day 2 |
| Charms, step 3: `autoreel`, `cluster`, `giants`, `lowtide`, `pressure` | shop after day 4 |
| Карточки рыбы | shop after day 2 |
| Большой набор | shop after day 3 |
| Полка больше | shop after day 4 |

Baits open with depth, at the moment a pack is opened:

- A bait with a `catches` list may appear when at least one fish of the list lives in an open zone.
- `spoon` may appear from the start. `lure`, `grub` and `pearl` may appear when the third zone («Полночь») is open.
- The free starting pack follows the same rule.

Show the steps:

- A thing that is new in this shop visit carries a badge «НОВОЕ».
- A section that is still closed stays visible as a dim tile with one line: «Откроется после дня 2». The player sees what comes next.
- When a purchase of «Длиннее леска» adds baits to the packs, show one banner: «В наборах появилась новая наживка: Светлячок, Кальмар, Краб.»
- The first time a bait kind appears on a pack card, the card carries «НОВОЕ».
- The debug hook `giveCharm(id)` and `openPack(kind)` keep working for tests; add an optional flag to bypass the gate if a test needs it.
- Tests: the first shop offers only step-1 charms and has no Big Pack, no Fish Cards, no Bigger Shelf; packs before the second zone hold only worm, bread, shrimp, cut, live, spoon; after day 4 a step-3 charm can appear. Every charm must still trigger somewhere in the test (use the hook).

### F. Page text

- The box «Что нового во второй пробе» becomes «Что нового в третьей пробе» and lists blocks A-E in short Russian sentences (active voice, at most 20 words, one idea per sentence).
- The box «Как играть» mentions the fish card under the cursor and the bucket.

## 5. Done means

1. Blocks A-E work as written. No rule or number changes beyond these blocks.
2. No English word on the canvas. No text leaves its box at 1280x720.
3. `node smoke.js` ends with `SMOKE OK` and holds the new tests.
4. The script part of the HTML passes a syntax check (the smoke test does it).

## 6. Verification

```bash
node "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/proto-test/smoke.js"
```

You cannot see the canvas. Do not open a browser. The orchestrator checks the frames.

## 7. Boundaries

- Write only inside the scratchpad folders `catch-prototype` and `proto-test`. The repository is read-only for you. No git. Do not publish. No servers, no browser, no network.
- Use the Read, Edit and Write tools for every edit. Shell heredocs, sed and PowerShell corrupt Cyrillic in this harness. Running `node` through the shell is fine.
- Priority if time runs short: A, D, B, C, E. Finish a block completely before the next one. Report what you left out.
- Time box: aim to finish within 45 minutes.

## 8. Report

English, at most 25 lines: what you did per block; decisions you changed; the bot's result on `normal` and `hard`; places where only a rendered frame can confirm the look (name the screen and the element); the last lines of the smoke test.

## 9. Chain of command

You report to the orchestrator. Do not wait for answers; choose the simplest option and note it.
