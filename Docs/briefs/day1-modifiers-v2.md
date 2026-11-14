# Brief: modifiers v2 — a charm pays only on ground that the player prepared

Status: written on 12.11.2026 at 14:15 (owner's clock). Executor: Claude Opus, effort high. This is design work on paper. No code, no prototype, no Unreal.

## 1. Goal

The owner (the game director) played the progression probe and rejected the present set of 20 modifiers ("charms"). The owner's words (Russian, verbatim, dictated by voice):

> «Сейчас механики работают все и сразу, а надо чтобы они срабатывали если игрок подготовил почву. Например в балатро есть джокер кота который работает на удачу. но для начала игрок должен добавить счастливые карты в колоду. Тоже самое должно быть и у нас. Игрок меняет состав водоёма, ценность рыб, наживки, удочки и т.п. и ему нужно делать выбор что взять, и собрать комбинацию должно быть трудно потому что модификаторов очень много. Кроме того хотелось бы больше синергий наших улучшений.»

> «мы сейчас просто улучшаем ценность рыбы, а могли бы ещё за деньги увеличивать количество рыб (прикормка) определённого типа.»

> «я не считаю что мы должны ограничивать игрока в покупках до 3го дня. А вот сделать там чтобы денег у него хватало в притык нужно.»

Recorded decisions (in `Docs/decisions/game-frame.answers.json`): `prepare-the-ground`, `chum-raises-fish-count`, `no-day-locks-money-tight`, `money-like-balatro`, `probe-both-verdict`. "Джокер кота" is the Lucky Cat joker of Balatro: it does nothing until the deck holds Lucky cards.

Write a proposal for a new modifier list built on that principle. The orchestrator critiques it, a browser probe is built from your data file, the owner plays it and decides. So the proposal must be concrete enough to implement without a single further question.

## 2. What exists (verified by the orchestrator; read these first)

- `CLAUDE.md` §2: the game in short. A fishing game, real-time day, the player baits rods, moves tackle and pulls. No reaction-test bite. Chains: a hunter eats the hooked fish and gets hooked itself.
- `Docs/decisions/game-frame.answers.json`: every owner decision (77 answers). The last six are the ones above.
- `Docs/design/rules-reference.md` (English, exact rules): §2 sea and populations ("wanted count"), §3 rod, §4 bait, §5 bite, hang timer and hunters, §6 pull, §7 price formula (the order of steps), §9 baits, §10 shop, §11 the 20 charms with exact effects, §13 special fish and day rules.
- `Docs/design/content-list.md` (Russian): §7 fish table, §8 baits and packs, §11 upgrades, §12 charms by class and the "paths of play" table, §13 fee.
- `Docs/design/core-interface.md`: how the rules core exposes commands, state and events. The core is plain C++ with data tables.
- `Docs/design/progression-proposal.md`: the earlier pacing analysis (bot runs).
- `Docs/briefs/day1-economy-probe.md` §3: the money frame that a parallel probe is building right now. Take it as given: the catch value of the day pays the fee; coins are a separate small payout (about 4–11 per day); coin prices are small (a charm costs 4–8 coins); three fee curves that start at 25 and grow x1.6, x2, x2.5 per day; no day locks; chum raises the count of one species by 50% per level.

Facts the orchestrator found:

1. Fish prices by zone: Sprat 2, Mackerel 6, Sea Bass 15 | Lanternfish 9, Cod 28, Tuna 70 | Hatchetfish 20, Anglerfish 130 | Oarfish 320, Gulper Eel 220 | Snailfish 90, Giant Squid 700 | Flounder 150, Coelacanth 1500. The step is about x4–5 per zone, and one purchase (Deeper Line) buys the whole step. Depth dominates every other choice.
2. Twelve of the 20 charms pay with no condition (examples: Sprat Fever, Deep Pressure, Giants, Dusk Frenzy, Long Day). Any charm is good in any run, so there is nothing to assemble.
3. The bot predicted money at 3.4 times the fee on day 5. The owner reached 28 times ($8,500 against $300). A human finds the dominant line fast.
4. The owner wants the player to be able to win near the surface too (decision in `CLAUDE.md` §2: modifiers raise the income of surface fish).

## 3. Design frame (decisions of the orchestrator; revocable — if one breaks the design, change it and say so)

1. **Ground and payoff.** Every purchase is one of two kinds. Ground: it changes a countable thing in the run. Payoff: a charm whose effect scales with, or switches on at, a countable thing. A payoff charm with no ground does nothing or next to nothing. At most 20% of the charms may pay without a condition, and those must be weak or have a real cost.
2. **Countable things ("ground") that already exist or are already decided.** Per species: chum level (count), fish card level (value). Bait: stock by kind and by rarity, kinds bought this run. Tackle: number of rods, rod level, hooks per rod, open depth zones, depth of each rod. Shelf: charms by class, empty slots, charm neighbours. Day: time left, day number, special-fish day. Catch: fish landed today by species, by zone, shoal or hunter, chain length, fish per pull, streaks. Money: coins held. You may propose at most one new kind of ground if the list above cannot carry enough builds; mark it as an owner question.
3. **A small set of primitives.** The core must implement the list as data in one day. Define a closed vocabulary: triggers (for example: on landing, on pull, on chain link, on day start, on day end, on shop enter, on purchase, passive), conditions over the countable things, and effects (flat value, additive percent, multiplier, count change, timer change, coins, bait, slot change, counter that grows and is remembered by the charm). Aim at no more than about 10 triggers, 15 condition kinds and 15 effect kinds. Every charm is a row of these. A charm that needs its own special code is allowed only if it is one of at most 5 and you say why it is worth it.
4. **How values combine.** Fix one order: base price, then flat additions, then the sum of additive percents, then multipliers. State it in one paragraph and show where it differs from `rules-reference.md` §7. Decide whether shelf order matters (left to right). The default is: it does not, because reordering costs interface work. Argue if you change that.
5. **Many charms, hard to assemble.** 45 to 50 charms in the four classes that the owner fixed (rod, bait, fish, sea), roughly even. Three rarities with shop weights; coin prices 4, 6, 8 by rarity (the Weightless variant stays: x1.5, takes no slot). The shop shows three charms, the shelf has five slots, so a build of three or four linked charms must take several days of looking, rerolling and preparing ground.
6. **Kinds of charm to cover** (structure only; names, numbers and content must be our own, from the sea and from fishing): a charm that switches on at a threshold of ground; a charm that scales with a count; a charm that grows a remembered counter from repeated actions in the run; a charm that pays coins instead of catch value; a charm that trades one thing away for another; a charm that changes the number of shelf slots; a charm that rewards a narrow sea (few species) and one that rewards a wide sea; charms that make the bare hook, the surface, a single rod, many rods, long chains, and waiting on the hook each a real line of play.
7. **Depth must stop being the only answer.** Propose the smallest change that does it: a flatter price ladder, or a dearer Deeper Line in coins, or strong surface payoffs, or a mix. Give the new fish price table if you change it. A surface build with good ground must be able to pay the Normal fee on day 9.
8. **Ground purchases.** Fix what the shop sells as ground and at what coin price: chum (given: 3 coins, +1 per level, +50% count per level, at most 5), fish card (given: 3 coins, +1 per level; state what one level gives), and anything else you need (for example a second chum offer, a bait crate of one kind). Keep it to what a five-tile shop row can show.
9. **Old charms.** For each of the 20 present charms say: kept as is, reworked (how), or dropped (why). Nothing is deleted from the documents; this is a proposal.
10. **Words.** The game is in English. Each charm has an English name (at most 14 characters, no brands, no film or director words, age 13+) and an English rule line of at most 90 characters that a player understands with no other text. The probe for the owner is in Russian, so give a Russian name and a Russian rule line too (same limits, short plain sentences). A charm with a counter shows it, for example "(now +12%)".
11. **Not Balatro.** We take the structure: ground first, payoff second. Do not copy its jokers, names, numbers or wording. Do not bring cards, poker or luck-of-the-draw scoring into the game. The theme stays the sea.

## 4. Deliverables (two files, nothing else)

1. `Docs/design/modifiers-v2-proposal.md`, English, in this order:
   1. Principles in ten lines, with the owner's words quoted.
   2. The ground table: each countable thing, how the player raises it, its coin price, its cap.
   3. The primitive vocabulary: triggers, conditions, effects, each with one line of meaning, and the combine order.
   4. The charm table, 45–50 rows: id, English name, class, rarity, coin price, English rule line, ground it needs, what it combines with (ids), kind (from frame point 6), "no condition" flag.
   5. Eight to ten builds. For each: the ground to buy, the three to five charms, why it is hard to assemble, what it gives up. One must be a surface build, one a bare-hook or cheap-fish build, one a chain build, one a single-rod build, one a many-rods build, one a coin (economy) build.
   6. Arithmetic for four builds on days 3, 6 and 9 against the Normal fee (100, 800, 6400): the expected catch value with your stated assumptions (fish per minute, day length from the rules reference, levels bought with the coins the payout gives). Hand arithmetic in a table; no simulation. Also one line each: what a player with no build earns on those days. The aim: a prepared build clears the fee with a margin of 1.2–2 times; no build fails by day 5 or 6; nothing reaches 10 times the fee.
   7. The fate of the 20 old charms.
   8. Changes to prices, fish table, upgrades and shop that the list needs (frame points 7 and 8).
   9. Your own critique: the three weakest points of this proposal, and two counter-options that could work better (for example: 30 charms with levels instead of 50; species traits bought with chum as a second kind of ground), each with pros, cons and cost, and your conclusion.
   10. At most five questions for the owner, director-level only (what the player feels, what the game is), each with options and your recommendation first.
2. `Docs/design/modifiers-v2.data.json`: the same list as data, so that the probe and later the core are built from it without retyping. Valid JSON, UTF-8, LF. Shape:
   - `"version"`, `"combineOrder"`, `"rarities"` (weight, price), `"ground"` (rows of table 2), `"primitives"` (`triggers`, `conditions`, `effects` with their parameter names),
   - `"charms"`: one object per charm: `id` (snake_case), `nameEn`, `nameRu`, `class` (`rod` | `bait` | `fish` | `sea`), `rarity`, `price`, `ruleEn`, `ruleRu`, `unconditional` (true or false), `needs` (ground ids), `combos` (charm ids), `rows`: a list of `{ "trigger", "conditions": [...], "effect": { "kind", ...params } }`, and `counter` (start, step, cap) where the charm remembers a number,
   - `"builds"` (name, ground, charm ids), `"fishPrices"` only if you change them, `"upgrades"` only if you change them.

## 5. Done means

1. Both files exist; the JSON parses (`python -X utf8 -c "import json,io;json.load(io.open('Docs/design/modifiers-v2.data.json',encoding='utf-8'))"`).
2. 45–50 charms; each class has 10–14; at most 20% carry `"unconditional": true`.
3. Every `trigger`, condition kind and effect kind used in a charm is declared in `primitives`. Every id in `needs` and `combos` exists. Check it with a short script and give the script's output in section 9 of the proposal.
4. No English rule line is longer than 90 characters and no English name longer than 14; same check, same script.
5. The table in the `.md` and the JSON agree (same ids, same prices, same rule lines).
6. Every number that you propose is marked provisional. Every rule that is not covered by a recorded owner decision is listed in section 10 or marked "orchestrator frame".

## 6. Boundaries

- Write only the two files of section 4. Scratch scripts go to your temp folder, not into the repository.
- Do not edit `content-list.md`, `rules-reference.md`, `core-interface.md`, the decisions file, the journal, `DEVLOG.md`, any source code or any prototype file. Two other executors are working in `Catch/Source` and `Catch/Content` right now.
- No git commands that change anything. No commits.
- Do not ask the owner anything. Questions go into section 10 of the proposal; the orchestrator answers the technical ones and carries the director-level ones to the owner.
- No network is needed. Do not quote or paraphrase the content of another game beyond the one structural example that the owner gave.

## 7. Report

Your final message to the orchestrator, English, at most 25 lines: the count of charms by class and by rarity, the share without condition, the eight to ten builds by name, the change to the price ladder in one line, the three weakest points, the questions for the owner, and anything in the frame of section 3 that you changed.

## 8. Chain of command

You report to the orchestrator. The orchestrator reads the proposal, runs the checks of section 5, writes a critique, and has a probe built from the JSON for the owner to play.
