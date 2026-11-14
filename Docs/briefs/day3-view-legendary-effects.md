# Brief: very beautiful effects for Legendary baits and charms, build 21 (14.11.2026 22:25)

Executor: the same Opus executor that built builds 17 - 20, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`. You do not commit. You start only after the orchestrator has committed build 20.

## 1. Goal

The director wrote at 22:23 (voice input; "нашивок" is read as "наживок", baits):

> "нам нужны очень красивые эффекты для легендарных нашивок и модификаторов"

Decision `legendary-items-beautiful-effects`. A Legendary item is the rarest thing the player can get. Today it differs from a Rare one only by the metal of its frame. After this task the player notices a Legendary item at once, three times: when it appears, when it works, and while it rests on the shelf or on the hook. This is the main pillar of the game ("Главное — ощущение и полировка. Каждое действие игрока отвечает красиво: движением, звуком, эффектом."), so the bar is high: the director judges every frame, and he asked for "очень красивые".

## 2. What exists (verified by the orchestrator unless marked; verify the rest before you rely on it)

- Five Legendary items. Baits: Golden Grub, Siren Pearl (`F:\UE\EpicJam_ui\Catch\Source\CatchCore\Private\Tables.cpp`, the bait rows with `Rarity::Legendary`). Charms: Trotline, Upside Down, Tiny Terror (`CharmRows.inc`, three rows with `Rarity::Legendary`; all three are `unconditional`). All five are in the default game, not behind a switch.
- The rarity frame: `Ui::PaintRarityFrame` / `Ui::PaintCardFrame` (`Catch/Source/Catch/CatchUiPanels.h:179-186`). Legendary is gold "with a slow glint that runs round it". Rarity colours: `Catch/Source/Catch/CatchUi.cpp:102-115` (Legendary `0xffcf4a`, dark `0xa8740a`), `Ui::RarityMetal`.
- A moving band of light exists: `Ui::PaintBand` (`CatchUiPanels.h:170`).
- The view has no Niagara and no particle system in its source (grep for `Niagara`, `UParticle`, `SpawnEmitter` finds nothing); interface effects are drawn by the painter. Not verified: how the sea draws its effects (the blast, the splash, the bucket) - you built them, use the same means.
- A cheat run for frames exists: `FrameRun(Seed, Names)` (`CatchSession.h:60-64`): the shop offers only the named charms and the packs only the named baits, with spare coins.
- Not verified by the orchestrator: which core events name a Legendary charm when it does its work (all three are passive), where a pack shows its bait cards, how a bait looks on the hook in the sea. Read the code and write the facts into the report.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. Three layers, built and proven in this order. Each layer is complete on its own; if the time box ends, the finished layers ship and the report says which layer is missing.
   - Layer 1, appearance. A Legendary charm offer in the shop (and on the reward and unlock cards, if they share the card painter cheaply) and a Legendary bait card in a pack arrive with a reveal: a short golden flash, then slow rays of light that turn behind the picture and fine gold dust that rises from the card while it stays on screen. Buying it gives a burst that follows the item to its place (the shelf or the bait box). One new sound for the reveal and one for the purchase.
   - Layer 2, work. When a fish is caught on a hook that carries a Legendary bait: a golden burst at the hook, the fish flies to the bucket with a golden trail, and its coin pop-up at the sale is gold. When a Legendary charm does its work in a way the player can see (decide from the events what "its work" is for each of the three; if a charm has no visible moment, say so and give it only layers 1 and 3), the shelf chip answers with a golden pulse.
   - Layer 3, rest. The shelf chip of a Legendary charm, the bait box tile of a Legendary bait, and the Legendary bait on the hook in the sea carry a calm shimmer: a slow glint and a rare small sparkle. Calm means: it must not pull the eye away from the rods during a day. A period of several seconds, small amplitude.
2. Only Legendary gets these effects. Rare, Uncommon and Common stay as they are. Do not restyle the frames.
3. Means. Use what the view already uses (the painter for the interface, the existing means of the sea for the sea). No asset from outside, no download, nothing from the browser prototype. A new sound comes from the project's own sound script (`Catch/SourceArt/Sound/make_sounds.py`) and is listed in the report so that the orchestrator declares it in DEVLOG.
4. One place for the numbers of the effects (durations, counts, sizes, colours), and one switch that turns all Legendary effects off. No game rule and no game number changes; the core is not touched.
5. No new player-facing text.
6. Cost. Measure the frame time of one fixed heavy scene before and after (a windowless run, the same seed and commands); the effects must not cost more than 1 ms on that scene. Report the two numbers.
7. A launch file for the director: `Catch_RU_legend.bat`, Russian layer, a run in which the first shop offers the three Legendary charms and the first pack offers the two Legendary baits, with spare coins. Use `FrameRun` or the smallest testing-only argument that reaches it. A testing aid; it never goes into the submission archive.
8. Package into `F:\UE\EpicJam_ui\Packaged\v21\Windows` with all launch files of build 20 plus the new one.

## 3a. Addendum after phase 0 (14.11.2026 23:13; orchestrator, revocable)

The executor read the code first and wrote the plan (report section "Legendary effects (build 21) - plan"). The orchestrator accepts the plan with these answers:

1. Soft glow. Allowed: one soft radial glow picture drawn by a script of the project (numpy / Pillow, like the other script-drawn pictures) and imported the same headless way as the item atlas, used as a brush by the painter. No open editor, no outside asset. A material asset only if the picture route fails; say so in the report. The script and the asset are listed in the report so that the orchestrator declares them in DEVLOG.
2. Tiny Terror gets layers 1 and 3 only. The view does not copy the hunter ladder rule of the core to guess its strikes.
3. The time box stays 90 minutes. Cut order when the time runs short, first to last: the third sound, the gold coin pop-up at the sale, the sea ring for Upside Down, the unlock cards. The reward cards stay as long as they can share the helper of the shop card: a Legendary charm most often reaches the player as a reward, so that is where the reveal matters most.
4. Facts from phase 0 that the build relies on: Upside Down sends `CharmFired` with slot -1 at day preparation; Trotline has no event, its moment is `FishLanded` from a hook with an index below `rod.extraHooks`; `LandedFish` carries no bait, so the view remembers the bait of each hook at `FishHooked` (Golden Grub is used up on the bite). If that memory is wrong, an effect lands on the wrong fish; it must never change a catch.
5. The executor's note for the submission: `Tools/make_zip.py` drops `.pdb` files and `Saved` but not `.bat` files, so the submission archive must come from a clean package without the launch files. The orchestrator owns that step.

## 3b. Added task: the rarity of a bait must read at a glance (14.11.2026 23:15; decision `bait-rarity-background`)

The director, while playing build 19:

> "Я смотрю на рамки наживки, обычная от необычной почти не отличается, может фон будет отличаться?"

He sees that a Common bait and an Uncommon bait look almost the same: Common has no frame and Uncommon has a thin copper rim, which is lost on a small tile. He proposes that the background differs. Build this FIRST, before the Legendary layers: it is cheap and it is his direct complaint.

1. Where a bait is shown as a tile, a chip or a card - the bait box tile (`PaintBaitBox`, `CatchUiSide.cpp:181`), the bait card of a pack (`PaintBaitCard`, `CatchUiPanels.cpp:704`), the bait chip on a rod card (`CatchUiHud.cpp:442`) and any other place you know - the background carries the rarity. Common keeps today's background. Uncommon, Rare and Legendary each get their own background tone, taken from the rarity colours the game already uses for the frames, strong enough that Common against Uncommon reads at once on the smallest tile at 1080p. The frame stays.
2. The text and the picture on the tile keep their contrast. The states that already use the background or the border (a selected bait, an empty box, a bare hook, a blink) stay readable; say in the report how each state looks on each rarity.
3. Charms are not part of this task: their cards have the class band and the frame. If the same weakness shows on the shelf chips, say so in the report and change nothing.
4. Proof: one frame with baits of all four rarities side by side in the bait box and one with them in a pack, plus a 2x crop of the tiles, on the packaged build. The orchestrator shows the crop to the director.
5. No new text, no rule, no number of the game.

The time box of the whole task becomes 100 minutes; the cut order of 3a.3 stands and applies to the Legendary layers only.

## 3c. Added task: the eleven new charms enter the default game (14.11.2026 23:18; decision `all-eleven-candidates-enter-default-game`)

The director, asked which new charms enter the default game, answered before the rule freeze:

> "Все одиннадцать"

1. The default game (no launch switch) offers the eleven candidate charms by the normal shop rules of rarity, unlocking and needs - exactly what `-newmix` does today for the charms. `-new` stays a testing aid (fixed offer order). Add a testing switch `-nonew` that plays the old default game without the candidates, for comparison runs.
2. The one switch `Tables::candidates` also turns on two candidate day rules, Poachers and Pest (`Days.cpp:34`, `DayRuleData::candidate`), and changes one offer rule (`Offers.cpp:165-167`). The orchestrator asked the director about the two day rules; his answer at 23:20 (decision `poachers-and-pest-enter-default-game`): "Да, оба входят". So the default game is exactly what `-newmix` is today: the view sets `Tables::candidates` on by default, nothing is split, no core file changes.
3. Proof: a default run without switches whose log shows the candidate line and a candidate charm among the shop offers within the first shops; `-nonew` gives the event log of build 20's default game on the same seed.
4. Launch files: `Catch_RU.bat` is now the game with the new charms; keep the others; add `Catch_RU_nonew.bat`.
5. Defect C, one more proof scene: the director sent a second screenshot from build 19 with two tuna on one line and the same patch on the head of the lower one. Add one before/after pair with two tuna on one line (build 19 against the new build) to the proofs.

Order of the whole task: 3b (bait rarity background), 3c, then the Legendary layers.

## 4. Done means

1. `python -X utf8 Tools/check_all.py --ue` in `F:\UE\EpicJam_ui` has no fail.
2. Proof on the packaged build 21, windowless. A still frame does not show motion, so each effect is proven by a short sequence of frames from the running game (at least 6 frames across the life of the effect, at a fixed frame step), put side by side on one sheet per effect: the reveal of a Legendary charm in the shop; the reveal of a Legendary bait in a pack; the purchase burst; a fish caught on a Legendary bait from hook to bucket; the shelf chip, the bait box tile and the hook at rest. Sheets and frames in the build's `Catch\Saved\ue_mcp\`.
3. A comparison frame: one shop with a Rare and a Legendary offer side by side, so the difference is plain.
4. The default game without any switch starts, plays one day and exits with code 0 in your own windowless smoke run; the two frame-time numbers of decision 6.
5. `git status` shows only files you meant to change. `Catch/SourceArt/Fish/atlas.png` and `atlas.json` stay untracked and untouched.

## 5. Verification commands

```
python -X utf8 F:\UE\EpicJam_ui\Tools\check_all.py --ue
powershell -File F:\UE\EpicJam_ui\Tools\package-win64.ps1 -Project F:\UE\EpicJam_ui\Catch\Catch.uproject -OutDir F:\UE\EpicJam_ui\Packaged\v21
```

Proof game pattern (no window, no sound, no focus taken), as in your earlier proofs:

```
Catch.exe -RenderOffScreen -unattended -nosplash -nosound -ResX=1920 -ResY=1080 -benchmark -fps=30 -log -abslog=<log> -ExecCmds="<commands>" -lang=ru
```

## 6. Boundaries

- Write only inside `F:\UE\EpicJam_ui`. Always absolute paths. Do not touch `F:\UE\EpicJam`, `F:\UE\EpicJam_core`, `F:\UE\EpicJam_submit`. Do not edit core files.
- Invent no rule and no number of the game. An effect shows what the game already does; it never changes what happens. If an effect seems to need a new rule or a new word, write the question down and build the part that needs none.
- Theme rule of the jam entry: no director, cinema or "cut" wordplay; no logos. All player-facing text stays English; Russian only in the testing layer behind `-lang=ru`. Do not change the gate's `english-only` check.
- Do not copy the look of an effect of a specific existing game. Gold light, rays, dust and sparkles are common means; build ours from the game's own shapes and colours.
- Before you compile, package, or start a game: check for a running `Catch` process (`Get-CimInstance Win32_Process -Filter "Name='Catch.exe'"`, read its command line). A game without `-RenderOffScreen` is the director's game: do not build, do not start anything, do not stop it; finish the code, report "code ready, not built" and hand back.
- One game instance at a time. Never stop a process you did not start. Never click or answer a system dialog.
- Delete nothing. No commits. No downloads.

## 7. Report

Add a section "Legendary effects (build 21)" to `F:\UE\EpicJam_ui\Docs\briefs\reports\day3-view210-charge-and-links.md`: the facts you found (events, painters, the sea), what changed by file, the table of effect numbers, the new sounds, the launch file, the proof sheets with their names, the two frame-time numbers, which layers are complete, what is not verified, open questions. Final message to the orchestrator: the same in short, blockers first. Time box: 90 minutes of work. You report to the orchestrator, not to the director.
