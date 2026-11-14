# Brief: the modifier probe — 50 data-driven charms on top of the money probe

Status: written on 12.11.2026 at 14:42 (owner's clock). Executor: Codex, then Opus from 14:50 (section 9), effort high. Work folder: the prototype folder in the scratchpad (path in section 2). The Unreal project is not touched.

## 1. Goal

The owner rejected the 20 present charms: they all pay at once. The owner's words (Russian, verbatim): «Сейчас механики работают все и сразу, а надо чтобы они срабатывали если игрок подготовил почву... собрать комбинацию должно быть трудно потому что модификаторов очень много. Кроме того хотелось бы больше синергий наших улучшений.»

A designer wrote a new list of 50 charms as data: every charm is a few rows of trigger, conditions and effect from a closed vocabulary. Build a browser probe that plays this list, so that the owner can judge it today. The probe is also the first test of the vocabulary: later the C++ rules core implements the same data. So the charms must run through one row evaluator, not through 50 hand-written branches.

It is a decision aid. Nothing from it goes into the Unreal game.

## 2. What exists (verified by the orchestrator)

Folder: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\`.

- `catch-economy.html` — the money probe you built today, with five small edits by the orchestrator (font link back, first fish card costs 3, compact run log, new "what is new" box, day-1 fee 15). It is one self-contained page: canvas game, Russian interface. Start from a copy of it. `var ECON` holds all money numbers; `window.CatchProto` is the test interface (`state()`, `step(dt)`, `draw()`, `buttons()` with `id`, `fn`, `off`; `giveCharm`, `addMoney`, `setBait`, `fee(day)`, `selfTest()`, `log()`, `newRun()`).
- `modifiers-v2.data.json` — the charm data (a copy of `F:\UE\EpicJam\Docs\design\modifiers-v2.data.json`). Top keys: `combineOrder`, `shelfOrderMatters`, `rarities`, `weightless`, `shop`, `ground`, `primitives` (`triggers`, `conditions`, `effects`, `counts`, `ruleFlags`, each with its parameters and its meaning in one line), `charms` (50), `builds`, `fishPrices`, `fishPricesOld`, `dayRules`.
- `modifiers-v2-proposal.md` — the design text. Read section 2 (ground), section 3 (the vocabulary and the combine order; this is the specification of the evaluator), section 4 (the charm table) and section 7 (what became of each old charm).
- `preview.html` — its first line is the wrapper for a local preview page.
- `F:\UE\EpicJam\Docs\design\rules-reference.md` — the exact rules of the game in words (read-only), if you need the meaning of a rule that a flag names (auto-reel 6.3, cluster 5.7, low tide 2.3, wanted count 2.4).

One charm object, as an example of the shape:

```json
{"id": "sprat_fever", "nameEn": "Sprat Fever", "nameRu": "Килькомания", "class": "fish", "rarity": "common", "price": 4,
 "ruleRu": "Килька дороже: +x1 за каждый уровень прикормки кильки (сейчас x{n}).", "unconditional": false, "needs": ["chum"],
 "rows": [{"trigger": "on_land", "conditions": [{"kind": "fish", "species": ["sprat"]}], "effect": {"kind": "mult", "value": 1, "per": "species_chum"}}]}
```

A scaled effect uses `count = max(0, floor(source / perStep) - minus)`, then `cap`. `{n}` in a rule line is the live value of the charm: its counter, or the scaled value of its first scaled row.

## 3. Technical decisions (orchestrator; revocable — if one breaks the page, change it and say so in the report)

1. **The data is embedded, not retyped.** The page holds the JSON verbatim between two markers: `/*MOD-DATA-BEGIN*/ var MOD = { ... }; /*MOD-DATA-END*/`. A script `embed_mod_data.py` (Python 3, standard library) replaces the text between the markers with the current `modifiers-v2.data.json` and rebuilds the preview page. The orchestrator will change numbers in the JSON after the owner plays; a retune must be: edit JSON, run the script, reload.
2. **One row evaluator.** A table of handlers keyed by id for the 8 triggers, 15 conditions, 14 effects and 17 counts of `MOD.primitives`. The game asks the evaluator at fixed points: the price of a landed fish (`on_land`: flat, percent, mult, chain_step), a pull (`on_pull`), an escape, a chain link, the day end (after the fee is paid), the shop opening, a purchase, and the passive values: wanted count per kind, bite rate per fish, hunter factor per hook, day length, shelf slots, pack choices, and the rule flags (`auto_reel`, `no_escape`, `cluster`, `bait_knife`, `giant_size`, `low_tide` per kind, `no_day_rule`). No charm id appears in game logic outside the data, except where a rule flag already has code.
3. **The old 20 charms leave the page.** `CHARMS` and every `has("...")` branch are replaced by the evaluator and the flags. The effects that old code gave to a flag (auto-reel, barbed hooks, cluster, bait knife, giants, low tide) stay as code behind the flag.
4. **The price of a fish** follows `MOD.combineOrder` exactly: Sunfish and Bait Knife first; base = table price x card level; flat sum; one percent sum; multipliers with the chain factor and the Golden Grub; round halves up; then the day rule "under $10 pays nothing" if it is on. The pull preview and the fish hover card use the same function.
5. **Fish prices** come from `MOD.fishPrices`.
6. **Shop.** Three charm offers drawn by `MOD.rarities[...].weight`, never a charm that is on the shelf or already offered. Prices by rarity; Weightless by `MOD.weightless` (chance 10%, price x1.5 rounded up, takes no slot). Selling returns half, rounded down, and drops the counter. All 50 charms can appear from the first shop. Reroll as now. The chum row, the fish card row, the packs, the tin and the upgrades stay as the money probe has them.
7. **Shelf entry**: `{ id, wl, counter }`. A counter starts at `counter.start`, moves by `counter_add` and `counter_reset`, stops at `counter.cap`, and is lost on sale.
8. **Charm card** (shop offer, shelf tooltip): `nameRu`; the class colour as now; a rarity word («обычный», «необычный», «редкий»); `ruleRu` with `{n}` replaced by the live value; one more line «нужно: ...» built from `needs` through `MOD.ground[].labelRu` (skip the line when `needs` is empty). Fit the text by wrapping and, if needed, a smaller size; no text may leave the card.
9. **Feedback stays.** When a row of a charm changes the price of a landed fish, gives coins, adds a chum level or moves its counter, its shelf chip wobbles and shows a short tag, as the old charms did.
10. **Day rule 3 switch.** `ECON.rule3Shallow`, default `false`: rule 3 stays as it is now. When `true`, rule 3 is the proposal in `MOD.dayRules` (a hook above depth 100 attracts no fish; the Gold Sunfish still bites) with its Russian text «У поверхности рыба не клюёт. Опустите крючки ниже.»
11. **Money is not touched.** `ECON`, fees, payout, coin prices, chum, fish cards, no day locks, hunters from day 2: all as in `catch-economy.html`.
12. **Run log additions**, per day: fish landed by kind (kinds with at least one fish), pulls, chains of 2 or more (count and the longest), the charms on the shelf with their counters, the day rule if any. Keep the log compact: one line per item.
13. **Words on screen** (Russian; this page is for the owner). Title: «Проба модификаторов Catch». Probe label: «Проба модификаторов: 50 новых. Модификатор платит, когда вы подготовили для него почву. Цены рыбы новые.» The "what is new" box:
    - «В магазине 50 новых модификаторов. Старые 20 переделаны или убраны.»
    - «Модификатор платит, когда вы подготовили для него почву: прикормку, карточки рыбы, удочки, наживку, глубину или монеты.»
    - «Строка „нужно“ на карточке называет почву.»
    - «Число „сейчас“ на карточке показывает текущую силу модификатора.»
    - «Редкостей три: обычный стоит 4 монеты, необычный 6, редкий 8.»
    - «Цены рыбы новые. Каждая зона глубже даёт рыбу дороже примерно вдвое, а не в 4–5 раз.»
    - «Деньги работают как в пробе денег.»
    - «Журнал забега пишет рыбу по видам, цепочки и счётчики модификаторов.»
14. Everything else stays: the look, the sounds, the fish, the baits, the rod cycle, the chain, the special fish, the three fee levels.

## 4. Work

1. Copy `catch-economy.html` to `catch-modifiers.html`. Change only the copy.
2. Embed the data (decision 1) and write `embed_mod_data.py`.
3. Write the evaluator and wire it in (decisions 2–7). Remove the old charm code.
4. Cards, feedback, the rule switch, the log, the words (decisions 8–13).
5. Extend the test interface:
   - `CatchProto.data.MOD`; `giveCharm(id, weightless, bypass)` takes the new ids; `state().shelf[i].counter`.
   - `CatchProto.price(kind, opts)`: the price of one fish of `kind` in the present run state; `opts` may give `chain`, `zone`, `hookIndex`, `pullSize`, `bait`, `hung` (seconds), `timeLeft`, `rod` (index). It must not change any state (no counters, no streak).
   - `CatchProto.selfTest()` keeps the money checks and adds three groups:
     a. Vocabulary: every trigger, condition, effect and count id that a charm uses has a handler; every rule flag is known.
     b. Price order: at least 12 cases with the expected number worked out by hand from the rows and the combine order (single charm; flat with multiplier; two percents; a chain with and without Food Chain; a card level; the Golden Grub; rounding at .5). List the cases and the hand arithmetic in the report.
     c. No ground, no pay: for each charm with `"unconditional": false`, in a fresh run (day 1 before the start: one rod of level 1 on Worm, no chum, every card at level 1, one open zone, 0 coins, the shelf holds only this charm, its counter at the start) nothing differs from an empty shelf: the price of each of the 14 kinds (Sunlight hook, hook 1, chain 0, pull of 1, 30 s left), the wanted count of each kind, the bite rate and the hunter factor, the day length, the shelf slots, the coins of a day that ends exactly at the fee, and every rule flag. Return the ids that differ in `failed`, with what differed.
     The self-test saves and restores the running game.
6. Write `preview-modifiers.html` (the first line of `preview.html`, then the whole page) and `modifiers-probe-report.md`.

## 5. Done means

1. `catch-modifiers.html` loads with no console error; a run can be played from day 1 through at least three shops with charms bought and sold.
2. `CatchProto.selfTest().ok` is true. If group c fails for a charm and you believe the data is at fault (the charm really pays without ground), do not bend the code: leave the check failing for that id, set `ok` by the other checks only, and list the id under "design findings" in the report.
3. `node --check` passes on the extracted script.
4. `python embed_mod_data.py` run twice in a row leaves the page byte-identical (the embedding is stable).
5. Unchanged, byte for byte, with SHA-256 before and after: `catch-economy.html`, `preview-economy.html`, `catch-progression.html`, `catch-prototype.html`, `preview.html`, `modifiers-v2.data.json`, `modifiers-v2-proposal.md`.
6. No charm id is hard-coded in game logic outside the embedded data and the flag code. Give the grep you used to check it.

## 6. Boundaries

- Write only in the prototype folder, and only the four new files of section 4. Change nothing under `F:\UE`.
- Do not change the JSON. If a row cannot be implemented as written, or two rows contradict each other, implement the plainest reading and list it in the report under "data questions". The orchestrator corrects the data.
- You do not invent the game. No new charm, number, rule or screen. A thing that neither the data nor this brief gives: plainest reading, listed in the report.
- No network, no packages, no build tools; the page stays one self-contained file (the font link stays as it is).
- Do not publish the page. No git.

## 7. Report

English, `modifiers-probe-report.md`, at most 60 lines: the evaluator in ten lines (where each trigger is called from); the self-test result with the 12 price cases; design findings from group c; data questions; readings; the hashes; what you could not verify without a browser.

## 8. Chain of command

You report to the orchestrator. The orchestrator loads the page in a browser, checks prices against hand arithmetic through `CatchProto.price`, plays three days, and then gives the page to the owner.

## 9. State at hand-over (12.11, 14:50): the executor is now Opus

Codex started this task and hit its usage limit after about three minutes. The project contract gives the task to Opus. What Codex left in the prototype folder (never run, unverified):

- `catch-modifiers.html`, 278,820 bytes. The JSON is embedded between the markers (lines 193 to 2948). `var CHARMS` and all 36 `has("...")` calls are gone. About 100 lines of a row evaluator follow the data (`RULE_FLAGS`, `focus`, `hookFocus`, `COUNTS`, `CONDITIONS`, `TRIGGERS`), and `ECON.rule3Shallow` exists. Outside the data block the page differs from `catch-economy.html` by about 200 lines. `CatchProto.price`, the new self-test groups, the cards, the log additions and the rule switch were not found.
- Known damage: the Russian lines of the "what is new" box are question marks. Codex wrote them through a shell that lost the encoding. Any other Russian text that Codex wrote may be damaged in the same way. Search the page for runs of `??` and restore every text from section 3 of this brief.
- `embed_mod_data.py`, 1,000 bytes. It reads complete. Check it against "done" item 4.
- `preview-modifiers.html` and `modifiers-probe-report.md` do not exist.
- Codex left one note on the data: `cap` is described both as the limit of a counter and as the limit of the total effect; for Slow Hands "up to +300%" is a limit of the effect; Groundbait fires when the shop opens, as its row says. Treat these as readings and list them in the report.
- The seven protected files are intact. First 16 hex digits of SHA-256, taken by the orchestrator at 14:49: `catch-economy.html` 7efcd6133ef5aca3, `preview-economy.html` 2a511f713cfdbeb6, `catch-progression.html` a3ceee0f85e313ba, `catch-prototype.html` c6848b24cbce2e6b, `preview.html` 9aa16d6699eba37f, `modifiers-v2.data.json` 28c143db625bd4af, `modifiers-v2-proposal.md` 2b829c094d61c839.

Your choice: continue from the partial page, or start again from a copy of `catch-economy.html`. The orchestrator's advice: read the difference first (the page without the data block against `catch-economy.html`). Continue if the evaluator reads sound. Otherwise start again and keep `embed_mod_data.py`. Say in the report which you did.

Two working rules:

- Write files with a tool that keeps UTF-8. Do not pass Russian text through shell redirection.
- Do not open a browser: the orchestrator holds the only preview server. For checks use `node --check` and a Node run of the page script with a stubbed `window`, `document` and canvas context, as the money probe did. Say in the report what such a run cannot prove.
