# Brief: Russian text inside the browser prototype

Date: 2026-11-12, 03:39 (owner's clock). Written by the orchestrator. Executor: Opus, effort high.

## 1. Goal

The owner asked: «хоть игра будет на английском, пожалуйста в артефакте переведи на русский». Translate every text that the prototype draws inside the canvas into Russian, so that the owner reads the prototype in his own language. The Unreal game stays English; this change is only for the throwaway prototype.

## 2. What exists now

- File: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\catch-prototype.html`. It is published as version 3. A backup of this English version lies next to it as `catch-prototype.v2c-en.html`. Do not edit the backup.
- The orchestrator changed the file after your last write. Read the parts you touch again before editing. The changes: in `bandOf` a hunter whose home zone is locked no longer roams into neighbouring zones; `tryPredator` skips hunters of locked zones; the locked-zone label is not drawn when its centre is above y 150; `enterPrep` sets `G.camT = 0`; one word in the Russian page text.
- The page text around the canvas is already Russian.
- `smoke.js` currently fails when Cyrillic text is drawn on the canvas. That check must turn around.

## 3. Technical decisions (revocable; say so in the report if you change one)

1. Translate in place. Keep ids, data keys and the debug hook unchanged. Only displayed strings change.
2. Money keeps the sign `$` in front of the number. Multipliers keep the form `x2`. Level is `ур. 3`.
3. Font: Rubik has Cyrillic. Canvas text does not always trigger the loading of a font subset, so load it explicitly once at start: `document.fonts.load` for weights 400, 500, 700 and 800 with a Cyrillic sample string, inside try/catch. The game redraws every frame, so the text switches to Rubik when the font arrives.
4. Russian strings are longer. Where a string sits in a fixed box (rod card, shop tile, charm card, button, gauge label), measure it with `ctx.measureText` and reduce the font size until it fits, or wrap it. Add one small helper for that if it does not exist. Never let text leave its box.
5. `smoke.js`: remove the "no Cyrillic on the canvas" check. Add the opposite: collect every string passed to `fillText` during the run and fail if one contains an English word of three or more Latin letters. Allowed Latin: the single letter `x` in multipliers.

## 4. Glossary (use exactly these words; one word for one thing)

Interface:

| English | Russian |
| --- | --- |
| PLAY | ИГРАТЬ |
| START DAY / or press Space | НАЧАТЬ ДЕНЬ / или пробел |
| NEXT DAY | СЛЕДУЮЩИЙ ДЕНЬ |
| PAY $15 | ЗАПЛАТИТЬ $15 |
| NEW RUN | НОВЫЙ ЗАБЕГ |
| DAY 1 | ДЕНЬ 1 |
| MONEY | ДЕНЬГИ |
| CATCH 12 fish $87 | УЛОВ: 12 рыб, $87 (use the right plural: 1 рыба, 2 рыбы, 5 рыб) |
| fee tonight $15 | плата вечером $15 |
| FISHERMAN'S SHOP / after day 1 | МАГАЗИН РЫБАКА / после дня 1 |
| CHARMS, RODS, BAIT, FISH CARDS, UPGRADES, TOMORROW | МОДИФИКАТОРЫ, УДОЧКИ, НАЖИВКА, КАРТОЧКИ РЫБЫ, УЛУЧШЕНИЯ, ЗАВТРА |
| Charm shelf | Полка |
| Weightless / Takes no slot | Без места / Не занимает место |
| REROLL $3 | ДРУГИЕ $3 |
| SOLD | КУПЛЕНО |
| SHELF FULL | ПОЛКА ПОЛНА |
| SELL $10 | ПРОДАТЬ $10 |
| SHARPEN | ЗАТОЧИТЬ |
| NEW ROD | НОВАЯ УДОЧКА |
| ROD 1, Lv 3, 2 hooks | УДОЧКА 1, ур. 3, 2 крючка |
| DEEPER LINE | ДЛИННЕЕ ЛЕСКА |
| BIGGER SHELF | ПОЛКА БОЛЬШЕ |
| BAIT BOX | ЯЩИК НАЖИВКИ |
| OPEN PACK | ОТКРЫТЬ НАБОР |
| SMALL PACK / BIG PACK / WORM TIN | МАЛЫЙ НАБОР / БОЛЬШОЙ НАБОР / БАНКА ЧЕРВЕЙ |
| Pick 1 bait / Pick 2 baits | Выберите 1 наживку / Выберите 2 наживки |
| Catches: | Ловит: |
| x8 portions | 8 порций |
| Bare hook / BARE | Пустой крючок / ПУСТО |
| Common, Uncommon, Rare, Legendary | Обычная, Необычная, Редкая, Легендарная |
| CAST, SINK, WAIT, FISH ON, PULL, READY | ЗАБРОС, ВНИЗ, ЖДЁТ, РЫБА!, ТЯНЕТ, ГОТОВА |
| SLIPPING! | СРЫВАЕТСЯ! |
| HUNTER NEAR | ХИЩНИК РЯДОМ |
| CHAIN x2 | ЦЕПОЧКА x2 |
| SELLING THE CATCH | ПРОДАЖА УЛОВА |
| DAY 1 IS OVER | ДЕНЬ 1 ОКОНЧЕН |
| THE RUN IS OVER | ЗАБЕГ ОКОНЧЕН |
| SPECIAL FISH / Rule of the day | ОСОБАЯ РЫБА / Правило дня |
| LOCKED / Buy Deeper Line | ЗАКРЫТО / Купите «Длиннее леска» |
| Rod, Bait, Fish, Sea (charm classes) | Удочка, Наживка, Рыба, Море |

Depth zones: Sunlight - Солнце; Twilight - Сумерки; Midnight - Полночь; Abyss - Бездна; Trench - Жёлоб; Seabed - Дно.

Fish: Sprat - Килька; Mackerel - Скумбрия; Sea Bass - Морской окунь; Lanternfish - Рыба-фонарик; Cod - Треска; Tuna - Тунец; Hatchetfish - Рыба-топорик; Anglerfish - Удильщик; Oarfish - Сельдяной король; Gulper Eel - Мешкорот; Snailfish - Липарис; Giant Squid - Гигантский кальмар; Flounder - Камбала; Coelacanth - Латимерия; Gold Sunfish - Золотая рыба-луна.

Baits: Worm - Червь; Bread - Хлеб; Shrimp - Креветка; Cut Bait - Нарезка; Glow Worm - Светлячок; Squid Strip - Кальмар; Crab - Краб; Live Sprat - Живая килька; Shiny Spoon - Блесна; Deep Lure - Глубинная приманка; Golden Grub - Золотая личинка; Siren Pearl - Жемчужина сирены.

Charms: Auto-Reel - Автокатушка; Barbed Hooks - Крючки с бородкой; Cluster Hook - Гроздь; Old Faithful - Верная удочка; Big Haul - Большой улов; Bait Knife - Нож для наживки; Thrifty Hook - Бережливый крючок; Chum Bucket - Ведро прикормки; Lucky Pack - Счастливый набор; Fresh Scent - Свежий запах; Sprat Fever - Килечная лихорадка; Giants - Гиганты; Same Again - Ещё такую же; Big Shoals - Большие стаи; Food Chain - Пищевая цепь; Low Tide - Отлив; Long Day - Долгий день; Warm Current - Тёплое течение; Dusk Frenzy - Вечерний жор; Deep Pressure - Давление глубины.

The short tags on the shelf cards (now `DEEP`, `x5`, `BARB` and so on): use a short Russian word or a sign of at most five letters.

## 5. Style of the Russian sentences

The owner reads by these rules. Apply them to every rule text, hint, banner and tooltip:

- Active voice. It is clear who does what.
- At most 20 words in a sentence. One idea in a sentence.
- One word for one thing: use the glossary. Do not alternate synonyms.
- No idioms, no slang, no gamer jargon.
- Address the player with «вы» and the imperative: «Нажмите», «Выберите», «Потяните».
- Examples: "Hooked fish never escape." - «Рыба на крючке не срывается.» "Pull now for the money, or wait: a bigger fish may eat it." - «Тяните сейчас и получите деньги. Или ждите: рыбу может съесть хищник крупнее.» "Fish ignore Worm." - «Рыба не клюёт на червя.» "Half of your casts use no bait." - «Половина забросов не тратит наживку.»

## 6. Done means

1. No English word remains in anything the canvas draws: HUD, rod strip, bait box, menus, pack cards, hints, banners, shop, shelf, tooltips, depth gauge, locked-zone labels, summary, shipping bar, end screens, pop-up labels such as `+1 ON HOOK`.
2. The names in section 4 are used exactly.
3. No text leaves its box at 1280x720.
4. `node smoke.js` passes with the new Latin-word check, and still plays twelve days.
5. The page text around the canvas uses the same words as the canvas (for example the "Как играть" box must say «НАЧАТЬ ДЕНЬ» and «ОТКРЫТЬ НАБОР» where it now says START DAY and OPEN PACK).

## 7. Verification

```bash
node "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/proto-test/smoke.js"
```

You cannot see the canvas. Do not open a browser. The orchestrator checks the frames.

## 8. Boundaries

- Write only inside the two scratchpad folders `catch-prototype` and `proto-test`. The repository is read-only for you. No git. Do not publish. No servers, no browser, no network.
- Do not change rules, numbers or behaviour. Text only, plus the fit helper and the font loading.
- Bash heredocs with Cyrillic fail in this harness. Use the Edit and Write tools.

## 9. Report

English, at most 20 lines: what you translated, strings you shortened or abbreviated to fit, places where you expect an overflow that only a frame can confirm, the last lines of the smoke test.

## 10. Chain of command

You report to the orchestrator. Do not wait for answers; choose the simplest option and note it.
