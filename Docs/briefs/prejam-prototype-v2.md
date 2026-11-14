# Brief: browser prototype, second version (pre-jam, throwaway)

Date: 2026-11-12, 02:10 (owner's clock, UTC+1). Written by the orchestrator. Executor: Opus, effort high.

## 1. Goal

Rebuild the throwaway browser prototype of the fishing game so that it shows the owner's feedback on version 1 and the orchestrator's content list. The owner (game director) plays it before the jam starts at 06:00 and decides director-level questions from what he feels.

The owner's words after playing version 1: «выглядит очень здорово, мне очень понравилось» and «В общем, надо эти моменты продумать, улучшить». His points are recorded in `F:\UE\EpicJam\Docs\decisions\game-frame.answers.json` (ids `bait-packs`, `start-one-rod`, `modifiers`, `pacing`, `day-start`, `bait-loading`, `rods-visible`, `depth-unlock`, `feedback-sounds`, `hooked-fish`).

This prototype is a decision aid. Nothing from it goes into the Unreal game. It lives outside the repository.

## 2. What exists now (verified by the orchestrator)

- Version 1: `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\catch-prototype.html`. 1149 lines, about 71 KB. A backup copy `catch-prototype.v1.html` lies next to it. Do not edit the backup.
- The file is an HTML fragment for a claude.ai artifact: no doctype, no `<html>`, `<head>` or `<body>`. It starts with `<title>`, a `<meta name="claude:viewport-fit-sides" content="cover">`, a `<style>` block with theme tokens on `:root` plus dark-mode blocks, then the page markup, then one `<script>` with one IIFE (line 97 to the end).
- Page chrome is Russian (header note, three boxes under the canvas, "Как играть"). All text drawn inside the canvas is English.
- Canvas `#game`, logical size 1280x720. Constants at lines 102-141: `W, H, SEA_L, SEA_R, SURF_Y=210, DX, DY, WORLD_D=1500, SLOT_X=[600,440,760,280,920], DAY_LEN=50`, tables `FISH`, `BAITS`, `RULES`, `UPG`, `WATER`.
- State object `G`, settings `SET = { variant, winMark, sound }`, audio helpers `tone()` and `hiss()` (lines 183-232, synthesised WebAudio, no files).
- Rod logic: `attract`, `hookFish`, `bite`, `tryPredator`, `eat`, `escape`, `pullRod`, `priceOf`, `land`, `updRod` (lines 345-450). Day flow: `startDay`, `endDay`, `openSummary`, `payFee`, `openShop`, `buyCard`, `buyPack`, `nextDay`, `update` (lines 452-582). Drawing: lines 584-1031. Input: lines 1033-1122 (`pt`, `rodAt`, `dropRod`, `giveBait`, `gaugeCam`, pointer handlers). Frame loop: lines 1134-1148.
- Debug hook used by tests and by the orchestrator's browser checks: `window.CatchProto = { state, settings: SET, endDay, step(dt), draw }`.
- Headless smoke test: `...\scratchpad\proto-test\smoke.js` (88 lines, Node 24). It stubs canvas and DOM, loads the script and lets a bot play ten days. It passes on version 1.
- The design source for version 2: `F:\UE\EpicJam\Docs\design\content-list.md`. Its prose is Russian; every in-game name, rule text and number in it is ready to use. Sections 3 to 11 are the specification of data and rules. Read it first.

## 3. Technical decisions (each one is revocable; say so in the report if you change one)

1. Edit `catch-prototype.html` in place. Keep the artifact fragment format, the `<title>`, the meta tag, the theme tokens, the Russian chrome style and the single IIFE. One file, no external resources except the Google Font link that is already there.
2. Keep `window.CatchProto` and extend it: `state`, `settings`, `step(dt)`, `draw`, `endDay`, plus `startDay()`, `gotoShop()`, `openPack(kind)`, `giveCharm(id)`, `addMoney(n)`, `unlockZone(n)`. The orchestrator uses them to capture frames of each screen.
3. Phases: `intro -> prep -> day -> dayEnd -> summary -> shop -> prep -> ...`, plus `over` and `win`. Opening a bait pack is a modal sub-state that works in `prep` and in `shop`.
4. The hooked-fish rule is fixed now: a hooked fish escapes when its timer ends. Remove the three-variant radio group from the page chrome and from `SET`. The predator chain survives only as the charm "Live Bait". Keep the `opt-win` and `opt-sound` checkboxes and the `btn-skip` and `btn-new` buttons.
5. A rod owns: level (1-5), bait type, target depth, and an array of hooks. Each hook has its own fish and escape timer. Extra hooks sit 80 px above each other, counted up from the deepest hook. One pull lands every hooked fish of that rod.
6. Bait stock is global per bait type. A rod is assigned a bait type and takes one portion per hook per cast.
7. Charms are data: `{ id, cls, name, price, text }` plus small checks at the hook points listed in content-list section 12. No charm needs its own subsystem.
8. Sounds stay synthesised with WebAudio. No audio files.

## 4. Work, in priority order

Finish the blocks in this order. If time runs short, stop after a complete block and report what is missing.

### Block 1. One rod, preparation, start button, pace (decisions `start-one-rod`, `day-start`, `pacing`)

- A run starts with one rod of level 1 on the centre slot, 20 Worm and one free Small Pack.
- New phase `prep`: time stands still. The player loads bait and drags hooks to a depth. A large canvas button `START DAY` (also the Space key) starts the day. Before the first day show two short English hint lines, for example "Put a bait on your rod." and "Press START DAY.". The button pulses only when at least one rod has bait.
- On start the rods cast one after another, 0.15 s apart.
- Rod cycle times from content-list section 4. A rod repeats its cycle by itself while its bait is in stock.
- Each rod shows its state as a word above it: `CAST`, `SINK`, `WAIT`, `FISH ON`, `PULL`, `NO BAIT`.
- Fish density from section 4. A matching fish near a baited hook visibly turns and swims to it. The wait for a bite is 1 to 3 seconds when a matching fish is in the same zone.

### Block 2. Bait packs, nine baits, labels, loading (decisions `bait-packs`, `bait-loading`)

- Nine baits and eight fish from content-list sections 6 and 7. A bait that needs a locked zone does not appear in packs.
- Shop sells two packs per day (Small Pack, Big Pack) and the Worm Tin. Opening a pack: the packet tears, the bait cards fan out, the player picks one (Small) or two (Big). Each bait card shows its name, portions and a line `Catches: Sprat, Mackerel`. Special property goes on a second line.
- Hovering a bait packet in the tray shows the same card as a tooltip.
- Loading: click or press a packet to take it in hand. It follows the pointer with spring and tilt, as in version 1. While a packet is in hand, every rod shows a large glowing socket at its base on the raft and in the rod strip (block 3). Within about 90 px the socket pulls the packet (magnet). Releasing the button over a socket, or clicking a socket, assigns the bait. Both drag and click-then-click must work. A click on empty water puts the packet back.
- The socket of a rod always shows the icon of its bait and the portions left in stock.

### Block 3. Rod strip (decision `rods-visible`)

- When the camera has scrolled so far that the raft is off screen, a strip of rod cards stays at the top of the canvas under the HUD. One card per rod: level, bait icon and portions, state word, escape ring when a fish hangs. The fishing lines run from the cards down to the hooks.
- A click on a card pulls that rod. A card is also a bait socket.

### Block 4. Upgrades and depth zones (decisions `incremental`, `depth-unlock`)

- Shop section "Rods": one tile per owned rod with `SHARPEN $N` (level 2 to 5; each level makes the whole rod cycle 20% faster; levels 3 and 5 add a hook) and one tile `NEW ROD $N` while fewer than five rods exist. Prices from content-list section 8.
- `DEEPER LINE`: opens Twilight, then Abyss. Prices from section 8.
- Three zones from section 5. A locked zone is visible but darker, with dim silhouettes of its fish and a centred label, for example `TWILIGHT - LOCKED` and `Buy Deeper Line`. Hooks cannot go below the unlocked depth. The camera can still scroll down to look. When a zone opens, the dark veil slides down with a low rumble.
- Remove the version 1 upgrades that the content list does not contain.

### Block 5. Charms (decision `modifiers`, `hooked-fish`)

- Twenty charms in four classes from content-list section 9, with the exact English rule texts. Minimum for this prototype: all five Rod charms, Bait Knife, Thrifty Hook, Chum Bucket, Live Bait, Sprat Fever, Giants, Same Again, Big Shoals, Low Tide, Long Day, Dusk Frenzy. Add the remaining four if they are cheap.
- Shelf of five slots, always visible in the HUD during the day and in the shop. Each charm is a small card with a class colour (Rod red, Bait green, Fish blue, Sea teal). Hover shows name, class and rule. A charm wobbles and shows its bonus when it triggers.
- Shop offers three random charms the player does not own. `REROLL $3`, plus 2 for each further reroll in the same shop. With a full shelf the buy button is disabled and says `SHELF FULL`. A click on an owned charm in the shop offers `SELL $N` at half price.
- "Haul": two or more fish landed within one second. Show `HAUL x3` near the money. It pays a bonus only with the charm Big Haul.

### Block 6. Feedback on every action (decisions `feel-first`, `feedback-sounds`)

- Implement the event table of content-list section 11: motion, sound and effect per event. The owner named these sounds explicitly, so they are mandatory: splash when the hook enters the water, line tension while a fish hangs (a creak or tone that rises as the escape comes closer), a reel ratchet that lasts as long as the pull lasts, a snap when bait goes on a rod, a pleasant coin sound when a fish is collected (pitch rises with each fish in a row).
- Keep every sound short and soft. The master volume must not clip when five rods act together.

### Block 7. Fee, special fish, page chrome

- Fee formula and the five day rules from content-list section 10. The shop shows the rule of the next day when it is a special day.
- Page chrome under the canvas, in Russian, short sentences (at most 20 words, active voice, no jargon): update "Как играть" and add a box "Что нового во второй пробе" that lists the changes in seven to nine short lines. Remove the "рыба на крючке" variant box. Keep the note that all names and numbers are temporary.

## 5. Done means

1. A new run starts with one rod, a free pack to open, and the `START DAY` button. Nothing fishes before the press.
2. A player can finish day 1 with only the mouse and without reading the page chrome.
3. Bait comes from packs with a choice. Every bait shows which fish it catches.
4. With the camera at the bottom of the sea the player still sees every rod's state and can pull and re-bait from the strip.
5. Twilight and Abyss are locked at the start, visible as a tease, and open through `DEEPER LINE`.
6. The shelf holds at most five charms. Buying, selling and reroll work. At least sixteen charms change the game as their text says.
7. Every event of the feel table gives a visible and an audible answer.
8. `node smoke.js` passes: the bot plays twelve days through prep, packs, upgrades, charms, zone unlocks, special days, loss and a new run, with every implemented charm active at least once, without an exception and without NaN in money, timers or positions.
9. No Cyrillic text is drawn inside the canvas. No layout overlap in the HUD, the strip, the shelf, the shop or the pack overlay at 1280x720.

## 6. Verification

```bash
node "C:/Users/<user>/AppData/Local/Temp/claude/c6c51a23e224438d/5d6e7386-f38f-41ac-a6c9-a74df26bf541/scratchpad/proto-test/smoke.js"
```

Update `smoke.js` for the new phases and helpers. Keep its style: stubbed canvas and DOM, captured listeners, a bot that plays through the public debug hook and synthetic pointer events. Add a syntax check of the extracted script (`node --check` on a temp copy).

You cannot see the canvas. Do not open a browser. The orchestrator captures and judges the frames. Compute text widths with care: use `ctx.measureText` for anything that sits next to other text, and keep shop tiles on a fixed grid.

## 7. Boundaries

- Write only inside `...\scratchpad\catch-prototype\` and `...\scratchpad\proto-test\`. The repository `F:\UE\EpicJam` is read-only for you. No git commands. Do not publish the artifact. Do not start servers or browsers. No network.
- Do not invent rules, fish, baits or charms beyond the content list. Where the list leaves a gap, take the simplest reading and list it in the report under "Placeholders I chose".
- Do not copy a specific existing game: no names, texts or art from other games.
- Shell note: Bash heredocs that contain Cyrillic or apostrophes fail in this harness. Change files with the Edit and Write tools. Read the big file in chunks.

## 8. Report

Reply in English, at most 40 lines:

1. Blocks finished, blocks partly finished, what is missing.
2. Placeholders you chose where the content list had a gap.
3. Technical decisions from section 3 that you changed, and why.
4. The last lines of the smoke-test output.
5. The debug helpers on `window.CatchProto` with their arguments.
6. Places where you expect a visual problem that only a frame can confirm.

## 9. Chain of command

You report to the orchestrator, not to the owner. Put open questions into the report. Do not wait for answers: choose the simplest option and continue.
