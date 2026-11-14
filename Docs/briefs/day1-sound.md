# Brief: day one — the game sounds, made by script

Status: written and handed out on 12.11.2026 at 10:55 (owner's clock). Executor: Opus, effort high. No Unreal Editor in this task.

## 1. Goal

The owner has not heard the game yet. His words: «Звук я пока не слышу». The first playable version has 11 temporary sounds with no design behind them. The look reference now describes every sound of the approved prototype in words and numbers. Make these sounds new, by script, as WAV files that an Unreal executor imports later. Also make two preview files, so that the owner can hear the character of the game today.

## 2. What exists now (verified by the orchestrator)

- The sound table: `Docs/design/look-reference.md`, section 9 (28 sounds: trigger, length, pitch, character, level, gate). The moments that use them: section 8 of the same file.
- The temporary sounds of the first playable version: `Catch/SourceArt/TempSound/make_temp_sounds.py` and its WAV files. Leave them as they are. Another executor works in that part of the project now.
- Python 3.11 with numpy is installed. FFmpeg is on the PATH.

## 3. Decisions of the orchestrator (revocable; say so in the report if you change one)

1. One script makes everything: `Catch/SourceArt/Sound/make_sounds.py`, numpy only, fixed random seed, so that a run gives the same files.
2. Format: WAV, 48 kHz, 16 bit, mono. File names `S_<Name>.wav` with the names of the table in CamelCase (`S_Bite.wav`, `S_CoinArrive.wav`).
3. Sounds whose pitch depends on a count (bucket, coin, link, creak, note): one base file at the lowest step. The game raises the pitch at play time. Write the rule for each one into `Catch/SourceArt/Sound/README.md` (which file, pitch factor per step, cap, repeat interval, gate time, level).
4. Levels: keep the relative levels of the table. Peak of the loudest file at -3 dBFS. No clipping. A short fade at the end of every file, so that no file ends with a click.
5. The table gives the recipe. Where a sound made by the recipe is harsh or dull, you may shape it (filter, a little noise, a second partial) as long as the length, the pitch and the character stay. Say what you shaped.
6. No music and no ambient loop: the prototype has none, and the owner has not asked for them.

## 4. Work

1. The script and the 28 base files.
2. `Catch/SourceArt/Sound/README.md`: the table of files with trigger (the core event), level, gate, pitch rule.
3. `preview_all.wav`: every sound once, in table order, 0.5 s apart, with the level of the table.
4. `preview_day.wav`: about 30 seconds that sound like one short day of play, mixed the way the game would play it. Day bell; a cast and a plop; nibbles; a bite; the creak that gets faster; a hunter warning, a chomp and a link; a pull with the ratchet; a burst; three fish into the bucket, each higher; one escape; the day bell; coins that climb for eight items; the stamp; a purchase jingle; a pack rustle, tear and two flips. Use the timings of look-reference section 8 where it gives them.
5. Listen with numbers, since you cannot hear: for every file print length, peak, RMS and the strongest frequency at the start and at the end. Check them against the table.

## 5. Done means

1. 28 base files, the README and the two previews exist in `Catch/SourceArt/Sound/`.
2. Every file has the length of the table within 10% and the pitch movement of the table.
3. No sample clips. `preview_day.wav` is between 25 and 40 seconds.
4. `python -X utf8 Tools/check_all.py` is green.

## 6. Boundaries

- Files you may create or change: `Catch/SourceArt/Sound/**` and `Docs/briefs/reports/day1-sound.md`. Nothing else. Not `Catch/SourceArt/TempSound/`, not `Catch/Content/`, not `Catch/Source/`.
- Do not open the Unreal Editor, do not run a build, do not import anything. Another executor works in the editor.
- Do not look for the prototype and do not read it. The look reference is the only source.
- Do not download anything. Do not commit, do not push, do not delete files.

## 7. Report

English, `Docs/briefs/reports/day1-sound.md`, at most 20 lines: the table of files with length and peak, what you shaped, what the numbers say about `preview_day.wav`, questions.

## 8. Chain of command

You report to the orchestrator, not to the owner.
