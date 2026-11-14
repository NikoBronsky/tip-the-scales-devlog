# Addendum to core 2.9: balance pass of the four candidates (core 2.9.1)

Written by the orchestrator on 14.11.2026, 10:55, after the report `Docs/briefs/reports/day3-core29-risky-fishing.md`. Same executor, same worktree `F:\UE\EpicJam_core`, base = the commit of core 2.9 that the orchestrator made on `track/core` (your work, unchanged). Everything in the brief `day3-core29-risky-fishing.md` still holds (boundaries, commands, report rules) unless this addendum changes it.

## 1. Why

Your measure shows that the candidates cannot go to the director as they are: Dynamite forced into the first shop takes the careful bot from 23.7 % to 91.0 % alive after day 12, Harpoon to 75.3 %; the two rule days change nothing because the bot lands about 3 fish on such a day. The director's rule for the game: «Игра не должна быть лёгкой». He wants a trade ("gets a lot of fish, but there are fewer fish after"), not a win button. Answers to your questions are in section 2.

## 2. Decisions (each is revocable; numbers are data)

1. **Only a pull that lands a hooked fish triggers Harpoon and Dynamite.** An empty pull does nothing. Reason: a player can cast and pull an empty rod every second; the bot never does. (Count hooked fish at `StartPull`; Cluster Hook extras count as hooked fish.)
2. **Extra fish only from zones the hook may fish.** A fish of a kind whose zone of the day is closed for that hook (`HookFishesZone` of the hook it would land on: the nearest hook for a blast, hook 0 for the harpoon) is not free. A harpoon bounce skips it and looks for the next nearest fish.
3. **Dynamite.** New data number `dynamiteWeight` (start 0.5): a blasted fish weighs x that, credited to Dynamite as a weight multiplier entry (the charm shows in the fish's charm list). Tune by data, in this order, changing as little as possible: `dynamiteWeight` 0.5 -> 0.35; `dynamiteRadius` 130 -> 100 -> 80; `dynamiteSeaFactor` 0.8 -> 0.6. Keep the cost "until tomorrow".
4. **Harpoon.** Tune by data, in this order: `harpoonReach` 260 -> 180 -> 130; then `harpoonFactors` {1, 1.5, 2} -> {0.5, 1, 1.5}; then `harpoonHits` 3 -> 2 (last resort; it changes the rule text).
5. **Target band for both charms.** First measure three strong existing charms forced into the first shop the same way (take the three strongest of the build-strength table of balance pass 1; if you cannot find it, measure Food Chain, Cluster Hook and Giants). The band: day-12 survival of the candidate, forced, is at least 5 points over the no-charm run and at most 10 points over the strongest of those three. Stop at the first setting inside the band. Report the table of every setting you measured.
6. **Poachers.** New data number `poachersSea` (start 0.35): on a Poachers day the sea factor of the day starts at this value (the surplus fish leave when the day is prepared, by the existing surplus rule), and no new fish swim in (value 0 stays). With Storm Glass: nothing happens. The Dynamite factor multiplies on top.
7. **Pest.** Value 3 -> 15 (percent per landed fish), `pestFloor` 0.5 -> 0.4.
8. **Target for both rule days.** Measure the hardest existing rule day as the reference: force each existing non-light rule (Nothing bites in Sunlight, Fish lighter than 10 kg, Only small kinds) on day 3 and day 6 through the seam and take the one with the lowest share of runs that pass those days. Tune `poachersSea` (0.5 / 0.35 / 0.25) and the Pest value (10 / 15 / 20) so each new rule costs about as much as that reference (pass share within 10 points of it, on day 3 and on day 6). Report the table.
9. **Rule texts follow the final numbers.** Write them in the game's plain style and keep them short. Starting forms:
   - Dynamite: `A pull that lands a fish blasts the water: every fish near this rod's hooks lands too, at half weight. After each blast the sea holds 20% fewer fish until tomorrow.`
   - Harpoon: `A pull that lands a fish also throws a harpoon at the nearest fish. It bounces to 2 more fish nearby. The 2nd fish weighs x1.5, the 3rd x2.`
   - Poachers: `Poachers: two thirds of the fish are gone. No new fish swim in today.`
   - Pest: `Pest: each fish you land makes the next ones 15% lighter, down to 40% weight.`
   Update `ruleRu` of the two charms in the data file to match (Russian, the same plain style; "x1,5" with a comma). Put the final English and Russian texts of all four into the report, one block, so the orchestrator can copy them.
10. Your other choices stand: an extra fish keeps its own depth as catch depth; the Gold Sunfish counts as landed for Pest; Poachers lasts the whole day number; one `CharmFired` after each event; the offer seam offers no candidate with the switch off; the prices 8 and 6.
11. Version: core 2.9.1, API stays 9 (additions only: the two data numbers).

## 3. Done means

- Tests updated and extended for decisions 1, 2, 3, 6, 7; all tests pass; both fingerprints unchanged with the default tables (`b0d881d44f065645`, `7d632f58e4dd608e`); the survival table with candidates off still equals 2.8.1.
- `python -X utf8 F:\UE\EpicJam_core\Tools\check_all.py --ue` prints `11 ok, 0 fail, 0 skip`.
- The report gets a section "2.9.1 balance pass": the measure tables, the final numbers, the final texts (English and Russian), what the bot does with each candidate, what you could not verify.
- If a candidate cannot reach its band with the listed steps, stop at the last step, say so, and give the numbers; do not invent a new rule.

Final message to the orchestrator: at most 20 lines. No commits, no pushes.
