# Addendum 3 to the brief core 2.5: a second mode, the day on time

Written by the orchestrator on 14.11.2026. It extends `Docs/briefs/day3-core25-fish-reward.md`. Same worktree, same boundaries, same report file (add a section "Timed mode"). Do it last, after the refit of the default mode.

## 1. Goal

Until core 2.3 the day ran its full time and the player caught as much as possible. Since 2.3 the day ends when the target is met. The owner likes both and wants the old one back as a separate mode.

Owner, decision `timed-mode-approved`, 14.11: «хотелось бы все-таки добавить режим отдельный для игры именно на время, чтобы вот как до этого было. ... мне как бы оба варианта нравятся, просто вот этот вариант, где тебе надо обычное количество набрать, медленный, размеренный, как Балатро. А тот вариант был именно вот на скорость, на раш, на азарт, и ты не замечал, как ты там большое количество рыбы возил.» Then: «Да, делай отдельный режим.»

## 2. What exists now (verify before you rely on it)

- The day ends at the target: `Private/Game.cpp:165` (`if (TargetMetNow()) { ReachTarget(state_.secondsLeft); EndDay(); }`), `Private/Days.cpp:92`.
- The coins for points over the target still exist and are switched off by data: `Public/CatchCore/Tables.h:323-332` (`overCap = 0; off since 2.3 (was 4)`), `Private/Prices.cpp:82`.
- The day length, the targets and the time bonus of 2.3 and later: the pace and pay tables. The values before 2.3 are in the git history of `Private/Tables.cpp` and in the older core reports under `Docs/briefs/reports/`.

## 3. Technical decisions (each is revocable; say so in the report if you change one)

1. A run has a mode, chosen when the run starts: the default mode (the day ends at the target, as now) or the timed mode. The mode is data of the run; it is in the state so the view can read it. Starting a run without a mode gives the default mode, so every existing test and bot keeps its result.
2. The timed mode restores the rules before 2.3: the day runs its full time; reaching the target does not end the day; at the end of the day a met target passes the day and a missed target ends the run; points over the target pay coins by the old formula with the old cap; there is no time bonus.
3. The day length of the timed mode is the day length the game had before 2.3. Take it from the git history and name the commit in the report.
4. Everything else is the same in both modes: fish, bait, charms, shop, the prize for the Gold Sunfish, the day rules.
5. The timed mode has its own target table, fitted by bot runs to the same goals as the default mode. Do not touch the default table in this block.
6. Charms and day rules that speak about the end of the day or about time (for example a rule that makes the day shorter, a charm about the last seconds, the time bonus) must stay true in both modes. List each one and how it behaves in the timed mode. If one makes no sense there, report it as a question; do not invent a new rule.
7. Tests: the default mode is unchanged (same event logs as before for the same seed); in the timed mode the day does not end at the target, pays for points over the target, and ends the run on a missed target; determinism in both modes.
8. Interface changes (the start command with the mode, the state field, new or changed events) go into the report.
