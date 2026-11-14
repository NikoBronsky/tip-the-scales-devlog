# Modifiers probe: report (Opus, 12.11)
Pass 1 started from a fresh copy of `catch-economy.html`. I did not continue the Codex partial page (old ids, a syntax error, broken Cyrillic). `embed_mod_data.py` is the Codex script; pass 2 changed only its source file name.

## Pass 1
### Evaluator: one for all 50 charms
1. A charm is a list of rows: trigger + conditions (all must hold) + one effect. Handler tables: `TRIGGER_AT`, `CONDITIONS`, `EFFECTS`, `COUNTS`, `FLAG_CODE`.
2. `evaluate(trigger, focus, only)` reads every row of every shelf entry into one sum. `commit()` applies coins, chum, counters and the chip tags.
3. A focus is `{kind, rod, zone, hookIndex, chain, pullSize, bait, hung, timeLeft, streak, item}`, built by `focus()` / `hookFocus()`.
4. `passive`: `wantCount`, `attract`, `hunterMul`, `eatReach` (pass 2), `dayLength`, `shelfCap`, `packExtra`, `flagHits`. `on_land`: the score function (each landing, pull preview, hover card, award table, `CatchProto.price/weigh`).
5. `on_pull`: `land()`. `on_escape`: `escapeHook()`. `on_chain_link`: `eat()`. `on_day_end`: `payFee()` after a paid day. `on_shop_enter`: `openShop()`. `on_purchase`: `bought()`.
6. Flag code: auto_reel `updRod`, no_escape `updHook`, cluster `updHook`/`updFish`, bait_knife score function, giant_size `spawnFish`, low_tide `effZone`, no_day_rule `activeRule`.
7. Scaled effect: n = max(0, floor(source / perStep) − minus); value × n; `cap` limits the effect. A scaled `mult` is ×(1 + value × n).
8. Orchestrator note, three places: score in one function (pass 1 `priceOf`, pass 2 `scoreOf`); eat test in one function (`canEat`); money words in format helpers (pass 1 `money()`, pass 2 `kgText`, `ptsShort`, `ptsFull`, `AWARD_TAG`). Charm texts come from data only (`nameRu`, `charmRule()`, `needsText()`).

### Pass-1 results
- Self-test `ok: true`: money checks, per-charm prices, vocabulary, 27 price cases, group c (44 conditional charms) all pass.
- Group c can fail: with deep_reel's `catch_zone` removed at runtime the self-test gives `ok: false` ("price sprat 2 -> 4 …"); restored data gives `ok: true`.
- Design observation: a chain build (Food Chain, Apex, Slow Hands, Big Haul) reached 24× the day-4 fee. Chain builds stack steeply.

### Pass-1 data questions (plainest reading implemented)
1. `purchase.item` has no `tin` or `shelf`; the page sends them and no row matches.
2. Chum level = bought level `G.chum[kind]` (0–5); `ground_add` stops at `ECON.chum.max`.
3. `cap` limits the effect value, not the count.
4. Groundbait fires on `on_shop_enter`; its kind is the most-landed kind of the day, Sunfish excluded (also for `today_top`).
5. `on_day_end` fires only after a paid day; it sees the coins after the payout.
6. Sunfish and Bait Knife Sprat: score rows do nothing; coin and counter rows still apply.
7. `rods_same_bait` is false when rod 1 has no bait. `hook_bait` reads the bait on the hook now.
8. `{n}` ignores the row conditions, uses rod 1 and the species of the row's `fish` condition.
9. Lost shelf slots keep the charms; the page draws max(cap, used) slots.
10. `pull_size` counts every fish on the rod at the pull, cluster joiners and chain fish included.

### Pass-1 readings (revocable)
- Chain = fish eaten on the hook (`h.chain`). The pop says «ЦЕПОЧКА x(chain+1)». The log counts pulls with fish.
- Cluster takes only fish of the hooked kind, up to 4. A Weightless offer says «БЕЗ МЕСТА». A switched-off day rule says «Отключено модификатором.».
- A shelf chip shows `nameRu`; `FLAG_TAG` holds short pop labels for flag rows only.
- Charm id check (`modprobe-work\idgrep.py`): ids appear only in the self-test cases and as rule-flag names in the flag code.

