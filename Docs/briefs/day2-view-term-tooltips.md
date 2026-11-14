# Brief: view, part 7 - marked terms with a tooltip; the Russian rule texts

Executor: Opus, effort high. Written by the orchestrator on 13.11.2026, about 19:05 (owner's clock, UTC+1).
Worktree: `F:\UE\EpicJam_ui`, branch `track/ui`. Base: the commit the orchestrator names in the launch message (view parts 5 and 6 are committed in it). This brief lives only in the main checkout.

## 1. Goal

A player who does not know a word of the game can point at it and read what it means. The terms in rule texts are marked; the cursor on a term shows a small tooltip. A player who knows the word never has to read it.

Why it matters. The owner (game director), decision `keyword-tooltips`, 13.11, playtest of build 9: «Еще надо в тексте прям выделять какой-то термин, чтобы игрок мог навестись и увидеть всплывающую подсказку. Например, что значит цепочка, рыба с цепочкой. Вот, что такое зона и так далее. Надо, что это было удобно. Чтобы если игрок не хочет, он просто не будет курса наводить на эту надпись. Но если он не понимает, о чем речь, то он может и навестись.»

Second note, decision `free-feast-text-unclear`: he could not understand the Russian text of the charm Free Feast («На крючке с редкой или легендарной наживкой рыбу ест любая рыба крупнее.»). The English text is fine. The Russian text is his testing aid, so a bad Russian row costs him playtest time.

## 2. What exists now (verify before you rely on it)

Paths are under `F:\UE\EpicJam_ui\`.

- Rule texts of charms come from the core (English). They are shown on the shop charm cards, the reward cards, the shelf tip (`Ui::PaintCharmTip`, `CatchUiSide.cpp:315`), the shop hover card (`Ui::PaintShopTip`, `CatchUiPanels.cpp:1130`), the sea hover card (`Ui::PaintHoverCard`, `CatchUiSide.cpp:409`), the pack cards, the gift cards, the rule card of the day (`Ui::PaintRuleCard`, `CatchUiHud.cpp:532`).
- The text layer and the Russian testing table: `Catch/SourceArt/Text/gen_text_ru.py` (`--check`), `Docs/design/text-ru.tsv`, the Russian names and rules of charms in `Docs/design/modifiers-v2.data.json` (`nameRu`, `ruleRu`). Report of the text layer: `Docs/briefs/reports/day2-text-language.md`.
- The rules the terms name live in the core: `Catch/Source/CatchCore/**` (read only), the content list `Docs/design/content-list.md`, the core reports `Docs/briefs/reports/day2-core-charms.md`, `day2-core23-gift-dayend.md`, `day2-core24.md`.
- The helpers of part 6 (the cursor light, the tilt, the shared hover answers): report `Docs/briefs/reports/day2-view-settings-motion.md`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. The glossary is one table in the view: term, the word forms that match it in a text, the tooltip text. Matching is by whole words in the shown text, so a new charm text needs no markup. Start from the words the texts really use; candidates: chain, chum and chummed kind, shoal, card level, zone and the zone names, rod level, hook, bare hook, escape, pull, weight, points per kg, the bait rarities, weightless, shelf slot, day rule, Sunfish day, target, larger fish and "eats the hooked fish". Drop a candidate that no text uses.
2. Every tooltip text explains a rule that the core already has. Write it from the core code, not from memory, and put the source (file and line) next to each row in the report. Short: one or two sentences, at most 140 characters, plain words, the same term the texts use. If you cannot find the rule in the core, leave the term out and write the question into the report. Do not invent a rule or a number.
3. The mark. A term in a text is drawn in an accent colour with a thin dotted underline. It must not make the text harder to read; at most the terms of the glossary are marked, once per term per text.
4. Texts that are always on screen (shop cards, reward cards, pack cards, the rule card): the cursor on a marked term shows the tooltip next to the term, after about 0.2 s, with the same entrance as the other hover cards.
5. Texts inside a hover card (the shelf tip, the shop hover card, the sea hover card): the cursor cannot reach them today. Choose the simplest reliable way and say which you chose: either the hover card stays while the cursor moves from the item into the card, or small term notes appear beside the hover card after the cursor rests on the item for about 0.7 s. The player who moves on never sees them.
6. One tooltip at a time. It stays inside the screen. It never hides the term it explains.
7. Russian. Every tooltip text gets a Russian row. Fix the Russian rule of Free Feast (for example: «На крючке с редкой или легендарной наживкой рыбу на крючке может съесть любая рыба крупнее.»). Then read all Russian charm rules against the English ones and fix every row that is unclear, wrong or uses two words for one thing; list the changed rows. Change only `nameRu` and `ruleRu` in `Docs/design/modifiers-v2.data.json` and rows of `Docs/design/text-ru.tsv`; regenerate.
9. The hover tip of a reward card covers the picture of the neighbour card today (frame `Catch/Saved/ue_mcp/items_en_1920_reward.png`). Place every hover tip so that it never covers another card of the same row: above or below the row, or over the dimmed background. Check the shop cards, the reward cards, the pack cards and the shelf. Second case, found on 13.11 at 20:13: the hover tip of the second shop charm card covers the Small Pack tile of the BAIT column (frame `Catch/Saved/ue_mcp/text_living_en_1920_tile_left.png`). A shop tip may cover the empty part of the panel or the dimmed sea, never a tile or a card that the player can buy.
10. Two leftovers of part 6, last, only when blocks 1-5 are done. (a) Measure the Gold Sunfish with `catch.HookMeasure` (it did not bite in the bite runs: make it bite inside the cheat run, a cheat only) and add its row to the report; fix it only if it is over 3 px. (b) The rim (`Fx::Rim`) is faint on the cream cards (charm, pack, reward): make it readable there without hiding text, for example a brighter highlight on the near edge and a soft shade on the far edge. Show one frame before and after.
8. Proof. A proof mode `terms`: a shop card with marked terms; the tooltip of "chain" and of a zone name; a hover card with its term notes; the same in Russian; one frame at 1280x720.

## 4. Work by blocks (in this order; extend the report after each block)

1. The glossary from the core, with sources.
2. The mark in texts and the tooltip on always-visible texts.
3. Terms inside hover cards.
4. Russian rows, the Free Feast rule, the review of the Russian charm rules.
5. Proof frames and checks.
6. Decision 10 (the two leftovers of part 6).

## 5. Done means

- `python -X utf8 Tools/check_all.py --ue` prints `10 ok, 0 fail, 1 skip`.
- `python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check` passes.
- The proofs pass with 0 errors and 0 missing keys: `run_text_proof.py plain en 1920`, `flow en 1920`, `reward en 1920`, `terms en 1920`, `terms ru 1920`, `screens ru 1280`.
- The frames of decision 8, looked at by you.
- The report holds the glossary (term, forms, English text, Russian text, core source) and the list of changed Russian rows (before and after).

## 6. Verification commands (run from `F:\UE\EpicJam_ui`)

```
python -X utf8 Tools/check_all.py --ue
python -X utf8 Catch/SourceArt/Text/gen_text_ru.py --check
python -X utf8 Catch/SourceArt/Text/run_text_proof.py plain en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py flow en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py reward en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms en 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py terms ru 1920
python -X utf8 Catch/SourceArt/Text/run_text_proof.py screens ru 1280
```

Run game proofs one at a time. Builds use `-WaitMutex`. Do not kill a process you did not start; do not touch `Packaged/`. If the gate reports a time-out of the core tests while the machine is busy, run `Tools\core-test.ps1` alone and report both results.

## 7. Boundaries

- You may change only files under `F:\UE\EpicJam_ui`: `Catch/Source/Catch/**`, `Catch/SourceArt/**`, `Docs/design/text-ru.tsv`, the fields `nameRu` and `ruleRu` of `Docs/design/modifiers-v2.data.json`, and your report file. Use absolute paths.
- Do not change `Catch/Source/CatchCore/**`, `Catch/CoreTests/**`, any English rule text, `Catch/Source/Catch/Catch.Build.cs`, `Catch/Config/**`, the level, the assets under `Catch/Content`, `CLAUDE.md`, `AGENTS.md`, the journal, `DEVLOG.md`, the check gate, permission settings.
- Do not invent rules or numbers of the game.
- No downloads, no generated pictures, no fonts, no new sound files.
- Do not delete or hide old work. No commits, no pushes.
- All player text is English; no Cyrillic under `Catch/`. No director, cinema or "cut" wordplay.

## 8. Report

File: `F:\UE\EpicJam_ui\Docs\briefs\reports\day2-view-term-tooltips.md`, in English. Create it right after reading, with one status line per block, and extend it after each block. Sections: what changed by block, the glossary with sources, the way chosen for hover cards, the changed Russian rows, frames with paths, checks with their output lines, not checked, decisions changed, open questions. Your final message to the orchestrator: a summary of at most 30 lines.

## 9. Chain of command

You report to the orchestrator, not to the game director. Write questions into the report; the orchestrator answers them. Do not wait for an answer if other work remains.
