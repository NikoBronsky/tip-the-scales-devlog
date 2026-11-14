# Addendum 2 to brief day3-core210-link-charms: the dynamite is a thrown charge with a random path (14.11.2026 15:08)

Executor: the same Opus executor, effort high. The brief and addendum 1 hold except where this file says otherwise. You do not commit.

## 1. The director's words

In answer to addendum 1 point 2 ("a blast takes only fish of chummed kinds"): "Нет, взрыв должен лететь по рандомной траектории, биться о стенки и взрываться через случайны промежуток времени, он может никого не задеть, а может задетьмногих". And two minutes later: "Пусть взрыв удаляет рыбу хоть полностью".

The orchestrator's reading: the dynamite is a charge that is thrown into the water, travels on a random path, bounces off the walls of the sea and explodes after a random time. It may hit nobody or many. It takes any fish, chummed or not. Its price is the permanent downgrade of the kinds it took, and that downgrade may go all the way: a kind can vanish from the sea.

## 2. Technical decisions (orchestrator, temporary balance, each revocable; every number is data)

1. **Revoked:** addendum 1 section 2 point 2. A blast takes every free fish in its radius by the 2.9 pick rules, whatever its chum level.
2. **The charge.** A blast is no longer instant at the rod. Its source (Dynamite: a pull of the player or Auto-Reel that lands a fish; Lucky Fuse: a lucky chance that comes true) throws one charge: a new entity in the state (`State::charges`: id, x, depth, velocity, fuse seconds left, source charm, rod). Expose the list read-only the way the fish list is exposed, so the view can draw the charge every frame.
   - Start: the x of the throwing rod, at the surface.
   - Direction: uniform random between `chargeAngleMin` = 20 and `chargeAngleMax` = 160 degrees below the surface line. Speed `chargeSpeed`: choose a value with which the charge crosses about one zone of depth per second. Fuse: uniform random between `chargeFuseMin` = 0.8 s and `chargeFuseMax` = 3.0 s. All draws from the core rng, so a seed repeats.
   - Movement at the fixed step: a straight line with mirror bounces off the left wall, the right wall, the surface and the seabed (the bottom of the last zone, open or not). No gravity, no drag.
   - Events: `ChargeThrown` (charge id, rod, source charm, x, depth, velocity, fuse), `ChargeBounced` (charge id, x, depth) at every bounce.
3. **The explosion.** When the fuse ends, the charge explodes where it is: every free fish within `dynamiteRadius` of that point lands at once through the landing-outside-a-pull path of the brief (decision 1; rod = the throwing rod; weight factor 35 %; credited to the source charm). Event `Blast` carries the point (x, depth), the charge id, the count and the fish ids, and comes before their `FishLanded` events. A blast that reaches no fish still emits `Blast` with count 0, so the view can show it. The charge then disappears.
   - A charge that is still flying when the day ends is removed without an explosion. The automatic pulls of the day end throw nothing.
4. **The price** (replaces addendum 1 section 2 point 1 only in its lower bound). Every kind the blast took at least one fish of loses 1 owned and 1 active chum level, at once, for the rest of the run, and the level may go below 0. A negative level L multiplies the wanted count by `max(0, 1 + chumGrowth x L)`: level -1 is half the fish, level -2 is none. The lowest level is the one where the factor reaches 0 (`chumMinLevel`, -2 with today's growth of 0.5). At that level the kind has no fish: the "at least 1" floor of `Game::WantedCount` does not apply to it, no fish of it enters, the ones in the sea stay until caught or gone.
   - The shop offers chum for a kind with a negative level like for any kind below the maximum; the price never goes below `chumBasePrice`. Buying a level works as today (from the next prepared day, or as chum works now).
   - A kind counts as "chummed" for every charm and condition only at level 1 or more; a negative level reads as 0 there.
   - Prize kinds, the Gold Sunfish and `unique` kinds are never downgraded (they are not picked by a blast anyway; add the guard).
5. **Rule texts.** Dynamite: `A pull that lands a fish throws a charge of dynamite. It bounces off the walls and blasts after a random time: every fish near the blast lands too, at 35% weight. Each kind it takes loses 1 chum level for good.` Russian: `Рывок с рыбой бросает в воду заряд динамита. Заряд отскакивает от стенок и взрывается через случайное время: вся рыба рядом со взрывом попадает в улов, с весом 35%. Каждый такой вид навсегда теряет 1 уровень прикормки.`
   Lucky Fuse: `When a lucky chance comes true, that rod throws a charge of dynamite.` Russian: `Когда удача сбывается, эта удочка бросает в воду заряд динамита.` Lucky Fuse works without the Dynamite charm on the shelf.
6. **Shockwave** now reads: every explosion, with or without fish, lifts the zones by one for the day. A Bait Bomb throw of at least one fish still counts as a blast for it.
7. Bait Bomb is as in addendum 1 section 2 point 5. A level it gives may lift a kind out of a negative level.

## 3. Tests to add (besides those of addendum 1 that still apply)

The same seed gives the same path, bounces and fuse; a bounce off each of the four sides; the fuse stays inside its bounds; a blast with count 0; a blast in a locked zone takes the fish there; levels go to -1 and to the lowest level, the wanted count is half and then zero, no fish of an erased kind enters; the shop offers chum for a negative kind at the base price; "chummed" conditions are false below level 1; a day end with a flying charge; the capacity test with 50 charges alive; both fingerprints with the candidate switch off still equal core 2.9.3.

## 4. Measure

As addendum 1 section 4. Add to the report how often a charge hits nobody (share of blasts with count 0) and the mean fish per blast, by sea state if cheap.

## 5. Time

This adds about 45 minutes to the time box. If the moving charge costs much more, stop and report what is built and what is not; do not cut the random path, the bounces or the random fuse: those are the director's words.