## Pass 2 (modifiers v2.1: weight, points, scales, ladder)
**What changed.** The page reads `modifiers-v2.1.data.json`. `scoreOf(kind, focus, apply)` follows combineOrder 1–9 and returns `{p, weight, award}`; `CatchProto.price` returns the points, the new `CatchProto.weigh` returns `{weight, award, points}`; neither changes state. No «$» is drawn: weight «3,5 кг», points «оч.» or «1 очко / 2 очка / 5 очков». «Плата» is now «норма дня». The fee bar is gone; scales stand on the left end of the raft (raft extended left; rods not moved), with small scales in the top panel when the raft is out of view. A «Таблица наград» button in prep, the day summary and the shop opens a modal, scrollable award table. The hover card shows weight, award, points now, size, «берёт на крючке», «её берут». `canEat(eater, preySize, hookCtx)` is the one ladder test; Live Sprat bites follow it. The run log has no dollars.

**Self-test** (Node): `ok: true`, `failed: []`, 31 price cases, 16 ladder cases, group c passes with the new `reach` snapshot. Mutations prove group c can fail: deep_reel without `catch_zone` (price changes) and big_mouth without `ground_level` ("reach sprat 1 -> 2 …"). New price cases (hand arithmetic = page):
- Whetstone, rod lvl 2, Mackerel card 3: (5+4)×3 = **27** (flat inside a card level) · Blue Ribbon, card 4: 5×(4×2) = **40**; card 3: **15**
- Sea Bass chain 2: 12×2 = **24**; with Food Chain 12×3 = **36**; Food Chain chain 1: **18** · Big Haul pull 2, Mackerel card 3: 5.5 kg×3 = 16.5 → **17**
- Hatchetfish chain 1: 10.5 → **11** · Rule 4: Grub Mackerel 10.0 kg → **10**; Mackerel chain 1 7.5 kg → **0**; Small Fry Mackerel 9 kg → **0** · Sunfish day 3: **50**
- Ladder: Sprat hooked → Mackerel yes, Sea Bass no, Tuna no; Mackerel → Sea Bass yes, Mackerel no; Big Mouth + chummed Sea Bass → takes a Sprat; no chum → no; chummed Sprat (hook side) → no; Free Feast + Spoon (Rare) → Giant Squid takes a Sprat; + Worm → no; hooked Sunfish → nobody; Sunfish never eats; Live Sprat → Mackerel bites, Tuna not, chummed Sea Bass with Big Mouth bites; day 1 nobody eats, day 2 on.

**Node run** (`modprobe-work\harness.js`, `play.js`): run 3 played 4 days and 4 shops, a charm bought and sold, 23 two-link chains Sprat → Mackerel → Sea Bass; the eats seen were only Sprat>Mackerel and Mackerel>Bass. A chain run saw `["sprat","mackerel","bass"]`. Award table opened from prep, summary and shop, Escape closed it, wheel scrolled it (168 px with all zones), the foot line counted 11 locked kinds. Scales: the first crossing fired at r = 1.00; end-of-day angle 10.00° = rule. 1191 distinct drawn strings, none with «$»; no «$» in any run log; search `[$]` over the code outside the data block: 0 lines. `node --check` passes. Embed run twice: catch-modifiers.html 2c961672…ed77f1f, preview-modifiers.html 4970b900…78bd8 both times. All 9 protected files unchanged (SHA-256 before = after).

**Data questions** (plainest reading implemented)
1. The final weight is rounded to 0.1 kg, and that value goes to rule 4 and to the points (9.96 kg counts as 10.0).
2. Rule 4 Russian text is my translation of `dayRules[1].new`: «Рыба легче 10 кг не даёт очков.»
3. «Points of a plain fish now» (table, card) = rod 1 with its bait, home zone, no chain. «берёт / её берут» use that hook among open-zone kinds.
4. The `hunter` flag still sets the arrival day (`swimsFrom`, money probe) besides the wide band; data says "only the wide band and the hover note".
5. Free Feast + a Live Sprat hook: Live Sprat is Rare, so every bigger kind bites it (data `liveSprat` says so); the ground check uses rod 1's own bait.

**Design findings.** Group c: no charm pays without ground. Observation: plain worm play fell short on day 4 (104 vs 200 points); the chain run reached 1158 vs 200 (Sea Bass 56,2 kg). Chain builds still stack steeply.

**Readings (revocable).** Shipping text is «ПОДСЧЁТ УЛОВА»; coin particles stay. Eat tag «АТАКИ x». `tryPredator` stops before `huntersIn()`. Prep table button sits above «НАЧАТЬ ДЕНЬ».

**Not proven by Node.** The look of the scales and the award table (layout, text fit: `measureText` is stubbed, fonts), the spring feel, the chime and weight sound, pointer feel, real frame timing, the clipboard copy. **Not done:** no browser check of any frame.
