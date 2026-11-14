# Delta to the brief "browser prototype, second version"

Date: 2026-11-12, 02:28 (owner's clock). Written by the orchestrator after a new voice answer from the owner.

This delta overrides `prejam-prototype-v2.md` and `Docs/design/content-list.md` wherever they differ. Everything else in the brief stays: file to edit, fragment format, debug hook, smoke test, boundaries, report format. The orchestrator rewrites the content list in parallel; if you find a difference, this delta wins.

## What the owner said (short)

- Five depth zones instead of three, plus maybe a sixth far below for bottom fishing.
- Baits have rarity, like jokers: common, better, legendary. Variety matters most.
- A bare hook must still catch something, only very little. The player must never hit a dead end.
- The shelf has five slots, but some things may change that, for example items that take no slot.
- "Combo" means the chain: one fish eats the hooked fish, then a third eats that one, and so on. He could not tell whether to pull now or to wait. The game must make that clear.
- He does not like dragging a big packet onto a small slot: «он не очень удобный». His ideas: the packet tears and real bait things come out; bait loads by itself and the player only rearranges it; the player switches bait on the rod's bait counter.
- Rods always visible at the top: yes.
- The sea must feel alive: fish swim in schools, floats drift a little, plants sway, a turtle swims by, the ocean fills up gradually.
- Middle paths must work too, not only extremes. Hooks need a design: for example five fish at once on one hook.

## D1. Zones (replaces brief block 4 zones and content-list section 5)

| Zone | Depth | Opened by |
| --- | --- | --- |
| Sunlight | 0-300 | open at start |
| Twilight | 300-700 | Deeper Line 1 |
| Midnight | 700-1200 | Deeper Line 2 |
| Abyss | 1200-1800 | Deeper Line 3 |
| Trench | 1800-2500 | Deeper Line 4 |
| Seabed | floor at 3400; empty dark water from 2500 to 3300 | Deeper Line 5 |

- `WORLD_D` is about 3450. Water colour goes from turquoise at the top to almost black in the Trench. The long empty fall to the Seabed has only sparse glowing specks. The floor has sand, rocks and swaying plants. A hook in the Seabed zone rests on the floor.
- Deeper Line prices: 25, 70, 160, 380, 900.
- Locked zones stay visible as a tease, as in the brief. The camera can scroll all the way down.
- The depth gauge at the side shows all six zones with names and marks the locked ones.

## D2. Fish (replaces content-list section 6)

| Fish | Home zone | Price | Tier | Flags | Hang, s | Bites on |
| --- | --- | --- | --- | --- | --- | --- |
| Sprat | Sunlight | 2 | 1 | shoal | 7 | Worm, Bread |
| Mackerel | Sunlight | 6 | 2 | | 6 | Worm, Shrimp |
| Sea Bass | Sunlight, lower half | 15 | 3 | hunter | 6 | Shrimp, Cut Bait, Live Sprat |
| Lanternfish | Twilight | 9 | 1 | shoal, glows | 6 | Glow Worm |
| Cod | Twilight | 28 | 3 | | 6 | Shrimp, Squid Strip, Crab |
| Tuna | Twilight | 70 | 4 | hunter | 5 | Cut Bait, Squid Strip, Live Sprat |
| Hatchetfish | Midnight | 20 | 2 | shoal, glows | 6 | Glow Worm |
| Anglerfish | Midnight | 130 | 4 | hunter, glows | 5 | Glow Worm, Crab, Live Sprat |
| Oarfish | Abyss | 320 | 5 | | 5 | Squid Strip, Live Sprat |
| Gulper Eel | Abyss | 220 | 5 | hunter | 5 | Cut Bait, Live Sprat |
| Snailfish | Trench | 90 | 2 | shoal | 6 | Crab, Glow Worm |
| Giant Squid | Trench | 700 | 6 | hunter | 4 | Cut Bait, Live Sprat |
| Flounder | Seabed | 150 | 3 | lies on the floor | 6 | Worm, Crab |
| Coelacanth | Seabed | 1500 | 6 | one in the whole sea | 4 | Crab, Golden Grub |

- Hunters roam their home zone and half of each neighbouring zone.
- Fish count per zone at day 1: Sunlight 60, Twilight 40, Midnight 30, Abyss 20, Trench 18, Seabed 8. The sea gets fuller: +8% fish per day, up to +100%.

## D3. Baits with rarity (replaces content-list section 7)

| Bait | Rarity | Portions per pick | Catches | Special |
| --- | --- | --- | --- | --- |
| Worm | Common | 12 | Sprat, Mackerel, Flounder | Worm Tin is always for sale |
| Bread | Common | 16 | Sprat | Sprats bite twice as fast |
| Shrimp | Common | 8 | Mackerel, Sea Bass, Cod | |
| Cut Bait | Uncommon | 6 | Sea Bass, Tuna, Gulper Eel, Giant Squid | |
| Glow Worm | Uncommon | 6 | Lanternfish, Hatchetfish, Anglerfish, Snailfish | |
| Squid Strip | Uncommon | 5 | Cod, Tuna, Oarfish | |
| Crab | Uncommon | 4 | Cod, Anglerfish, Snailfish, Flounder, Coelacanth | |
| Live Sprat | Rare | 3 | every hunter, Oarfish | |
| Shiny Spoon | Rare | 1 | any Sunlight fish | never used up; lost when a fish escapes |
| Deep Lure | Rare | 1 | any fish below Twilight | never used up; lost when a fish escapes |
| Golden Grub | Legendary | 3 | any fish | a fish caught on it is worth x2 |
| Siren Pearl | Legendary | 1 | any fish | never used up, never lost; hunters come twice as fast |

- Pack rolls by rarity: Common 60, Uncommon 28, Rare 10, Legendary 2. A bait appears only when at least one of its fish lives in an open zone. No duplicate baits inside one pack.
- Card frame and name colour show the rarity. A legendary card shimmers.
- Each bait is drawn as the thing itself (a worm, a bread ball, a shrimp, a fish chunk, a glowing worm, a squid strip, a crab, a small fish, a metal spoon, a lure with a light, a golden grub, a pearl), not as a packet with a letter.

## D4. Bare hook (new rule)

- A rod without bait still fishes with a bare hook. Only shoal fish bite a bare hook, and the wait is four times longer.
- When a rod's bait runs out, the rod goes on with a bare hook. Its bait chip shows `BARE` and blinks for a moment. There is no `NO BAIT` state any more.

## D5. Chain is the base combo (replaces the charm "Live Bait" and the base "Haul" label)

- A hunter of a higher tier that can reach a hooked fish may eat it. The hunter is now hooked. The escape timer restarts with the hunter's hang time. The chain can go on while a bigger hunter can reach the hook.
- Each eaten fish adds +50% to the price of the fish that is finally landed. Show `CHAIN x2`, `CHAIN x3` at the hook and on the rod card, with a rising note per link.
- Remove the base `HAUL` label. The charm Big Haul stays as a charm.

## D6. Make "pull now or wait" clear (owner: he did not understand it while playing)

- Around a hooked fish and on its rod card a ring drains. In the last two seconds it turns red, the rod shakes harder, ticks get faster, and the word `SLIPPING!` shows.
- The rod card shows what a pull pays right now: `$6`, or `$21 CHAIN x2`.
- When a hunter has chosen a hooked fish, the hunter gets a visible mark (bright outline or eyes) and the rod card blinks `HUNTER NEAR`.
- The first time a fish hangs, show one hint line: "Pull now for the money, or wait: a bigger fish may eat it." The first time a fish escapes: "It slipped away. Pull before the ring runs out."

## D7. Bait loading (replaces brief block 2 "Loading" and the sockets)

No sockets. No dragging of a packet onto a rod.

- The rod strip (brief block 3) is visible always: in `prep` and in `day`, at any camera position. One card per rod. Card: rod number, level pips, state word, bait chip (bait picture and portions, or `BARE`), value and ring from D6.
- A click on the card body pulls the rod. A click on the bait chip opens a row of the baits the player owns, right under the card: picture, portions, `Catches: ...`, plus `Bare hook`. A click on one selects it. The mouse wheel over a card steps through the owned baits.
- New bait loads by itself: when the player gets a bait and a rod has a bare hook, that rod takes the new bait. The bait thing visibly drops from the chip down the line to the hook, with a snap sound.
- Bait box: a compact list of owned baits (picture and portions) at the left edge under the HUD. Hover shows the bait card. Dragging a bait from the box onto a rod card also assigns it; the whole card is the target. This drag is optional: do it only if it is cheap.
- Pack opening is the "packet" moment: the packet follows the pointer with spring, tilt and sheen until the player clicks to tear it; the bait things fan out on cards; the picked ones pour into the bait box.
- Rods on the raft stay clickable for a pull when they are on screen.

## D8. Shelf, weightless charms, fish cards, hooks (changes to brief blocks 4 and 5)

- Shelf: five slots. Upgrade `BIGGER SHELF`: +1 slot, prices 150 and 400, at most seven slots.
- A shop charm rolls as `Weightless` with 10% chance: price x1.5, it takes no slot, it is drawn with a pale glowing frame and sits beside the shelf.
- Fish Cards: the shop sells two random Fish Cards for species of open zones. A card raises that species by one level. Each level adds 100% of the base price. Card price: `6 + 4 x (current level - 1)`. Popups and the summary show the level, for example `Sprat Lv3`.
- Sharpen Rod: each level makes the rod cycle 20% faster and adds one second of hang time. Levels 3 and 5 add a hook.
- Charm list after this delta (five per class):
  - Rod: Auto-Reel 30; Barbed Hooks 20; Cluster Hook 25 "Up to 5 shoal fish can bite one hook at once."; Old Faithful 25; Big Haul 25.
  - Bait: Bait Knife 15; Thrifty Hook 20; Chum Bucket 25; Lucky Pack 15; Fresh Scent 20 "Hunters come to your hooked fish twice as fast."
  - Fish: Sprat Fever 20; Giants 35; Same Again 25; Big Shoals 20; Food Chain 30 "Each link of a chain adds +100% instead of +50%."
  - Sea: Low Tide 30; Long Day 25; Warm Current 20 "The Sunlight zone holds twice as many fish. Deeper zones hold half."; Dusk Frenzy 25; Deep Pressure 35 "Fish below the Sunlight zone are worth x2."
  - Removed: Heavy Sinker, Live Bait, Trophy. Rule texts of the others are in the content list.
- Shop layout must hold: rods and line, bigger shelf, three charms with reroll, two packs and the Worm Tin, two Fish Cards, the next-day note and the `NEXT DAY` button. Use a fixed grid. Small tiles are fine.

## D9. Living sea (new block, after block 6)

- Shoal species swim as schools: a leader on a slow path, followers with offsets and a little noise. Schools turn together.
- Fish enter from the edges of the sea block. They do not pop into view.
- Floats bob and drift a little sideways with the swell. Lines bend slightly.
- Plants sway at the sides of the block and on the Seabed.
- Ambient creatures that cannot be caught: a turtle in Sunlight, jellyfish in Twilight, glowing specks from Midnight down, a large slow shadow far in the background of the Abyss.
- Keep 60 frames per second in mind: plain canvas shapes, no per-pixel work.

## D10. Priority after this delta

1. Blocks 1 and 3 with D4, D6, D7 (one rod, prep, start, pace, always-on rod strip, bait chip).
2. Block 2 with D3 (packs, baits, rarity).
3. D5 chain.
4. Block 4 with D1, D2 (zones, fish, upgrades).
5. Block 5 with D8 (charms, shelf, fish cards).
6. Block 6 (sounds) and D9 (living sea).
7. Block 7 (fee, special fish, page chrome). In the Russian page chrome call the zones «зоны глубины», the charms «модификаторы», the chain «цепочка».

Smoke test additions: bare-hook day, a chain of three links, a weightless charm with a full shelf, a Fish Card, all five Deeper Line levels, a hook on the Seabed.

## D11. The catch is shipped at day end (owner request, 02:40)

The owner wants the fish to be sold at the end of the round, one by one, with coins popping up, so that the player enjoys every caught fish a second time.

- A landed fish no longer pays at once. It flies into a crate on the raft. Its price is computed and stored at landing time, with everything that depends on the moment: chain, Dusk Frenzy, Same Again, Golden Grub, Big Haul, species level, day rules.
- During the day the HUD shows the catch next to the money: `CATCH 12 fish  $87`. The player can compare it with the fee at any time. The crate fills up visibly.
- Phase `dayEnd` is the shipping sequence. Fish leave the crate one by one in catch order. Each one pops a coin and `+$N`, the money counter ticks up, and the pitch of the coin sound rises with every fish. A charm that changed the price of that fish wobbles and shows its bonus.
- The sequence speeds up as it goes and takes at most about 8 seconds for any catch. With a large catch, runs of the same species go as one batch (`Sprat x14  +$56`). A click or the Space key skips to the end.
- After shipping, the summary pays the fee as before.
- Bait Knife: a Sprat that becomes Live Sprat goes to the bait box at landing time and is not shipped.
- Landing still gets its own soft, pleasant sound and a small bump of the crate.
- Headless: `CatchProto.endDay()` plus `step()` must run the shipping to its end and credit all money. Smoke test: money gained in shipping equals the stored sum of the catch.
