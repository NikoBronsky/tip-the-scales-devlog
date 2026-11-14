# Feedback audit: every player input and every game event, and the answer each one gets (13.11)

Read-only audit for the orchestrator. Source: worktree `F:\UE\EpicJam_ui`, branch `track/ui`, read on 13.11 while another executor edits the same files (brief `day2-hud-fixes.md`). Line numbers are approximate and move; function names are the anchor. Paths below are under `F:\UE\EpicJam_ui\Catch\Source\Catch\` unless given in full.

Owner's rule (13.11, voice): every movement of the player and every game event gets a fun answer: motion, sound, effect. Where an answer exists it is good; many moments have none. Clarified the same day: the rule covers the whole game, not only the interface. The fish and the sea must answer too, and everything must feel pleasant. Table D covers the world; the gap list and the top 15 mix the interface and the world.

How to read the verdicts:

- **full**: motion and (where it fits) sound and effect; hover and refusal also answer.
- **partial**: some answer, but a part is missing (no sound, no motion, no answer on press or refusal, or the answer is hidden).
- **none**: nothing visible or audible.
- **in work**: section 3 of `day2-hud-fixes.md` covers it now. Not counted as a gap.
- "inferred" marks what I conclude without a line of code that proves it.

Where the answers live:

- Core events reach three listeners in the same broadcast (`UCatchSession::Tick`, `CatchSession.cpp` ~95-104): `UCatchHud::OnCoreEvent` (`CatchHud.cpp` ~2035), `ACatchTackleView::OnCoreEvent` (`CatchTackleView.cpp` ~715, which first calls `FCatchLookSound::OnEvent`), and `ACatchSeaView::OnCoreEvent` (`CatchSeaView.cpp` ~292, empty).
- State-driven answers (no event): `ACatchTackleView::DrawRods` (rod bend, line, float, hooks, hang ring, nibbles), `FCatchLookFish::Update` (`CatchLookFish.cpp` ~196-453: swim, hook swing, nibble shake, strike ring, hover silhouette), `FCatchLookSea::Tick` / `StepLife` / `DrawPlant` (`CatchLookSea.cpp`: snow, plants, turtle, jellyfish, big shadow), `FCatchLookScales::Step` (`CatchLookScales.cpp` ~92), `FCatchLookSound::Tick` (creak and ratchet tickers), `UCatchHud::RodCardView` (card words, ring, border).
- Fish positions and states come only from the core (`FishEntity`: x, depth, state Swim / ToHook / OnHook / Strike / Leave, fleeSeconds). The view adds no motion of its own except the hook swing, the nibble shake, the tail beat and the size ease.
- HUD sounds go through `CatchUi::Sound` -> `FCatchLookSound::PlayUi` (`CatchLookSound.cpp` ~163).

## 1. Table A: player inputs

Phases: P = preparation, D = day, Sh = shipping, Su = summary, S = shop, K = pack opening, R = run over, V = victory panel.

| # | Input (phase) | Where handled | Motion | Sound | Effect | Hover / press / refusal | Verdict |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A1 | Click START DAY (P) | `UCatchHud::BuildPaintedHud` lambda `StartDay` (~210); paint `CatchUi::PaintStartButton` (`CatchUiHud.cpp` ~283) | Button breathes and lifts 3 px on hover. On success it just disappears (no exit). Banner fades out (`OnPhaseChanged`). | `Bell` (core PhaseChanged -> Day). No click sound. | none | Hover: yes (lift, colour). Press: none (invisible `UCatchButton`, `MakeInvisible` sets an empty pressed brush, `CatchUi.cpp` ~754). Refusal: `Deny`; disabled look while a pack is open. | partial |
| A2 | Space (P: start day) | `ACatchPlayerController::OnSpace` (`CatchPlayerController.cpp` ~36) | as A1 | `Bell`; `Deny` on refusal | none | Refusal: `Deny` | partial |
| A3 | Space or any click (Sh: skip the weigh-in) | `OnSpace`; `UCatchHud::NativeOnMouseButtonDown` (Shipping branch); `UCatchHud::FlushShipping` skip path | Popup "+N pts" under the coins panel | `CoinBig` | 18 gold sparks (`BurstSparks`) | A second skip with nothing left: silent (inferred). | full |
| A4 | Space or Esc with nothing to do (D, Su, S, R; Esc outside the prize table) | `OnSpace`, `OnEscape` | none | none | none | Refusal: none | none |
| A5 | Close the prize table (Esc, Space, click outside, CLOSE) | `OnEscape`, `OnSpace`, `UCatchHud::UpdatePanelHits` (`PrizeOutside`, `PrizeClose`) | The table vanishes at once (fade-in only, `PaintPrizeTable` ~939). | none | none | CLOSE hover: lift. | partial |
| A6 | Click PRIZE TABLE (P, Su, S) | `UpdatePanelHits` (`PrizeTable` hits) | Dim and table fade in over 0.2 s | none (`Hit` actions add no click sound) | none | Hover: lift (`PaintButton`). Press: none. | partial |
| A7 | Mouse wheel in the prize table | `UCatchHud::NativeOnMouseWheel` (first branch) | Rows scroll | none | none | At the ends: nothing (clamped). | partial |
| A8 | Click a rod card (D: pull) | `UCatchHud::UpdateCards`, hit `Rod%d.Pull` (~305) | Card squash; rod squash (`OnPullStarted`); rod bends | `Pull`, then the `Ratchet` ticker | Bubble trail on the pulled line | Hover: card fill colour. Press: none. | full |
| A9 | Click a rod card when refused (P, Idle, Cast, Pull) | same | Card squash | `Deny` | none | Refusal: yes | full |
| A10 | Click the bait chip of a rod card (P, D) | hit `Rod%d.Bait` (~313); `PaintBaitMenu` (`CatchUiSide.cpp` ~242) | Card squash; menu pops in (0.12 s back-out) | `Tick` | none | Hover: chip colour and border. Second click closes. | full |
| A11 | Click a bait menu row (P, D) | `UCatchHud::DeclareSideHits` (`Menu%d`) | Card squash; rod flash and squash; 4 bubbles at the hook (`DrawRods`, bait change block) | `Clasp`; `Deny` if refused | rod flash | Hover: row highlight. Refusal ("none" row): `Deny` only, no shake. Menu closes at once. | full |
| A12 | Click outside the bait menu | `NativeOnMouseButtonDown` (MenuRod branch) | Menu vanishes at once | none | none | - | partial |
| A13 | Wheel over a rod card (cycle bait) | `NativeOnMouseWheel` (card loop) | Card squash; rod flash | `Clasp` or `Deny` | rod flash, bubbles | Refusal: `Deny` | full |
| A14 | Wheel over the sea (camera depth) | `NativeOnMouseWheel` -> `ACatchSeaView::MoveCameraToDepth` (0.35 s) | Camera glides; gauge window moves; sea bed mixes by depth (`TickBed`) | bed mix only | none | At the surface or the seabed limit: nothing (clamped). | partial |
| A15 | Click or drag on the depth gauge | `NativeOnMouseButtonDown` (`InGauge`), `MoveViewTo` | Camera glides 0.25 s | none | none | Hover: none (`PaintGauge` has no hover state). | partial |
| A16 | Press, drag, release the depth handle (P, D; rod without a fish) | `PickHandle`, `NativeOnMouseButtonDown/Move/Up`, `ACatchTackleView::SetDragPreview`, `PaintDrag` | Handle ring 15->17 units and brighter; dashed line across the sea; label "depth N · zone: fish ✓"; red "line ends here"; FLOOR tag | none on grab, drag or drop | none | Hover: none (ring alpha fixed 0.28, `DrawRods` depth handle block). Refusal: the core clamps silently (`SetGoalDepth`); the drop has no answer. | partial |
| A17 | Click a hooked fish or its line end (D: pull) | `PickHookedRod`, `NativeOnMouseButtonDown`; hover chip "PULL" in `PaintHoverCard` | - | - | - | - | in work (brief 3.1) |
| A18 | Click empty sea (no handle, no fish) | `NativeOnMouseButtonDown` returns Handled | none | none | none | - | none |
| A19 | Hover a fish | `UCatchHud::UpdateHover`, `PaintHoverCard`; `FCatchLookFish::Update` hover block | White silhouette behind the fish; every fish of the kind glows; card follows the cursor | none | glow | Unknown fish: "Unknown fish." card | full |
| A20 | Hover the rod pole | `ACatchTackleView::DrawRods` (rod hover) | Pole 1 unit wider, colour #fff1cc | none | none | Click on the pole: nothing. The hover promises a click that does not exist. | partial |
| A21 | Hover / click OPEN PACK in the bait box (P, S) | `DeclareSideHits` (`OpenPack%d`), `PaintBaitBox` | Button breathes +3 % on hover; note box; pack overlay rises | `Rustle` (event and HUD, gated) | gold halo | Refusal: `Deny` | full |
| A22 | Hover / click a bait box tile | `DeclareSideHits` (`Tile%d`, empty action), `PaintBaitBoxHover` | Hover: tile 106 % and the bait card | none | none | Click: nothing. | partial |
| A23 | Hover / click a shelf chip outside the shop (P, D) | `DeclareSideHits` (`Shelf%d`, empty action), `PaintCharmTip` | Hover: chip lifts 3 px, tooltip | none | none | Click: nothing. | partial |
| A24 | Click to tear a closed pack (K) | `UpdatePanelHits` (`PackTear`), `UCatchHud::TearPack` | Pack follows the pointer on a spring and tilts; strip flies off; cards fan out | `Tear` | foil sheen | - | full |
| A25 | Hover / click a bait card in an open pack (K) | `UpdatePanelHits` (`Pick%d`), `OnBaitPicked`, `PaintBaitCard` | Hover: lift, tilt, outline, sheen. Click: squash (`PokeCard`), TAKEN stamp, card rises, bait pours to the box, box tile bumps | `Flip` + `Clasp`; `Tick` when the pour lands | glint | Early click ignored: in work (3.3). Core refusal: `Deny`. | full |
| A26 | Click TO THE SHOP (Su) | `UpdatePanelHits` (`ToShop`) | Summary vanishes; shop panel rises (`PanelRise`) | `Tick` | none | Hover: lift | full |
| A27 | Click elsewhere on the summary (skip the count) | - | - | - | - | - | in work (3.11) |
| A28 | Buy a charm card (S) | `UpdatePanelHits` (`Charm%d`), `OnPurchase`, `CharmCard` (`CatchUiPanels.cpp` ~652) | Hover: tilt, lift, sheen. Click: squash; chip flies to the shelf; card turns BOUGHT | `Buy` (twice, see 4.1); `Tick` when the chip lands | 10 coins burst at the cursor | Refusal: card shakes 8 px + `Deny`; SHELF FULL bar | full |
| A29 | REROLL (S) | `UpdatePanelHits` (`Reroll`) | Offers change at once; no card motion | `Flip`; `Deny` | none | Hover: lift. Refusal: `Deny` only, no shake. | partial |
| A30 | Sharpen a rod (S) | `UpdatePanelHits` (`Upgrade%d`, `Deny` lambda), `RodTile` | Tile text "lv N" changes at once | `Buy` (twice) | 8 coins burst | Hover: tile and button. Refusal: `Deny` only. No popup. | partial |
| A31 | Buy a new rod (S) | same, `NewRodTile`; `ACatchTackleView::SyncRods` (Grow) | New rod grows from its socket (0.45 s) behind the shop panel and dim (inferred: not seen) | `Buy` (twice) | 10 coins | Refusal: `Deny` only | partial |
| A32 | Buy Deeper Line or Bigger Shelf (S) | same, `UpgradeTile` | Text changes; the veil jumps (`ACatchSeaView::SyncCoreFish`, OpenDepth set at once) behind the shop; a new shelf slot appears at once | `Buy` (twice); `Rumble` for Deeper Line | 10 coins | Refusal: `Deny` only | partial |
| A33 | Buy a bait pack (S) | `Pack%d` hit, `OnPurchase`, `PackTile` | Tile squash; pack overlay rises at once | `Buy` (twice), `Rustle` | 10 coins | Refusal: squash + `Deny` | full |
| A34 | Buy the Worm Tin (S) | `WormTin` hit | Tile squash; bait box tile bumps | `Buy` (twice) | 6 coins; popup "+N worms" | Refusal: squash + `Deny` | full |
| A35 | Buy a fish card (S) | `Card%d` hit | Tile squash | `Buy` (twice) | 8 coins; popup "Name lv N" | Refusal: squash + `Deny` | full |
| A36 | Buy chum (S) | `Chum%d` hit | Tile squash | `Buy` (twice) | 10 coins; popup "Name chum lv N" | Refusal: squash + `Deny` | full |
| A37 | Click a shelf chip in the shop (select) | `Shelf%d` hit (shop branch), `PaintShelf` | Gold border; sell popover appears at once (no pop-in) | `Tick` | none | Hover: lift, tooltip "Click to sell." | partial |
| A38 | Click SELL (S) | `Sell%d` hit, `CharmSold` case in `OnCoreEvent` | The chip vanishes at once | `Buy` (twice) | 8 coins; popup "+N" | Refusal: `Deny` | partial |
| A39 | Click NEXT DAY (S) | `NextDay` hit | The shop vanishes at once (no exit, inferred from `CurrentPanel`); day banner drops in | none (no click sound) | none | Hover: lift. Disabled while a pack is open: no answer to a click. Refusal: `Deny`. | partial |
| A40 | Click NEW RUN (R) | `NewRun` hit | Panel vanishes; day 1 banner | none | none | Hover: lift | partial |
| A41 | Click KEEP FISHING (V) | `KeepFishing` hit | Panel vanishes, the shop shows | none | none | Hover: lift | partial |

Not bound: right mouse button, other keys (`ACatchPlayerController::SetupInputComponent` binds only Space and Escape). No answer is needed for them.

## 2. Table B: core events (`EventType`, `Game.h`)

| Event | Where the view handles it | Motion | Sound | Effect | Verdict |
| --- | --- | --- | --- | --- | --- |
| PhaseChanged | HUD `OnPhaseChanged` (~2254); tackle `OnPhaseChanged` (resets); sound `OnEvent`; bed `TickBed` | Prep: day banner drops in. Day: banner out. Shipping: ship bar. Shop: panel rises. RunOver: panel rises. | `Bell` at Prep -> Day; music glides by phase | eat-day tip + `Chord` | full |
| RodStateChanged | sound `OnEvent`; tackle `OnRodStateChanged`; `DrawRods` | Cast: bob flies on an arc. Wait on the seabed: sand puff. Sink: bubble trail. | `Cast` | 10 sand dots (seabed) | full |
| HookEnteredWater | sound; tackle `OnHookEnteredWater` | Float dives 12 units and returns | `Plop` + `Sink` | 10 splash drops, surface ring | full |
| FishAttracted | not handled as an event; `FCatchLookFish::Update` (ToHook), `DrawRods` (nibbles) | Fish swims to the hook and nibbles; float dips | `Nib` (`Sound.Nibble`) | none | full |
| FishHooked | sound; tackle `OnFishHooked`; HUD tips; card from state | Rod kick +6 and jerk; fish swings on the hook; card border gold, "FISH!" | `Bite`; `Creak` ticker | 6 bubbles, surface ring, hang ring | full |
| EscapeSoon | tackle `OnEscapeSoon` is empty; the state shows it (`DrawRods` ring, `RodCardView`) | Ring blinks red/orange; card "SLIPPING!" blinks and shakes; rod bends harder | `Creak` speeds up (continuous) | none at the moment of the warning | partial |
| FishEscaped | sound; tackle `OnFishEscaped`; HUD `OnFishEscaped` | Line goes slack; lost bait token floats up | `Bloop` | 8 bubbles; popup "got away"; first-time tip | full |
| HunterStrikes | sound; `FCatchLookFish::Update` (Strike); card from state | Hunter dashes; pulsing hunter ring; card "BIGGER FISH NEAR" blinks | `Warn` | ring | full |
| ChainLink | sound; tackle `OnChainLink`; HUD `OnChainLink` | Rod kick +9; screen shake 6 px | `Chomp` + `Link` (pitch per link) | 14 dots of the eaten fish's colour, 12 bubbles; popups "CHAIN xN", "X ate the fish"; tip | full |
| PullStarted | sound; tackle `OnPullStarted`; HUD (card squash) | Rod squash, bend; card squash | `Pull`; `Ratchet` ticker | bubble trail | full |
| FishLanded | sound; tackle `OnFishLanded`; HUD `OnFishLanded` (Gold only) | Fish flies on a spinning arc to the bucket | `Burst` | 18 splash drops, 8 bubbles, surface ring; Gold Sunfish popup | full |
| BaitUsed | not handled anywhere. The core emits it when a portion leaves the stock and goes onto a hook (`Rods.cpp` ~123, ~223, ~248: cast, bait change, reload after a catch or an escape). | The portion number on the card and in the box drops without any answer (`PaintRodCard`, `PaintBaitBox`; `TileBump` only on growth, `UpdateSide`); the bait just appears on the hook | none | none | none |
| HookWentBare | HUD `OnHookWentBare` (skipped when bare by choice); tackle empty | Card chip blinks red 1.6 s | none | popup "BARE HOOK" | partial |
| DayEnded | sound only | none of its own (the rods are pulled and answer as PullStarted) | `Bell` | none | partial |
| ShipItem | HUD `OnShipItem`, `FlushShipping`; tackle `OnShipItem` | A copy of the fish flies bucket -> coins panel; bucket reacts at 60 %; ship bar counts up | `Coin` / `CoinBig`, pitch climbs per item | 5-12 gold sparks; popup per item; weight charms wobble | full (more in work, 3.11) |
| MoneyChanged | HUD `OnMoneyChanged` (reason Gold only); `UpdateMoney` for every change | Counter rolls and bumps 18 %, the same for a gain and a loss | `CoinBig` (Gold only) | charm-coin popup (Gold only) | partial |
| FeePaid | not emitted in core v2 | - | - | - | n/a |
| RunOver | sound; music factor 0.35; run-over panel rises | Panel rise | `Over` | none | partial |
| DayRuleAnnounced | not handled (tackle `OnDayRuleAnnounced` empty); the banner reads the state | - | - | - | in work (3.8, 3.9) |
| ShopOpened | not handled; the shop panel rises when the summary closes (state) | Panel rise | none | none | partial |
| Purchase | sound; HUD `OnPurchase` | Charm: chip flies to the shelf. Others: none of their own. | `Buy` (twice) | 6-10 coins burst at the cursor; popups for worms, fish card, chum; none for the four upgrades | partial |
| CharmSold | sound; HUD `OnCoreEvent` case | The chip vanishes at once | `Buy` (twice) | 8 coins; popup "+N" | partial |
| CharmFired | sound (class `Note`, gated per charm; `Bite` for Cluster Hook; `Chord` for Big Haul); HUD (shelf chip wobble); tackle (Cluster Hook jerk) | Shelf chip wobbles at the bottom of the screen | `Note` | none at the fish, hook or bucket the charm acted on | partial |
| ZoneUnlocked | sound only | Veil jumps at once (`SyncCoreFish`); gauge veil line jumps | `Rumble` (plays under the shop panel) | none | partial |
| ContentUnlocked | HUD: bait only (`NewBaits`, shop notice + `Chord`) | none | `Chord` (bait) | shop notice (bait); NEW badges from `Offer.isNew` for the rest | partial |
| PackOpened | sound; HUD `UpdatePack` (state) | Pack rises or follows the pointer | `Rustle` | foil sheen | full |
| BaitPicked | sound; HUD `OnBaitPicked` | TAKEN stamp; pour to the box; tile bump | `Flip` (twice) + `Clasp`; `Tick` | none | full |
| FishInBucket | sound; tackle `OnFishInBucket`; HUD `OnFishInBucket` | Bucket tilts, squashes, sloshes; scales kick; new tail | `Bucket` (pitch climbs per haul) | 4-6 drops; popup "kg · pts"; coins panel catch line flashes | full |
| Victory | sound; HUD `OnVictory`, `UpdatePanels` | Victory panel rises | `CoinBig` + `Chord` at the event, both again when the panel shows | 40 coins burst | full |
| DaySettled | HUD `OnDaySettled` | Summary rises; counter rolls | `Stamp`, `CoinBig` | popup "+N" | in work (3.11) |

## 3. Table C: state changes without their own event

| Change | Where the view shows it | Motion | Sound | Effect | Verdict |
| --- | --- | --- | --- | --- | --- |
| C1 Coins rise | `UpdateMoney`, `PaintMoneyPanel` | Number rolls, bumps 18 % | by cause (payout, gold, sale) | popup by cause | full |
| C2 Coins fall (purchase, reroll) | `UpdateMoney` | Same bump as a gain | none of its own | none (the coin burst at the cursor is the only sign) | partial |
| C3 Catch points grow (bucket) | `CatchFlash` in `PaintMoneyPanel`; `PaintPanPlate`; scales `Step`/`Kick` | Catch line steps to 112 % white for 0.15 s (no ease); beam tilts; the pan plate and weight numbers change with no pop | (bucket sound) | none | partial |
| C4 Day clock runs | `PaintBack` -> `PaintDayPanel` | Bar shrinks, seconds count | none | none | full |
| C5 Late day (<= 15 s or dusk) | `PaintDayPanel` (`bLate`) | Colour change only, no beat when it starts | none | none | partial |
| C6 Last 5 s | - | - | - | - | in work (3.10) |
| C7 Catch reaches the target during the day | `FCatchLookScales::Step` (kick +4.2 rad/s, `Flash` bloom), `ACatchTackleView::StepBucket` (`TakeCrossed`); plate turns green | Beam kick; bloom at the pivot; ring of 16 drops from the bucket; slosh | none | none (no word) | partial |
| C8 A zone opens | `SyncCoreFish`, `FCatchLookSea::Tick`, `PaintGauge` | Veil, lock label and gauge jump at once, while the shop covers the sea | (`Rumble` at purchase) | none | partial |
| C9 Rule day | banner, rule box, forecast | - | - | - | in work (3.8, 3.9) |
| C10 A charm does its work | `CharmFired` (where the core emits it); `Mods().noEscape` -> teal held ring | Shelf chip wobble; passive charms (bite rate, population, size) have no sign at all | `Note` | none at the place | partial |
| C11 A charm counter grows | not in the core yet | - | - | - | n/a (section 6) |
| C12 An offer is not affordable | `MakeShopView` (`Style`, `bShort`) | Price bar red, static; it changes colour at once after a purchase | none | none | partial |
| C13 The shelf is full | `CharmCard` (SHELF FULL bar), `SellNote` | Static grey bar | none | none | partial |
| C14 Bait stock falls or runs out | `PaintRodCard` (portion count red at 0), `PaintBaitBox`, menu "none" | Number changes, turns red at 0 | none (until HookWentBare) | none; no warning at 1 left | partial |
| C15 Escape warning (state part) | `DrawRods`, `RodCardView`, `FCatchLookSound::Tick` | Ring blinks; card blinks and shakes; line trembles; rod bends | `Creak` speeds up and rises in pitch | none | full |
| C16 A new rod appears | `SyncRods` (Grow), `UpdateCards` (new card) | Grow-in happens under the shop (inferred: unseen); the new card appears without a pop | none | none | partial |
| C17 A rod levels up | `DrawRods` (`PoleLen`, `RodColorOf`), `PaintRodCard` (level dots) | Pole length and colour change at once | none | none | partial (badge digit in work, 3.7) |
| C18 Camera moves | `MoveCameraToDepth`, `UpdateLookCamera`, bed depth mix | Smooth glide | bed mix | none | full |
| C19 The run ends | `PaintRunOver`, bed music 0.35 | Panel rise | `Over` | none | partial |
| C20 A bite or strike outside the camera view | `RodCardView`, `MakeGaugeView` (gold dot) | Card turns gold; gauge dot gold; nothing points at the place | the event sounds | none | partial |
| C21 Gold Sunfish in the sea | `FCatchLookFish::Update` (bloom) | Golden bloom | none | bloom | full |
| C22 Fish in the bucket | `DrawRods` (tails) | Tails bob with the slosh | - | - | full |
| C23 First day when fish eat | `OnPhaseChanged` | Tip | `Chord` | tip bar | full |

## 4. Table D: the world answers

What the sea and the things in it do at each player action and core event. "World" = fish, schools, line, hook, bait, float, rod, water surface, raft, bucket, scales, plants, sea life, snow, camera. The sound column of tables A-C is not repeated here.

| # | Moment | What the world does now (file, function) | Verdict | Proposed answer with existing means | Cost | Where |
| --- | --- | --- | --- | --- | --- | --- |
| D1 | Day start: every rod casts | Rod bend -0.6 then +0.3 on its spring; the red bob flies on an 80-unit arc from the tip (`DrawRods`, Cast branch) | full | - | - | - |
| D2 | The hook hits the water | 10 splash drops, a surface ring (8 -> 54 units in 0.625 s), the float dives 12 units and returns in 0.4 s (`OnHookEnteredWater`, `SpawnSplash`, `SpawnSurfaceRing`, float block of `DrawRods`) | full | - | - | - |
| D3 | The hook sinks past fish | Bubble trail 30 per second at the hook (`DrawRods`, `BubbleCarry`). Fish near the falling hook do nothing: their motion is the core's only (`FCatchLookFish::Update`) | partial | **Startle shiver**: a view-only list of short "startle points" (x, depth, radius, age). Every drawn fish within the radius beats its tail x 2.2 for 0.5 s, fading (per-instance `D_Rate`, phase kept as the code already does when the rate changes) and rocks its pitch +-0.08 rad once. No position change, so picks and hooks stay true. The sinking hook drops one point every 0.25 s (radius 60 units). | M (the mechanism, once) | `FCatchLookFish::FFrame` gets `TArray<FVector4f> Startles`; `FCatchLookFish::Update` (tail beat block ~289); `ACatchTackleView` fills it (`DrawRods`, Sink branch); `ACatchSeaView::UpdateLook` passes it |
| D4 | The hook comes to rest (Sink -> Wait) | Seabed only: a puff of 10 sand dots (`OnRodStateChanged`). In open water the float and the line just stop | partial | The float settles: a 4-unit dip and return over 0.3 s (the `DiveAge` curve at a third of its depth); 2 bubbles at the hook | S | `ACatchTackleView::OnRodStateChanged` (any zone), `FRodParts` (one more age), float block of `DrawRods` |
| D5 | A fish is attracted and nibbles | The fish swims to the hook (core ToHook); at the hook it shakes 2.5 units at 18 rad/s (`FCatchLookFish::Update`, `Nibbling`); the float dips 5 units per nibble (`DrawRods`, `NibbleDip`) | full | - | - | - |
| D6 | The fish bites (FishHooked) | Rod kick +6 and jerk; 6 bubbles; a surface ring (`OnFishHooked`). The fish turns nose up and swings +-4 units at 14 rad/s, rocks +-0.22 rad, tail beat x 2 (`FCatchLookFish::Update`, Main block). The line goes straight and trembles; the float is held down 7 units (`DrawRods`) | full | - | - | - |
| D7 | Its school when one fish bites | The core takes the fish out of its school (`Game::AttachFish` -> `LeaveSchool`); the other members swim on as before | none | Startle shiver (D3) at the hook, radius 150 units: the school mates nearby flinch. | S (after D3) | `ACatchTackleView::OnFishHooked` adds the point |
| D8 | A hooked fish waits | Swing and rock; 4 bubbles per second leak from it; the hang ring arc runs; rod bend trembles 0.22 at 17 rad/s (`DrawRods`) | full | - | - | - |
| D9 | The escape warning (last seconds on the hook) | Ring blinks red/orange; rod tremble 0.45 at 34 rad/s; line tremble 2.4 units at 70 rad/s (`DrawRods`). The fish moves exactly as before: the Main block of `FCatchLookFish::Update` does not read the escape time | partial | **The fish fights**: while `escapeSeconds <= pace.escapeWarning` (and not No Escape) the swing grows from 4 to 7 units and 14 to 22 rad/s, the rock from 0.22 to 0.35 rad, the tail beat x 1.5. `FFrame` already carries `State` and `Tables`. | S | `FCatchLookFish::Update`, Main block (~314) |
| D10 | The fish escapes | The fish flees at 4 x speed for 1.2 s toward a new depth (core `Game::FleeFish`, `Fish.cpp` ~161); line slack bow of 40 units; 8 bubbles; a lost bait token floats up (`OnFishEscaped`). The float jumps up 7 units in one frame (the `bFish` term of `FloatUp` drops at once; inferred from the code, no spring) | partial | The float pops back on a curve: +6 units over the rest line, then settles in 0.4 s; a surface ring (`SpawnSurfaceRing`). | S | `ACatchTackleView::OnFishEscaped`, float block of `DrawRods` |
| D11 | A bigger fish strikes (HunterStrikes) | The hunter dashes (core Strike), nose tipped up, a pulsing #ff7850 ring, strike eye (`Mode` 2) (`FCatchLookFish::Update`). The prey on the hook, the rod and the sea around do nothing (`ACatchTackleView::OnHunterStrikes` is empty) | partial | **The prey panics**: while its hook has a hunter (`HookData::hunter != None`) the hooked fish swings as in D9. The rod jerks 0.6. A startle point at the hunter (radius 120 units): small fish nearby flinch. Shake 3 px (section 6). | S | `FCatchLookFish::Update` Main block, `ACatchTackleView::OnHunterStrikes` |
| D12 | The hunter eats the fish (ChainLink) | The eaten fish vanishes in one frame (core `Game::EatFish` -> `Doom`, removed in the same step); 14 dots of its colour, 12 bubbles, rod kick +9, shake 6 px (`OnChainLink`). The hunter becomes the hooked fish with no gulp | partial | **Gulp**: the hunter's drawn size jumps x 1.25 and eases back by the existing size ease (2 per second, `D.Size` in `Update`); a surface ring. Needs one `Punch(FishId, Factor)` call on `FCatchLookFish`. | S | `FCatchLookFish` (new small method), `ACatchTackleView::OnChainLink` |
| D13 | The player pulls (Pull) | Rod squash, bend 0.95; bubble trail 25 per second; the hooked fish rises with the hook and swings as while waiting (`DrawRods`, `FCatchLookFish::Update`). Nothing at the surface when the pull starts | partial | The fish thrashes while it rises (swing and tail x 1.5 while the rod state is Pull); a surface ring and 4 splash drops at the start of the pull. | S | `FCatchLookFish::Update` Main block (the rod state is in `State`), `ACatchTackleView::OnPullStarted` |
| D14 | The fish leaves the water (FishLanded) | 18 splash drops, 8 bubbles, a surface ring; the fish flies on a spinning arc to the bucket (`OnFishLanded`, `UpdateFlights`). Fish near the hook line and the raft do nothing | partial | A startle point at the hook line under the surface (radius 150 units); see D15 for the raft. | S (after D3) | `ACatchTackleView::OnFishLanded` |
| D15 | The fish drops into the bucket (FishInBucket) | Bucket tilt, squash, slosh, 4-6 drops, a new tail; scales kick (`OnFishInBucket`, `BucketHit`, `FCatchLookScales::Kick`). The raft does not move: it bobs on time only (`StepBucket`, `RaftBob`) | full (raft: partial) | The raft dips 2 units under the weight and springs back (one damped term added to `RaftBob`; the rods, bucket and scales already ride it). | S | `ACatchTackleView::StepBucket`, `BucketHit` |
| D16 | The catch reaches the target | Beam kick +4.2 rad/s, bloom at the pivot (`FCatchLookScales::Step`), 16 drops from the bucket, slosh (`StepBucket`). No sound (gap G25) | partial | See G25 (sound and words). In the world add a raft dip of 3 units. | S | `StepBucket` (`TakeCrossed`) |
| D17 | A new day: the target weight drops onto the scales | The weight falls 160 units onto the right pan and kicks the beam (`FCatchLookScales::Step`). `TakeLanded()` (`CatchLookScales.h` ~61) is never read: the landing has no sound and no dust | partial | On `TakeLanded()`: `Stamp` at pitch 0.6, 6 grey dots at the pan (`SpawnDots`), a raft dip of 2 units. The weight size says how heavy the day is; the thud makes it felt. | S | `ACatchTackleView::StepBucket` |
| D18 | Shipping: each fish leaves the bucket | A copy flies bucket -> coins (0.45 s, spinning, shrinking); the bucket reacts at 60 % with 2-3 drops (`OnShipItem`) | full | - | - | - |
| D19 | The player changes the bait of a rod | Rod flash, squash; 4 bubbles at the hook when it is in the water (`DrawRods`, bait change block) | full | - | - | - |
| D20 | A portion goes onto a hook (BaitUsed: cast, reload after a catch or an escape) | The bait card just appears on the hook (`DrawRods` hook block) | none | The bait card pops on: scale 0 -> 1.3 -> 1 over 0.2 s in the bait colour, 2 bubbles. Pairs with the card number pop of G7. | S | `ACatchTackleView::OnBaitUsed` (empty now), a per-hook pop age in `FRodParts`, `DrawRods` hook block |
| D21 | A hook goes bare (HookWentBare) | Nothing in the world (`ACatchTackleView::OnHookWentBare` is empty) | none | The bare hook flashes white once (hook card alpha 1 -> bright over 0.3 s) and lets out 3 bubbles. | S | `ACatchTackleView::OnHookWentBare`, `DrawRods` hook block |
| D22 | A fish under the cursor | White silhouette 14 % longer and 34 % taller; every fish of the kind glows (`FCatchLookFish::Update`, hover block). The fish itself does not react | partial | The hovered fish "notices": its tail beat x 1.5 while hovered (`D_Rate` of its slot and of the hover copy). | S | `FCatchLookFish::Update` (tail beat block uses `F.HoverId`) |
| D23 | Turtle, jellyfish, big shadow | Swim on their own clock only (`FCatchLookSea::StepLife`). Nothing reacts to a splash, a hook or a fish | none | The turtle within 150 units of a splash or a landing turns away and swims x 2 for 1 s; a jellyfish in a startle point pulses its bell x 2 for 1 s. Reuses the startle list of D3. | M | `FCatchLookSea::StepLife` (`FView` gets the startle list) |
| D24 | Plants on the walls and the floor | Sway 8 units at 0.9 rad/s on time only (`FCatchLookSea::DrawPlant`) | none | A plant within 80 units of a startle point (a hook on the seabed, a fleeing fish, a puff of sand) bends away by up to 12 units and springs back over 1 s. | M | `FCatchLookSea::DrawPlant`, `Tick` |
| D25 | Marine snow and plankton lights | Sink and sway on time only (`FCatchLookSea::Tick`, 3.3 block) | none | Specks within 40 units of a pulled line or a splash are pushed sideways and drift back (one offset per speck, damped). | M | `FCatchLookSea::Tick` |
| D26 | Camera | Glides on wheel and gauge (`MoveCameraToDepth`); shakes only on ChainLink, 6 px (`ACatchSeaView::AddShake`, the only caller is `OnChainLink`) | partial | Shake 2 px when a fish of tier 3 or more lands, 3 px on HunterStrikes, 10 px on RunOver (G33). No automatic camera moves (they would take control away from the player). | S | `ACatchTackleView::OnFishLanded`, `OnHunterStrikes`, `OnRunOver` |
| D27 | Dusk (the last 15 s, `State::dusk`) | The sea does not change; only the HUD bar turns orange (`PaintDayPanel`) | none | A warm overlay over the sea in the painted HUD: a full-screen box in `DuskBar` at 6 %, fading in over 2 s, under every panel. No material change. | S | `UCatchHud::PaintBack` (first layer) |
| D28 | A zone opens | The veil and the lock labels jump at once under the shop panel (`SyncCoreFish`, `FCatchLookSea::Tick`) | partial | See G44: the veil slides down on the next day start. | M | `ACatchSeaView::SyncCoreFish` |
| D29 | The Gold Sunfish enters | It swims in from an edge with its golden bloom (`FCatchLookFish::Update`); no moment marks its arrival | partial | On its first drawn frame its bloom pulses from 0.9 to 1.6 alpha and back over 1.5 s; `Chord` at volume 0.4. | S | `FCatchLookFish::Update` (new slot of kind GoldSunfish), sound through the sea view |
| D30 | Giants: fish grow | The size eases at 2 per second (`FCatchLookFish::Update`) | full | - | - | - |
| D31 | Cluster Hook: a shoal fish joins the hook | It hangs beside the main fish, 10 + 3k units out; the rod tip jerks (`OnCharmFired`) | full | - | - | - |
| D32 | Rods in preparation | A short line hangs from the tip with hook and bait; the dashed guide and ghost hooks show the goal depth (`DrawRods`, Idle branch) | full | - | - | - |

Notes to Table D:

- The fish in this game do not flee from hooks, pulls or strikes: only an escaped fish flees (core `Game::FleeFish`). Making nearby fish really swim away would change which fish can bite (`fleeSeconds > 0` blocks a bite, `Rods.cpp` ~326), so it is a rule, not feedback. The startle shiver of D3 moves no fish and changes no rule.
- The fish material already takes every value the proposals need per instance: colour, ratio, length, near/far, kind, phase, tail rate, mode (`FCatchLookFish::EData`; `Mode` 0 swim, 1 hover silhouette, 2 strike eye, 3 flight; `build_look_fish.py`). No material edit is needed for D3, D9, D11, D12, D13, D22.

## 5. Unused or under-used material

### 5.1 Sounds (29 action files and 3 bed loops; `SourceArt/Sound/README.md`)

- `S_CoinArrive`: never plays. The only call is in `UCatchHud::UpdatePanels` (~1695) for a coin with `bBurst = false` and `bTick = true`, and no code makes such a coin (`BurstCoins` makes only burst coins). Dead path, ready for coin-by-coin flights (3.11 may use it).
- Played twice in the same frame (bug, doubles the volume): `Buy` on Purchase and CharmSold (core path `FCatchLookSound::OnEvent` + HUD `OnPurchase` / `CharmSold` case; `Buy` has no gate in `GateOf`); `Flip` on BaitPicked (`OnEvent` + `UCatchHud::OnBaitPicked`; no gate). `Clasp` and `Rustle` double too but their gates drop the second play (inferred). `Chord` + `CoinBig` on Victory play at the event and again when the panel shows (`UpdatePanels`).
- Played in one place only: `Stamp` (DaySettled), `Rumble` (ZoneUnlocked, heard under the shop), `Warn` (HunterStrikes), `Over` (RunOver), `Tear` (pack tear), `Note` (CharmFired), `Bell` (day start and end), `Nib` (nibbles).
- Rare: `Chord` (Big Haul, new bait notice, eat-day tip, victory).
- Free for new answers without new files (pitch and volume are free per call, `FCatchLookSound::Play(Name, Pitch, Volume)`): `Warn` (escape warning, run over), `Bell` at a higher pitch (late day), `Note` (counters, level up), `Link` (rising steps), `Stamp` (the day weight landing, refusals with weight, run over), `Rumble` (zone reveal, run over), `Clasp` (depth drop), `Ratchet` (depth drag ticks), `Tick` (every button).

### 5.2 Interface helpers that exist and can be reused

HUD (`CatchHud.cpp`):

- `AddPopupWorld`, `AddPopupRef` (stack upwards); popup styles `EPopup::Fee` (SoftRed, 20 px) and `EPopup::Paid` (Coral, 50 px) are never used.
- `BurstCoins(At, N)` (coins under gravity), `BurstSparks(N)` (gold sparks from the catch line), `FCoin` flight with `From/To/Arc/Delay/bTick` (painted and updated, never spawned).
- `FlyChips` (charm chip flight with arc, scale and spin), `PokeCard(Key)` (squash any floating card), `CharmShake[3]` (shake, charms only), `FCardUi::Squash`, `FCardUi::BareBlink`, `TileBump`, `ShelfWobble`, `MoneyBump`, `CatchFlash`.
- `ShowTip`, `ShowBanner` / `PaintBanner`, `PaintDim`, `PanelRise`, `Appear`, `BackOut`, `Smoothstep`, `Ease`, `Blink`, `NewBadge`.
- New from the other executor: `CrossOut`, `PaintRuleIcon`, `PaintRuleCard`, `PaintRuleBox` pulse, `RuleReminderPulse` (`CatchUiHud.cpp` ~332-431, `CatchHud.cpp` ~494).
- Unused leftovers: `EPanel::Paid` (empty case in `PaintPanels`), `PaidTime`, `PayoutLandsAt`, `Panel::PayoutHold`, `PayoutLineStart`, `PayoutLineGap`, `FeeCoins`, `FeeCoinGap`, `FeeCoinFly`, `FeeCoinArc`, `StampSlam`, `StampFade`, `PaidHold` (`CatchUiPanels.h` ~35-48).
- `FPanelHit::PressName` / `PressClock` and `PressedSince` (added for 3.3): they record which hit a press began on. This is enough for a pressed look on every painted button.

### 5.3 World helpers that exist and can be reused

- Tackle (`CatchTackleView.cpp`): `SpawnBubbles`, `SpawnSplash`, `SpawnDots(Color)`, `SpawnSurfaceRing`, `BucketHit(Strength, FromX, Drops)`, the lost bait token (`LostBaits`), fish `Flights`, `RaftBob`.
- Per rod (`FRodParts`): `Kick`, `Jerk`, `Squash`, `Flash`, `Slack`, `DiveAge`, `NibbleDip`, `Grow` (`LookRegrow` replays it).
- Scales (`CatchLookScales.cpp`): `Kick`, `Flash` (bloom), `TakeCrossed()` (used), `TakeLanded()` (never used).
- Sea view: `ACatchSeaView::AddShake(px)` (one caller), `MoveCameraToDepth(depth, seconds)`, `SameScreenPoint`.
- Fish look (`CatchLookFish.cpp`): per-instance tail rate (`D_Rate`) with a phase-safe rate change already coded (~292), `D_Mode`, colour, the size ease (`D.Size`, 2 per second), `Nibbling`, the effect cards `ShapeDisc`, `ShapeRing`, `ShapeBloom`, `GetMouth`, `GetFishInfo`.
- Sea atmosphere (`CatchLookSea.cpp`): `FCreature` (turtle, jellyfish, big shadow) and `FPlant` with their own phase and sway; `FView` is the one input and can carry startle points.
- Material parameters driven every frame: `MPC_Sea` (`WaterB`, `OpenDepth`, `UnitsToCm`); bucket material `Wave`, `Tilt`; raft material `BucketX/Y`, `PanRX/RY`, `BucketTilt`, `RaftLeft`; beam `Flash`. The surface material `M_Surface` moves on time only (`build_sea_materials.py` ~242); no event reaches it, and the surface rings stand in for it.

## 6. The gap list, ranked by how often a player meets it

Interface and world are mixed. Inside a tier the order is the rank. No new assets, no new sound files, no new rules, no new screens. New English words are marked "text"; the orchestrator must allow each one.

### Tier 1: every few seconds (every click, every cast, every bite)

1. **G1 Pressed look on every button and hit** (A1, A6, A8, A26-A41). Press: the painted button scales to 96 % and drops its hover lift; release: the existing action. Read `FPanelHit::PressName` while the mouse button is down; for the fixed hits (`StartButton`, rod cards) use `UButton::IsPressed()`. Cost M. `UCatchHud::IsHot` gets an `IsPressed` twin; `CatchUi::PaintButton`, `PaintStartButton`, `PriceButton`, `PaintRodCard` get a press parameter.
2. **G2 Click sound for every panel hit** (A5, A6, A39, A40, A41). `UCatchHud::Hit` wraps the action with `Tick`, as `MakeButton` already does (~148); actions with their own sound (`Buy`, `Flip`, `Deny`) pass a flag to skip it. Cost S. `UCatchHud::Hit`.
3. **G3 Fish startle shiver** (D3, D7, D14, and the base of D11, D23, D24). The startle list of D3: fish near a sinking hook, near a bite, near a landing and near a strike flinch (tail x 2.2 for 0.5 s, one pitch rock). No fish moves, no rule changes. Cost M once, then S per use. `FCatchLookFish::FFrame`, `FCatchLookFish::Update`, `ACatchTackleView` (`DrawRods` Sink branch, `OnFishHooked`, `OnFishLanded`, `OnHunterStrikes`), `ACatchSeaView::UpdateLook`.
4. **G4 The hooked fish fights near the escape** (D9). Swing 4 -> 7 units, 14 -> 22 rad/s, rock 0.22 -> 0.35 rad, tail x 1.5 under the escape warning. Cost S. `FCatchLookFish::Update`, Main block.
5. **G5 Escape warning, one-shot** (B EscapeSoon). On the event: `Warn` at pitch 1.3 and volume 0.7 (gate 0.2 s exists); the hang ring of that hook pops (scale 1.25 -> 1 over 0.2 s); the rod card squashes (`Cards[rod].Squash = 1`). With No Escape active: no answer. Cost S. `FCatchLookSound::OnEvent` (new case); `ACatchTackleView::OnEscapeSoon` (empty now, add `RingPop` to `FRodParts`); HUD `OnCoreEvent`.
6. **G6 Depth handle** (A16). Hover within 45 px: ring alpha 0.28 -> 0.7 and radius 15 -> 17. Grab: `Tick` + 4 bubbles at the hook. Drag: `Ratchet` at volume 0.5 every 25 depth units crossed, pitch rising with depth (x 1.0 .. 1.5). Drop: `Clasp`, the handle ring pops once, the rod `Jerk` = 0.5. Over the deepest allowed depth: `Deny` once and the red "line ends here" line blinks. Cost M. `UCatchHud::NativeOnMouseButtonDown/Move/Up`; `ACatchTackleView::DrawRods` (depth handle block); add `SetHandleHover(Rod)` beside `SetDragPreview`.
7. **G7 A portion goes onto a hook** (B BaitUsed, C14, D20). In the world: the bait card pops onto the hook (0 -> 1.3 -> 1 over 0.2 s) with 2 bubbles. On the card: the portion number pops (1.3 -> 1 over 0.2 s, white flash); the box tile gets `TileBump` 0.5 on a fall too; at 1 portion left the number pulses gold. Sound: `Tick` at volume 0.6 (or none, decide by ear). Cost S. `ACatchTackleView::OnBaitUsed`, `DrawRods` hook block; HUD `OnCoreEvent` (new case), `FCardUi::PortionPop`, `PaintRodCard`, `UCatchHud::UpdateSide`.
8. **G8 The hovered fish notices** (D22). Tail beat x 1.5 while hovered. Cost S. `FCatchLookFish::Update`.
9. **G9 Bucket points pop** (C3). The pan plate and the weight number scale 1.25 -> 1 with `BackOut` over 0.25 s; `CatchFlash` becomes an eased pop instead of a 0.15 s step. Cost S. `PaintPanPlate`, `PaintWeightNumber`, `PaintMoneyPanel` (`CatchUiHud.cpp`), driven by `CatchFlash` in `UCatchHud::UpdateMoney`.
10. **G10 Charm fired at its place** (B CharmFired, C10). Keep the chip wobble and `Note`; add a popup at the place the charm acted on: the bucket for a landed fish (`E.catchIndex`), the hook for a rod charm (`E.rod`, `E.hook`), in the class colour (`ClassGlow`), text = `Ui::CharmTag(name)` + the amount from `E.price`/`charmAmounts` (format "+N pts" exists). A thin gold spark burst (4 sparks) from the chip. Cost M. `UCatchHud::OnCoreEvent` (CharmFired case).
11. **G11 The fish thrashes on the pull** (D13). Swing and tail x 1.5 while the rod pulls; a surface ring and 4 splash drops when the pull starts. Cost S. `FCatchLookFish::Update` Main block, `ACatchTackleView::OnPullStarted`.
12. **G12 The prey panics at a strike** (D11). The hooked fish swings as in G4 while a hunter comes; the rod jerks 0.6; startle point at the hunter; shake 3 px. Cost S. `FCatchLookFish::Update`, `ACatchTackleView::OnHunterStrikes`.
13. **G13 The hunter gulps** (D12). Size x 1.25 eased back by the existing size ease; a surface ring. Cost S. `FCatchLookFish` (`Punch`), `ACatchTackleView::OnChainLink`.
14. **G14 The float pops back after an escape** (D10). +6 units overshoot, settles in 0.4 s; a surface ring. Cost S. `ACatchTackleView::OnFishEscaped`, `DrawRods` float block.
15. **G15 The raft dips under each fish** (D15, D16, D17). One damped dip term on `RaftBob`: 2 units per fish in the bucket, 3 at the target, 2 at the day weight. Cost S. `ACatchTackleView::StepBucket`, `BucketHit`.
16. **G16 The float settles when the hook stops** (D4). 4-unit dip, 0.3 s, 2 bubbles, in every zone. Cost S. `ACatchTackleView::OnRodStateChanged`, `DrawRods`.
17. **G17 Hook went bare: sound and hook flash** (B HookWentBare, D21). `Deny` at pitch 0.8 with the existing popup and chip blink; the bare hook flashes white once and lets out 3 bubbles. Skip when bare by choice, as the HUD does. Cost S. `FCatchLookSound::OnEvent` (needs `S.rods[E.rod].bareByChoice`), `ACatchTackleView::OnHookWentBare`.
18. **G18 Camera shake for big moments** (D26). 2 px on a landing of tier 3 or more, 3 px on HunterStrikes (with G12). Cost S. `ACatchTackleView::OnFishLanded`, `OnHunterStrikes`.
19. **G19 Bite or strike outside the view** (C20). On FishHooked, HunterStrikes and ChainLink, when the hook is outside the view depths (`UCatchHud::ViewDepths`), the gauge dot of that rod pops and pulses for 1.5 s, and the rod card squashes. Cost S. `UCatchHud::OnCoreEvent`, `MakeGaugeView`, `PaintGauge`.
20. **G20 Rod pole hover promises a click** (A20). Either a click on the pole pulls, as the card does (a control mapping, revocable technical decision), or the pole hover is removed. Recommended: click pulls. Cost S. `ACatchTackleView::DrawRods` exposes the hovered rod; `UCatchHud::NativeOnMouseButtonDown` uses it after `PickHookedRod`.
21. **G21 Click on empty sea** (A18). 3 bubbles at the clicked point on the line plane (`DepthAtPixel` gives the depth), `Nib` at volume 0.6, and a startle point there (G3): the fish near the click flinch. Cost S. `UCatchHud::NativeOnMouseButtonDown` end; a public `ACatchTackleView` wrapper.
22. **G22 Bait menu closes without an answer** (A12, A11 refusal). The menu shrinks and fades over 0.1 s; a "none" row shakes 6 px (the `CharmShake` formula). Cost S. `UCatchHud::SetMenuRod`, `PaintBaitMenu`.
23. **G23 Camera at its limits; gauge hover** (A14, A15). At the surface or the seabed a wheel step gives an 8-unit overshoot and return (two `MoveCameraToDepth` calls). The gauge border brightens on hover. Cost S. `UCatchHud::NativeOnMouseWheel`, `PaintGauge`.

### Tier 2: every day (every shop, every day start and end)

24. **G24 START DAY press and exit** (A1, A2). Press: 94 % squash. On PhaseChanged -> Day: the button pops to 110 % and fades out over 0.25 s; 12 gold sparks burst from it. Cost S. `UCatchHud::OnPhaseChanged`, `PaintBack`, `PaintStartButton`.
25. **G25 Target reached during the day** (C7, D16). Add `Chord` at the scales crossing; a popup over the scales with the existing text "The catch reached the target" (style `GoldLanding`); the coins panel target line pops and turns green with a short white flash; `MoneyBump`; the raft dips (G15). Cost S. `ACatchTackleView::StepBucket` (`TakeCrossed`); HUD: first crossing per day in `UpdateMoney`.
26. **G26 Late day and dusk** (C5, D27). When the clock first goes late: `Bell` at pitch 1.5 and volume 0.5 once; the day panel rim pulses gold for 1.5 s; a warm overlay over the sea (6 %, 2 s fade-in). The 5-4-3-2-1 countdown is in work (3.10). Cost S. `UCatchHud::PaintBack` (edge detect on `bLate`), `PaintDayPanel`.
27. **G27 The day weight lands on the scales** (D17). `Stamp` at pitch 0.6, 6 grey dots at the pan, a raft dip, on `TakeLanded()`. Cost S. `ACatchTackleView::StepBucket`.
28. **G28 Day ended** (B DayEnded). A short banner "DAY N OVER" (text exists in `PaintSummary`) for 1.2 s; shake 3 px (`AddShake`). Cost S. `UCatchHud::OnCoreEvent` (DayEnded case).
29. **G29 Coins spent** (C2, B MoneyChanged). On a fall: the amount flashes SoftRed for 0.3 s instead of the gold bump, and a popup "-N" under the panel in the unused `EPopup::Fee` style. Cost S. `UCatchHud::UpdateMoney`, `PaintMoneyPanel`, `OnMoneyChanged` (reasons Purchase and Reroll).
30. **G30 Refusal shake for every shop offer** (A29-A36, A39 refusals; C12). Turn `CharmShake[3]` into a map keyed by hit name; every `Deny(R)` in `UpdatePanelHits` shakes its tile, and a refused `NotEnoughMoney` also pulses the coins amount red (with G29). A click on the disabled NEXT DAY gives `Deny` and a shake. Cost M. `UCatchHud::UpdatePanelHits`, `PaintShop` tiles.
31. **G31 Upgrade purchases** (A30-A32, B Purchase). The bought tile pops (110 % -> 100 %, `BackOut`, 0.25 s); popups with existing words: "ROD N lv M" for Sharpen, "ROD N" for New Rod, the opened zone name for Deeper Line (`ZoneData::name`), "now N slots" for Bigger Shelf. Cost M. `UCatchHud::OnPurchase` (content Upgrade), a `TilePop` map read by `MakeShopView`.
32. **G32 Reroll flips the cards** (A29). The three charm cards squash one after another (`PokeCard("Charm0..2")`, 0.06 s apart) and get a sideways spring kick (`FCardMotion::TiltX.V += k`); `Flip` per card, pitch +2 semitones each. Cost S. `UpdatePanelHits` (Reroll), `UCatchHud::UpdateCardMotion`.
33. **G33 Shop opens and closes** (B ShopOpened, A39). Shop rise gets `Rustle` at pitch 0.7; NEXT DAY plays `Tick`, and the shop sinks out over 0.25 s before the day banner drops. Cost M. `UCatchHud::UpdatePanels`, `PaintShop`.
34. **G34 Shelf select and sell** (A37, A38, B CharmSold). The sell popover pops in (0.12 s back-out). The sold chip flies from its slot to the coins counter (`FlyChips`), then the coins burst there. Cost S-M. `PaintShelf`, `CharmSold` case in `UCatchHud::OnCoreEvent`.
35. **G35 Affordability changes** (C12, C13). After a purchase, every offer that turns unaffordable flashes its price bar red once; SHELF FULL bars drop in over 0.2 s. Cost S. `UCatchHud::MakeShopView`, `UpdatePanels`.
36. **G36 Double sounds** (5.1). Remove the HUD `Buy` in `OnPurchase` and the `CharmSold` case, the HUD `Flip` in `OnBaitPicked`, and the second `Chord` + `CoinBig` in `UpdatePanels` (Victory), or give `Buy` and `Flip` a 0.05 s gate in `GateOf`. Cost S. `CatchHud.cpp`, `CatchLookSound.cpp` (`GateOf`).
37. **G37 Prize table open, close and scroll ends** (A5, A6, A7). `Tick` on open; shrink and fade over 0.15 s on close with `Tick`; a 6 px bounce at the scroll ends. Cost S. `UCatchHud::OpenPrizeTable` / `ClosePrizeTable`, `NativeOnMouseWheel`, `PaintPrizeTable`.
38. **G38 Space or Esc with nothing to do** (A4). `Deny` at volume 0.6. Cost S. `ACatchPlayerController::OnSpace`, `OnEscape`.
39. **G39 Dead clicks on the bait box tile and the shelf chip outside the shop** (A22, A23). The tile bumps with `Tick`; the shelf chip wobbles with its class `Note`. Cost S. `UCatchHud::DeclareSideHits`.
40. **G40 New content other than bait** (B ContentUnlocked). The NEW badge pops in (`BackOut` over 0.3 s on the first shop frame) with one `Tick`. Cost S. `NewBadge`, an age per offer key.
41. **G41 Sea life, plants and snow answer** (D23, D24, D25). The turtle turns away from splashes, jellyfish pulse, plants bend away from hooks on the seabed and from fleeing fish, specks drift aside from a pulled line. All read the startle list of G3. Cost M. `FCatchLookSea::StepLife`, `DrawPlant`, `Tick`.

### Tier 3: every few days, or once a run

42. **G42 A new rod is seen** (C16, A31). Defer the grow-in until the shop has closed: on the first Prep after a rod was added, `Grow = 0`, `SpawnSplash` at its socket, `Clasp`; the new rod card pops in. Cost S. `ACatchTackleView::SyncRods`, `UCatchHud::UpdateCards`.
43. **G43 A rod levels up** (C17, A30). On the first Prep after a level change: rod `Flash = 1` and `Kick += 6`, 10 gold dots at the tip, `Note` at pitch x 1.26; the new level dot pops. Cost S. `ACatchTackleView::SyncRods`, `PaintRodCard`.
44. **G44 A zone opens** (C8, D28, B ZoneUnlocked). On the first Prep after the unlock: the veil slides down over 1.5 s, the gauge veil line follows, `Rumble` plays again, shake 4 px; the lock label fades. Cost M. `ACatchSeaView::SyncCoreFish`, `UCatchHud::MakeGaugeView`.
45. **G45 The run ends** (C19, B RunOver, D26). Before the panel (0.6 s): the scales slam (`Scales.Kick(2)`), shake 10 px, the target line flashes SoftRed, `Stamp` at pitch 0.7 with `Over`; then the panel rises. NEW RUN gets `Tick` and a 0.25 s exit. Cost S-M. `ACatchTackleView::OnRunOver`, `UCatchHud::OnCoreEvent`, `PaintRunOver`.
46. **G46 KEEP FISHING and NEW RUN** (A40, A41). `Tick` (G2) and a 0.25 s panel exit. Cost S. `UpdatePanelHits`, `PaintVictory`, `PaintRunOver`.
47. **G47 The Gold Sunfish arrives** (D29). Bloom pulse 0.9 -> 1.6 -> 0.9 over 1.5 s and a soft `Chord` on its first frame. Cost S. `FCatchLookFish::Update`.

## 7. Feedback needed by the coming core changes (`day2-core-charms.md`, section 3)

| New moment | Proposed answer (existing material only) | Cost | Where |
| --- | --- | --- | --- |
| A modifier row fires (`on_land`, `on_pull`, `on_escape`, `on_chain_link`, `on_day_end`, `on_shop_enter`, `on_purchase`) | One generic answer for every row: chip wobble + class `Note` (as now) + a popup at the trigger place (fish or bucket for `on_land`, hook for `on_chain_link` and `on_escape`, rod tip for `on_pull`, the shelf chip for shop and day-end rows) with the charm tag and the amount. Coin rows (`coins`): coins fly one by one from the chip to the counter (the unused `FCoin` flight with `bTick` -> `CoinArrive`). In the world: a small burst of `SpawnDots` in the class colour at the same place. Needs the core event to carry rod, hook, catch index and amount for every trigger. | M | `UCatchHud::OnCoreEvent` (CharmFired), `ACatchTackleView::OnCharmFired`, `FCatchLookSound::OnEvent` |
| A counter grows or resets (`counter_add`, `counter_reset`) | A small number on the shelf chip (the count); on growth it pops (scale 1.4 -> 1) and `Note` plays a semitone higher per step (cap 12); on reset it drops to 0 with `Bloop` at pitch 1.3. The tooltip reads the core text query with `{n}` filled. Needs an event or a counter in `ShelfItem` (brief 3.6). | S-M | `PaintCharmChip`, `MakeShelfView`, `PaintCharmTip` |
| A lucky chance comes true (`chance` row, `on_lucky`, Lucky Bite) | The biggest beat of a day: `Chord` + `Link` at pitch x 2, shake 5 px, gold dots at the hook (`SpawnDots` gold), the fish gets the gulp size punch of G13, a large gold popup with the multiplier ("x3"; the word "LUCKY" is text). Fish Tale's chip wobbles at the same time. | M | HUD `OnCoreEvent`, `ACatchTackleView` (new case), sound `OnEvent` |
| Strike chance on a hooked fish (display query, brief 3.22) | A thin red arc outside the gold hang ring of that fish, length = chance; the arc pulses once at each strike check that fails (no event; read the query each frame). On the rod card the third line shows the chance (text, for example "EAT 40%"). In the fish hover card one line (text). | M | `ACatchTackleView::DrawRods` (hang ring block), `UCatchHud::RodCardView`, `UpdateHover` |
| A bigger fish strikes (rarer at about 40 %) | Make HunterStrikes a stronger beat: G12 (prey panics, rod jerk, startle, shake 3 px). `Warn` stays. ChainLink keeps its answer plus G13. | S | `ACatchTackleView::OnHunterStrikes`, `FCatchLookFish::Update` |
| Thin Line (strike x2, hang -2 s) and other plus-minus charms | No new answer: the hang ring is already shorter (from `fullHangSeconds`) and the strike arc above shows the higher chance. | - | - |
| The run opens in the shop (brief 3.23) | The shop rises with a banner over it (`PaintBanner`, existing words "FISHER'S SHOP" and the target of day 1); the starting pack opens there (`UpdateAutoPack` already allows the shop). The shop header "after day %d" (`PaintShop` ~859) would read "after day 0": hide it before day 1 (inferred bug). NEXT DAY before day 1 could read "START DAY" (existing text). | S | `PaintShop`, `UCatchHud::OnPhaseChanged`, `CurrentPanel` |
| A rule day drawn at random (brief 3.24) | Handle `DayRuleAnnounced` (empty everywhere now): when a rule day first enters the two-day window, the forecast plate drops in with `BackOut` and `Chord` at volume 0.5. The full-screen card of 3.8 stays for the day itself. | S | `UCatchHud::OnCoreEvent`, `PaintForecast` |
| New rule `OnlySmallFish` (brief 3.25) | `PaintRuleIcon` has no case for it: add a large fish shape crossed (the CheapFishWorthless icon at 1.4 x). Fish of 10 kg or more: the bucket popup already turns grey for 0 points (`BucketZero`); the hover card shows "0 pts" for such a fish. In the world such a fish could swim greyed (body colour lerped 40 % to grey through the per-instance colour) on that day. | S | `PaintRuleIcon`, `UpdateHover`, `FCatchLookFish::Update` |
| Dynamic shelf slots fall below the charms held (brief 3.10) | Extra chips stand on red dashed slots; a buy is refused with the card shake and `Deny` (exists). When the slot count changes, the shelf slots slide instead of jumping. | S | `PaintShelf`, `MakeShelfView` |
| Day-end coin rows (brief 3.7) | In the summary (3.11, in work): one row per paying charm with its chip and its coins, counted up like the other rows. | M | `PaintSummary`, `OnDaySettled` |
| Rarity of charms (60/30/10) | The charm card border takes the rarity colour (`RarityBorder`, used for baits) and an uncommon or rare card gets the bait card glint. | S | `CharmCard` |

## 8. Top 15 (fix first, in this order)

1. G1 Pressed look on every button and hit: every click in the game answers on press, the owner's first test.
2. G2 Click sound on every panel hit: one wrapper, many silent buttons fixed at once.
3. G3 Fish startle shiver near casts, bites, strikes and landings: the owner asked by name for the fish to react; one mechanism feeds five moments and later the sea life.
4. G4 + G5 The hooked fish fights near the escape, plus the one-shot warning: the owner's "pull now or wait" choice gets a clear moment in the fish itself, not only a blinking ring.
5. G6 Depth handle hover, grab, drag ticks and drop: the main day-time control has no sound and no hover now.
6. G7 A portion goes onto a hook (bait pop on the hook and on the card): happens at every cast and every reload, now in silence.
7. G8 The hovered fish notices the cursor: the player hovers fish all the time; a tail flick makes the sea feel alive for one line of code.
8. G10 Charm fired at its place: today the only answer is at the bottom shelf; the 53 rows of the core brief make this the most frequent event of a built run.
9. G9 Bucket points pop: every fish adds points; the numbers on the scales only change.
10. G11 + G12 + G13 The fish thrashes on the pull, the prey panics at a strike, the hunter gulps: the three big fish moments get motion in the fish, all S.
11. G25 Target reached during the day: the biggest good moment of a normal day has no sound and no word.
12. G29 Coins spent: red flash and "-N" in the unused Fee popup: every purchase.
13. G30 Refusal shake for every shop offer: "not enough coins" now answers with a sound only, except on charm cards.
14. G36 Double `Buy` and `Flip` sounds: a two-line fix of a loudness bug heard on every purchase and pick.
15. G42 + G43 + G44 Growth seen after the shop: new rod, level up, zone opening all happen under the shop panel today and are never seen.

Next after the top 15, in order: G15 raft dip, G14 float pop, G31 upgrade popups, G24 START DAY exit, G27 the day weight lands, G26 late day and dusk, G32 reroll flip.

## 9. Counts

- Table A (41 rows): full 16, partial 21, none 2, in work 2.
- Table B (30 event types): full 15, partial 11, none 1, in work 2, not emitted 1 (FeePaid).
- Table C (23 rows): full 7, partial 13, none 0, in work 2, not in the core yet 1.
- Table D (32 rows): full 11, partial 14, none 7 (D15 counts as full; only its raft part is partial).

## 10. Notes for the orchestrator

- New English texts proposed above that need your approval: "LUCKY", a strike chance line (for example "EAT 40%"), and the reuse of existing strings ("DAY N OVER", "The catch reached the target", "ROD N lv M", "START DAY" on the shop button before day 1).
- G20 (a click on the rod pole pulls) changes a control, not a rule. It is a revocable technical reading; confirm or drop it.
- Real fleeing of nearby fish (from hooks, pulls or strikes) would change which fish can bite. That is a rule and needs the owner. G3 gives the look of a reaction without it.
- Several answers (G24, G33, G45, G46) need a panel to keep painting for 0.25 s after the core phase has changed. One shared "leaving panel" timer in `UCatchHud::UpdatePanels` covers all of them.
- G3, G4, G8, G11, G12, G13 all touch `FCatchLookFish::Update` and fit in one brief for one executor. G14-G18 and G27 all touch `CatchTackleView.cpp` and fit in a second. The interface gaps stay with the HUD executor.
- The other executor is changing `CatchHud.cpp` now; line numbers in this report will move. The brief for the interface gaps should start after that work is merged. The world gaps (`CatchLookFish.cpp`, `CatchTackleView.cpp`, `CatchLookSea.cpp`) do not touch the HUD files.
