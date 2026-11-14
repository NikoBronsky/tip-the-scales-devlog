# Brief: second opinion on the new look code (read-only review)

Status: written on 12.11.2026 at 13:33 (owner's clock). Executor: Codex, effort high. Work folder: `F:\UE\EpicJam_core`, a spare git worktree that stands on main. Nobody else works there. You read only.

## 1. Goal

Two Opus agents wrote the first look code of the game today, each in about 30 minutes. More code goes on top of it within hours. The game ships as a packaged Development build for Windows that judges run on machines we do not know. Find the defects that would break the packaged game, or look broken on another screen, before more code depends on them.

The owner's rule (`CLAUDE.md`, section 2): «Главное — ощущение и полировка. Каждое действие игрока отвечает красиво.» A HUD that drifts on a wide screen or a crash on the second run breaks that.

## 2. What exists (verified by the orchestrator)

- Engine: UE 5.9 at `F:\UE_5.9`. Project: `Catch/Catch.uproject`. Module `CatchCore` is the rules core in plain C++20 with no engine. Module `Catch` is the Unreal layer.
- The core and its events: `Docs/design/core-interface.md`. The session object that owns the core: `Catch/Source/Catch/CatchSession.{h,cpp}`.
- New and changed today, all in `Catch/Source/Catch/`:
  - `CatchLookFish.{h,cpp}` (539 lines in the .cpp): the fish as instanced meshes, one material that owns its colour, per-instance data, the pick of a fish under the cursor.
  - `CatchSeaView.{h,cpp}` (1,022 lines): the mirror of the core fish, a slot pool keyed by fish id, the scene.
  - `CatchUi.{h,cpp}` (629 lines): the style kit. Colours by name, one font function, Slate paint helpers (rounded boxes, arcs, fans, stars, outlined words, bait icons), the static multicast delegate `CatchUi::OnUiSound`.
  - `CatchUiHud.{h,cpp}` (310 lines): the painted HUD widget.
  - `CatchHud.{h,cpp}` (1,377 lines): the `UUserWidget` that drives the painted HUD. Clicks go to invisible buttons placed over the painted parts. Sizes are pixels of a 1280x720 reference screen multiplied by viewport height / 720.
  - `CatchPlayerController.{h,cpp}`: the Space key and the mouse.
- Verified today: the editor target compiles, and a day played by console commands in a `-game` run at 1920x1080 shows no error in the log. Nothing else is verified: no packaged build of this code exists yet, and no run at another screen size.

## 3. What to look for, most important first

1. **Packaged build.** Editor-only API or includes that are not behind `WITH_EDITOR`. Assets loaded by a path in code (`LoadObject`, `ConstructorHelpers`, soft paths) that the cooker will not see unless the map or a config entry refers to them. Console commands or files under `Saved/` that the game needs at run time. Anything that works in `-game` and fails in a cooked build.
2. **Other screens.** 1280x720, 2560x1440, 3840x2160; aspect 16:10 and 21:9; a DPI scale other than 1; windowed and full screen. Look for layout that breaks on a wide or a narrow aspect, hit buttons that drift away from the painted parts, text that overflows, a world-to-screen projection that assumes one size.
3. **Crashes and lifetime.** A null session or world. The static delegate holding dead objects after a level restart or a second run. Raw pointers to `UObject`s without `UPROPERTY`. Events handled after `NativeDestruct`. Array access by fish id or slot without a range check. A new run (`catch.NewRun`) while popups, cards or pooled fish are alive.
4. **Cost per frame with 1,000 fish.** Allocations in tick or paint (string building, array growth). Instance updates that mark the render state dirty once per fish instead of once per frame. Loops over all fish inside loops over all fish. The pick under the cursor.
5. **Input.** Clicks that reach both an invisible button and the sea. Drag of a hook depth that starts over a HUD part. Focus loss. The Space key in a phase where it must do nothing.
6. **Disagreement with `Docs/design/core-interface.md`:** the HUD reads state it must not, or misses an event that it should show.

## 4. Not in scope

Style, naming and formatting. The look itself: colours, sizes, wording. Code that did not change today. The rules of the game. Do not propose a redesign.

## 5. Output

Reply only. At most 15 findings, most severe first. For each one:

- `file:line`;
- what is wrong, in one or two sentences;
- why it matters for the packaged game or for another screen;
- the smallest fix;
- CONFIRMED (you traced the code path) or SUSPECTED (say what would confirm it).

End with three lines: what you checked and found fine.

## 6. Boundaries

- Read only. Write no file, change no file, run no build, run no Unreal process, run no git command that changes anything.
- Read inside `F:\UE\EpicJam_core`. You may read the engine source under `F:\UE_5.9\Engine\Source` for facts about an API. Do not read `F:\UE\EpicJam` or `F:\UE\EpicJam_ui`: two executors change files there right now.
- You do not invent the game. A finding that needs a rule or a number from the owner: say so, and stop there.

## 7. Chain of command

You report to the orchestrator. The orchestrator checks each finding and passes the real ones to the executor that owns the file.
