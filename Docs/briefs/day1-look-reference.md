# Brief: day one — the look reference (what the approved prototype looks like)

Status: written and handed out on 12.11.2026 at 10:50 (owner's clock). Executor: Opus, effort high. Document only.

## 1. Goal

The owner compared the Unreal build with the browser prototype and the Unreal build lost badly. His words:

- «Сейчас, если честно, все выглядит очень плохо. Насколько это супер круто выглядело в нашем артефакте, настолько сейчас все ужасно выглядит.»
- «Из хорошего у нас глубина. Но опять-таки шейдеры могли быть лучше, сильно лучше.»
- «Поклевка тоже плохо выглядит.»
- «артефакт у нас выглядит практически как готовая игра»

The picture of the prototype is now the target of the look. Write the look reference: everything a maker needs to build this look in Unreal **without seeing the prototype**. The makers of the art, the shaders, the HUD and the effects will work only from your document.

## 2. What exists now (verified by the orchestrator)

- The prototype: one HTML file, 2,941 lines, canvas 2D, everything drawn by code, interface text in Russian:
  `C:\Users\<user>\AppData\Local\Temp\claude\c6c51a23e224438d\5d6e7386-f38f-41ac-a6c9-a74df26bf541\scratchpad\catch-prototype\catch-prototype.html`
- The rules of the prototype are already written: `Docs/design/rules-reference.md` (units in section 0, fish table in section 2, English names of everything). Do not repeat rules. Use its names and its units.
- The core events the Unreal layer receives: `Docs/design/core-interface.md` section 7 and section 10.
- The Unreal build today: a tilted camera on a block of water (a diorama), field of view 24 degrees, 1 game unit = 2 cm. Fish are instanced meshes with one temporary grey texture. Raft, bucket, rod are engine shapes. The HUD is plain UMG with the default font. Frames you can look at to see the gap: `Catch/Saved/ue_mcp/play_2_fish_on.png`, `Catch/Saved/ue_mcp/orch_new_day.png`, `Catch/Saved/ue_mcp/play_6_shop.png`.
- If you have a browser tool, you may run the prototype and look at it (launch name `catch-prototype` in `.claude/launch.json`, port 8765). If you do not, the drawing code is enough.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. Words, numbers, colours and timings only. No code. No coordinates of drawn paths. Describe every shape in words and ratios, so that a maker draws it new from the description. Reason: a jam rule. Nothing made before the jam goes into the game; the prototype was made before the jam.
2. Colours as hex, each with the place where it is used.
3. Sizes in the units of rules-reference section 0 and also as a share of the sea width or of the screen, because the Unreal picture has another camera.
4. Times in seconds. Easing in words (linear, ease out, overshoot and so on) with the numbers that define it.
5. Words the player sees: English, as in the rules reference. Where the prototype shows a word the rules reference does not have, translate it and mark it NEW.
6. Where the prototype decides nothing or you cannot tell, write UNKNOWN. Do not guess.

## 4. Work: the sections of `Docs/design/look-reference.md`

1. **The whole picture.** Screen layout in shares of the screen: sky, water line, sea, seabed, HUD areas. How the view moves along the depth. What is always on screen.
2. **Palette.** Sky. Water by depth zone (gradient stops). Seabed. Locked depth. Interface colours. Text colours. Accent colours of money, warnings, charms, rarity.
3. **Sea and atmosphere.** Surface and waves, light rays, drifting particles, bubbles, plants, rocks, floats, turtles, anything else alive. For each: count, size, speed, opacity, colour, where it is.
4. **Fish.** For every species and the Gold Sunfish: length in units and as a share of the sea width; body shape in words and ratios; colours of body, belly, fins, eye; marks; glow; what makes it readable at a glance. Then the motion every fish shares: tail and body wave (amplitude, frequency, how it depends on speed and state), the turn, the size change, the near/far look (tint, opacity, size by the near/far value), the look of a fish that goes to a hook, nibbles, hangs on the hook, flees, strikes as a hunter, flies to the bucket.
5. **Tackle.** Raft or boat, the fisher if there is one, bucket, rods, line, hook, the bait on the hook for every bait, the depth handle, anything that shows a rod state in the world.
6. **HUD.** Every element: place, size, colours, font (family, weight, size), corner radius, border, shadow, icon. The rod card in every state with its exact words. The hang timer and the escape warning. The top bar. The day clock. The buttons. The hover card of a fish.
7. **Panels.** Preparation, pack choice, summary, fee, shop (cards of upgrades, charms, packs, fish cards; rarity; tags; locked and new marks), run over, victory. Layout and look.
8. **Feedback.** A table, one row per moment: trigger (the core event name), what moves, what flashes, the popup text with its exact words, colours, duration, easing, the sound. Cover at least: cast, splash, nibble, bite, fish on the hook, hunter near, hunter strike, chain link, escape warning, escape, bare hook, pull, landing, flight to the bucket, bucket reaction, shipping of one fish and of a batch, money counter, fee paid, purchase, charm fired (every tag text), unlock, day start, day end, dusk, special fish, run over, victory.
9. **Sound.** Every sound: trigger, length, pitch, character in words, volume relative to the others. Music if any.
10. **Why it looks finished.** Your ranked list of the ten traits that carry the look most. Then the gap list: what the three Unreal frames lack against it, most important first.

Start the document with a list of UNKNOWN items.

## 5. Done means

1. `Docs/design/look-reference.md` exists, has the ten sections, and no code.
2. Every species of rules-reference section 2 has a fish entry.
3. Every event of core-interface section 7 has a feedback row, or the row says the prototype shows nothing.
4. A maker who never saw the prototype can build section 4 and section 8 from the text alone.

## 6. Boundaries

- You create one file: `Docs/design/look-reference.md`. Nothing else. No game file, no asset, no code.
- Do not copy code or drawing coordinates into the document.
- Do not commit, do not push, do not delete files.
- Do not open the Unreal Editor. Another executor works in it.

## 7. Report

In your hand-back, English, at most 15 lines: section sizes, the UNKNOWN items, the top five gaps, anything in the rules reference that the picture contradicts.

## 8. Chain of command

You report to the orchestrator, not to the owner.
