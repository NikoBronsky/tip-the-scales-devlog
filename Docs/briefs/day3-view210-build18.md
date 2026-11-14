# Brief: view 2.10 follow-up 2 - generated pictures for the eight candidate charms, short tags for the ten candidates, playable build 18 (14.11.2026 21:07)

Executor: the same Opus executor that built view 2.10 and build 17, effort high. Worktree `F:\UE\EpicJam_ui`, branch `track/ui`, base `8c40b2d` (build 17, accepted and committed by the orchestrator). You do not commit.

## 1. Goal

The director has not played build 17 yet; he is busy and said "если есть что делать - делай". Two known gaps of build 17 can be closed without any decision of his, so that the build he plays tonight looks closer to final:

1. The eight candidate charms carry script-drawn pictures with a red TEMP tag. The orchestrator had Codex image generation (the director's ChatGPT plan, the same tool and method as all other shop pictures) make real pictures. The director gets the sheet at the same time; a picture he rejects is swapped later.
2. The short word that pops up when a charm fires has no table entry for the ten candidates. Lucky Fuse shows the same word as Lucky Bite, Chain Shot the same word as Food Chain, and the Russian testing layer shows "POWDER" in English.

Result: build 18 in `F:\UE\EpicJam_ui\Packaged\v18\Windows`. Build 17 (`Packaged\v17`) stays untouched. No rule and no number changes; the core stays `694f1b3`.

## 2. What exists (verify before you rely on it)

- The orchestrator already copied the eight pictures into `Catch/SourceArt/Items/` (`powder_keg.png`, `sore_loser.png`, `chain_reaction.png`, `lucky_fuse.png`, `horseshoe.png`, `chain_shot.png`, `shockwave.png`, `bait_bomb.png`) and rewrote their rows in `Catch/SourceArt/Items/items.json` (real sizes, sheets `charms_blast_a` / `charms_blast_b`, no `temp` flag). `git status` in the worktree shows exactly these nine files as modified. The pictures are generated files: decode them only through the existing atlas scripts.
- The atlas and the game asset are NOT rebuilt yet: `Catch/SourceArt/Items/make_item_atlas.py`, then `build_item_art.py` (headless editor), as you did for the TEMP pictures.
- `Catch/SourceArt/Items/make_temp_charm_art.py` stays in the repository (history of the placeholders). Do not run it again: it would overwrite the new pictures. Add one line to its docstring that says so.
- `Ui::CharmTag` in `Catch/Source/Catch/CatchUiPanels.cpp:361` holds the tag table; a charm without an entry falls back to the longest word of its name. The Russian words of the tags are rows of `Docs/design/text-ru.tsv` (for example `LUCKY` -> `ШАНС`, `CHAIN` -> `ЦЕПОЧКА`).
- `Catch/SourceArt/Text/test_text_ru.py:20` still expects 53 charms; the data holds 69.

## 3. Technical decisions (orchestrator; each revocable, say so in the report if you change one)

1. Short tags, English (player-facing): Dynamite `DYNAMITE`, Powder Keg `KEG`, Sore Loser `SORE`, Chain Reaction `REACTION`, Lucky Fuse `FUSE`, Horseshoe `HORSESHOE`, Chain Shot `SHOT`, Shockwave `SHOCKWAVE`, Bait Bomb `BOMB`, Harpoon `HARPOON`. Add all ten to the table explicitly, so that the fallback no longer decides them.
2. Short tags, Russian testing layer: `ДИНАМИТ`, `БОЧКА`, `ОБИДА`, `РЕАКЦИЯ`, `ФИТИЛЬ`, `ПОДКОВА`, `ВЫСТРЕЛ`, `ВОЛНА`, `БОМБА`, `ГАРПУН`. Put them where the other tag words live and regenerate what depends on it. No Cyrillic under `Catch/` beyond what the gate allows.
3. Fix the expected charm count in `test_text_ru.py` only if it is that one constant; if the test fails for another reason, report it and leave it.
4. No other view change. Do not touch the layout of the shelf, the blast, the labels or any number.
5. Launch files of build 18: the same five `.bat` files as in build 17.
6. The director may start build 17 at any moment. Before every heavy step (headless editor, editor build, packaging, a proof game) check for a running `Catch` game process that you did not start. If there is one, do not start the step: tell the orchestrator and wait. Proof games run with `-RenderOffScreen -unattended -nosplash -nosound -log`, one at a time, `-ExecCmds` quoted.

## 4. Work and proofs

Rebuild the atlas and the item asset; edit the tag table and the Russian rows; `python -X utf8 Tools/check_all.py --ue`; package with `Tools\package-win64.ps1` into `Packaged\v18`; then on the packaged build with `-lang=ru -new -coins=300`:

- the first shop: the candidate cards show the new pictures, no TEMP tag anywhere (frame);
- the shelf during a day with several candidates: the small pictures read well on the shelf chips (frame, and a crop of the shelf at full size);
- a day where Powder Keg or Lucky Fuse and Chain Shot or Bait Bomb fire: the popup words are the new Russian words, none in English (frame or log line of the popup text);
- one run without arguments: English, no candidate (frame or log);
- the log of the `-lang=ru` run lists no missing text key.

## 5. Done means

The gate with `--ue` has no fail (if `core-test` times out under load, run `Tools\core-test.ps1` alone and report both); build 18 starts with each launch line; the proofs are in the report with frame paths; `git status` shows only files you meant to change plus the nine files of the orchestrator; never add `Catch/SourceArt/Fish/atlas.png` or `atlas.json`.

## 6. Boundaries

Write only inside `F:\UE\EpicJam_ui`. Always use absolute paths. Do not touch `F:\UE\EpicJam_core` or `F:\UE\EpicJam`. Invent no rule and no number. No downloads, no generator, no account. Never click, close, move or answer a system dialog. Do not stop any process you did not start. Delete nothing; do not touch `Packaged\v14` - `v17`. No commits.

## 7. Report

Add a section "Build 18" to `Docs/briefs/reports/day3-view210-charge-and-links.md`: changes by file, the gate, the proofs with frame paths, decisions you changed, open questions, what is not verified. Final message to the orchestrator: the same in short. Time box: 35 minutes. You report to the orchestrator, not to the director.
