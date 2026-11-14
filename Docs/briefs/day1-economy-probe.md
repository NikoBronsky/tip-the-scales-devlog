# Brief: the money probe — the catch pays the fee, the shop runs on coins

Status: written on 12.11.2026 at 14:06 (owner's clock). Executor: Codex, effort high. Work folder: the prototype folder in the scratchpad (path in section 2). The Unreal project is not touched.

## 1. Goal

The owner played the progression probe and said it is not enough. The owner's words (Russian, verbatim):

- «Прогрессия стала лучше, но не намного. на 5 дне у меня уже 8500.» (The fee on day 5 was $300.)
- «я не считаю что мы должны ограничивать игрока в покупках до 3го дня. А вот сделать там чтобы денег у него хватало в притык нужно.»
- «мы сейчас просто улучшаем ценность рыбы, а могли бы ещё за деньги увеличивать количество рыб (прикормка) определённого типа.»

The orchestrator showed the owner why money is never tight: the value of the fish is at once the score against the fee and the money for the shop. The owner chose the fix: «Как в Balatro». The catch of the day pays the fee. The shop money is a separate small payout. Prices are small.

Build a browser probe of exactly this, so that the owner can feel it today. It is a decision aid. Nothing from it goes into the Unreal game.

## 2. What exists (verified by the orchestrator)

- Folder: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\`.
- `catch-progression.html` (2,959 lines, one self-contained page, canvas game, Russian interface). It is the probe the owner just played. Start from a copy of it.
- In it: `var LEVELS` at line 225 (three fee levels), `var PROBE` at line 233 (`{ hunterDay: 3, bareZone: 0, deeperDay: 2, charmDay: 3, newRod: [...], smallPack: 4 }`), `probeOn()` at line 431, `feeOf(day)` at line 438, the shop and summary drawing near lines 2640–2730, and the test interface `window.CatchProto` at line 2927 (`state()`, `step(dt)`, `draw`, `startDay`, `gotoShop`, `buttons()` where each button has `id`, `fn`, `off`, `addMoney`, `fee(day)`, `data.PROBE`).
- `preview.html` in the same folder: its first line is a small wrapper with the charset. A local preview page is that first line followed by the whole page.
- The rules of the game in words: `F:\UE\EpicJam\Docs\design\rules-reference.md` (read-only, if you need the meaning of a rule).

## 3. Rules of this probe (decisions of the orchestrator; all numbers are temporary and live in one table)

Put every number below into one table `var ECON = {...}` near `PROBE`, so that a retune is a one-minute edit. Expose it as `CatchProto.data.ECON`.

1. **Two numbers.** Coins: the shop money, starts at 0. Day value: the sum of the prices of the fish landed today, computed exactly as the page computes fish prices now (all charms, chains and levels apply).
2. **End of the day.** Shipping stays: fish leave one by one and each adds its price to the day value on screen, not to the coins. After shipping: day value at or above the fee means the fee is paid and the payout comes; below the fee the run is over. Coins never pay the fee.
3. **Payout in coins** = base 4, plus 1 for each full 25% of the fee that the day value is above the fee (at most +4), plus interest: 1 for each 5 coins held before the payout (at most +3).
4. **Special fish** (every third day, as now): its price counts toward the day value as now, and landing it gives +3 coins at once.
5. **Fee curves**, three levels in `LEVELS`, default Normal. Days 1–9, then the same multiplier per day, rounded as the page rounds now.
   - Soft: 25, 40, 65, 100, 165, 260, 420, 670, 1070 (x1.6).
   - Normal: 25, 50, 100, 200, 400, 800, 1600, 3200, 6400 (x2).
   - Hard: 25, 60, 150, 380, 950, 2400, 6000, 15000, 37500 (x2.5).
6. **Prices in coins.**
   - Charms by their old price: 15 → 4, 20 → 5, 25 → 6, 30 → 7, 35 → 8. Weightless: x1.5, rounded up. Selling returns half, rounded down.
   - Reroll 2, +1 for each reroll in the same shop.
   - Small pack 2, big pack 4, worm tin 1.
   - Fish card 3, +1 per level that the species already has.
   - Chum 3, +1 per chum level that the species already has.
   - New Rod 5, 8, 12, 17. Sharpen Rod 2, 4, 6, 9. Deeper Line 6, 10, 15, 21, 28. Bigger Shelf 10, 18.
7. **No day locks on purchases.** Everything the shop sells is on sale from the first shop: all charms of all tiers, three charm offers, Deeper Line, fish cards, big pack, Bigger Shelf. Remove `deeperDay` and `charmDay` and the older day locks. Bait still follows depth as now.
8. **Hunters** arrive on day 2 (`hunterDay: 2`). The bare hook rule of the probe stays (`bareZone: 0`).
9. **Chum (new purchase).** A shop row "Прикормка" with two offers per shop: two random species from the open zones, never the special fish and never the Coelacanth. Buying raises the chum level of that species by 1, at most 5. From the next day the sea holds more fish of that species: base number x (1 + 0.5 x level). Other species keep their numbers. The level shows on the offer and in the fish hover card ("Прикормка ур. 2: рыбы больше на 100%").
10. **The 20 charms keep their rules.** A new list comes in the next probe. The label of the probe says so.
11. **Run log.** Keep one record per day: day, level, day value, fee, the three payout parts, coins after the shop, purchases by name, rods, open zones, charms on the shelf, chum and fish card levels. A button "Скопировать журнал забега" on the summary panel, on the run-over panel and in the shop copies the log as text to the clipboard; if the clipboard is refused, show the text in a box that the player can select. Also `CatchProto.log()`.
12. **Words on screen** (Russian, this page is for the owner). Money panel: coins with a coin mark, for example "Монеты 7". Day line: "Улов $120 из $200". Shop prices with the coin mark, not with "$". Summary rows: "Улов дня", "Плата", "Выплата", "За перевыполнение", "Процент с монет", "Итого монет". Run over: "Улов не закрыл плату". Probe label: "Проба денег: улов закрывает плату, магазин за монеты. Модификаторы пока старые." Page title: "Проба денег Catch". Fish prices keep "$".
13. Everything else stays as it is: the look, the sounds, the fish, the baits, the rod cycle, the chain, the day rules of the special fish.

## 4. Work

1. Copy `catch-progression.html` to `catch-economy.html`. Change only the copy.
2. Implement section 3. Keep the change small and local; do not restyle or refactor.
3. Extend the test interface: `state()` also returns `coins`, `dayValue`, `fee`, `lastPayout` (the three parts), `chum` (levels by species); `addMoney(n)` adds coins; add `CatchProto.selfTest()`: it checks the payout arithmetic for a list of cases (day value below the fee, at the fee, 24% over, 25% over, 100% over, 300% over; coins 0, 4, 5, 14, 15, 40) and the price table, and returns `{ ok, failed: [...] }` without touching the running game.
4. Write `preview-economy.html`: the first line of `preview.html`, then the whole of `catch-economy.html`.
5. Write `economy-probe-report.md` in the same folder.

## 5. Done means

1. `catch-economy.html` loads with no console error, and a run can be played from day 1 through at least three shops.
2. `CatchProto.selfTest().ok` is true.
3. The script of the page passes a syntax check (for example `node --check` on the extracted script), if node is on the machine. Say if it is not.
4. `catch-progression.html`, `catch-prototype.html` and `preview.html` are unchanged, byte for byte. Give their SHA-256 before and after.
5. The report lists every changed place by function name, in the order of section 3.

## 6. Boundaries

- Write only inside the prototype folder of section 2, and only the three new files. Read `F:\UE\EpicJam\Docs\design\rules-reference.md` if you need it; change nothing under `F:\UE`.
- No network, no packages, no build tools. The page stays one self-contained file.
- You do not invent the game. A rule that section 3 does not give: take the plainest reading, mark it in the report.
- Do not publish the page anywhere. No git.

## 7. Report

English, `economy-probe-report.md`, at most 35 lines: what changed, by function; the self-test result; the syntax check; the hashes; readings you chose where the brief was silent; what you could not verify without a browser.

## 8. Chain of command

You report to the orchestrator. The orchestrator loads the page in a browser, plays three days through the test interface, and then gives it to the owner.
