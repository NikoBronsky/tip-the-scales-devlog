# Day 1 / track 1 delivery — 12 November 2026, 09:03 UTC+1
Status: blocks 1–9 implemented and standalone-tested; Unreal integration acceptance remains pending.
Blocker: game-target builds were attempted after block 3 and at the end. Both stopped making visible progress after `Running UnrealBuildTool: dotnet ...`; each was interrupted after about two minutes (exit 1). No compiler diagnostic or permission error was returned; the cause is unconfirmed. No sandbox workaround was attempted.
Orchestrator must run: `& "F:\UE_5.9\Engine\Build\BatchFiles\Build.bat" Catch Win64 Development -Project="F:\UE\EpicJam_core\Catch\Catch.uproject" -WaitMutex`.
Works: preparation, six-zone fish population/schools, rods/hooks, all 12 baits, escape/chain/pull, per-fish bucket events, stored-price shipping, three fee levels, loss/reset and all five special rules.
Works: upgrades, fish cards, packs/picks, shelf/Weightless/resale/rerolls, unlock order and all 20 charms. Prices and local charm contributions are retained for shipping.
Tests cover the requested timings, bait matrix, three eaten links, every charm, multi-hook/cluster landings, shipping equality, fees/unlocks, 10,000 random commands with equal-seed event equality and invariants, and 1,000 entities.
No missing standalone block; no visual/editor/runtime-in-Unreal validation. Individual shipping rows meet the cap; optional same-kind batching is not implemented.
Final standalone command: `powershell -NoProfile -ExecutionPolicy Bypass -File Tools/core-test.ps1` (both test and runner executables compile with the brief's MSVC flags).
```text
PASS block 7: all five special rules, Gold Sunfish reward, active pull completes at day boundary
PASS block 7: cluster chain counts every eaten fish; Thrifty Hook rolls once per whole cast
CORE TESTS PASSED
core-test: exit 0; 13,60 seconds
```
Final `python -X utf8 Tools/check_all.py` tail (other skip: no submission archive):
```text
CORE TESTS PASSED
core-test: exit 0; 13,78 seconds
skip  ue-build           run with --ue to compile the editor target
9 ok, 0 fail, 2 skip
```
Runner: `Catch\Saved\CoreTest\core_runs.exe 10 9`, exit 0; 60 runs across both policies and all three fee levels. On Normal, day-4 survival was 0% saving / 40% buying; both were 0% after day 5. This measures these simple policies, not player balance. Last lines (buying / Hard):
```text
7 0 0.0 n/a 3650
8 0 0.0 n/a 9150
9 0 0.0 n/a 22900
CORE RUNS COMPLETE
```
Rule questions / provisional readings (details and data locations in core-interface.md):
- Confirm updated Deeper Line prices from section 11 over section 6; initial money 0; initial goal 150; linear rod speed `1 + 0.2*(level-1)` and fractional-zone travel.
- Confirm species are split equally within each zone, daily population grows multiplicatively by 8% (cap x2), unique fish stay unique, and the empty floor gap overrides hunter-band extension. Movement defaults use 80-unit speed/reach and a 1-second strike; all movement parameters are in PaceData.
- Confirm a pull already started completes at day end, unpulled fish are lost, and Same Again continues across days. A cluster shares one hang timer; a hunter eats every member and counts every eaten fish toward the chain.
- Confirm the documented price order, downward money rounding, and day rules applying to Gold Sunfish too. Special rules currently use uniform seeded selection; its half-fee reward is the base before price modifiers.
- Confirm distinct pack choices, no offers of already-owned charms, and the current bait rules: selected bait changes next cast; unused portions stay reserved; reusable lures survive landing; Siren Pearl survives escape; Thrifty Hook rolls once for the entire cast.
Section 3 decisions changed: none. ApiVersion remains 1; all frozen names/units remain. Added payload fields and exact C++ signatures are documented in the interface note.
Track 2: include `CatchCore/Game.h`; send commands, Advance, DrainEvents, then GetState. Resolve names from Tables. FishId is not a vector index; rebuild fish mirrors on a new day. Use one flight per FishLanded and the stored charmAmounts on ShipItem.
Evidence: `Catch/Saved/CoreTest/final-tests.txt`, `final-check-all.txt`, `final-runs.txt`; standalone outputs are ignored build artifacts. MSVC batch failure detection handles negative process exit codes too.
Scope: only the brief's source/document/script allowlist and specified build-output folder were written; no editor, commits, pushes or file deletions. Orchestrator owns DEVLOG/journal updates and the remaining Unreal build gate.
