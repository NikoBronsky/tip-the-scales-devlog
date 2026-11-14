# Brief — pre-jam: what an agent can do through the editor link

Date: 2026-11-12. Author: orchestrator, on the owner's decisions of 2026-11-11 and 2026-11-12. Executor: Opus, effort high.
Project under test: `F:\UE\EpicJam_ToolTest\ToolTest.uproject` (throwaway, Unreal Engine 5.9, C++). Repository of the jam game: `F:\UE\EpicJam` (contract: `CLAUDE.md`, in Russian). Time box: 90 minutes.

## 0. Why
The owner works alone and all production is done by agents («Я работаю один. Всё должно делаться аи.»). Polish is the top priority («для нас главное - полишинг»). From 06:00 agents build the whole game through the editor link. Before the clock starts we must know which editor operations an agent can really perform through that link, with the exact calls that work. Otherwise the first jam hours go into discovering the tools. The jam rules forbid making game content before the start, so this check uses neutral test objects only.

## 1. What exists now (verified facts)
- The Unreal editor is running on the throwaway project. Its MCP server answers at `http://127.0.0.1:8000/mcp`.
- Client for any shell: `python -X utf8 F:/UE/EpicJam/Tools/ue_mcp.py <command>`. Commands: `status`, `toolsets`, `describe <Toolset>...`, `schema <Toolset.Tool>...`, `call <Toolset> <Tool> [json | @file.json]`. Read the docstring at the top of the file. Image results are saved to `F:/UE/EpicJam/Saved/ue_mcp/` and the path is printed.
- Verified calls: `status`; `toolsets` (106 names); `describe EditorToolset.LevelToolset`; `schema SceneCaptureToolset.SceneCaptureToolset.CaptureActiveEditorView`; `call EditorToolset.LevelToolset GetCurrentLevel`; `call EditorToolset.LevelToolset LoadLevel @args.json` with `{"levelPath":"/Engine/Maps/Templates/Template_Default","unsavedChangesPolicy":"Fail"}`; `call SceneCaptureToolset.SceneCaptureToolset CaptureActiveEditorView @args.json` with `{"mode":"Lit","options":{"width":960,"height":540}}` (returns a PNG).
- A tool rejects a call when a parameter listed as `required` in its schema is missing, even a boolean (example: `ListModes` needs `bIncludeUnavailable`). Read the schema before the first call of each tool.
- Pass JSON arguments through `@file.json`. Shell quoting of JSON on Windows is fragile. In this harness the Bash tool halves double backslashes inside heredocs, so write paths with forward slashes.
- While the editor is open, UnrealBuildTool refuses to build the editor target ("Unable to build while Live Coding is active"). Code changes in the open editor go through `LiveCodingToolset`.
- Close the editor: `call EditorToolset.EditorAppToolset ExitEditor`. Start it: PowerShell `Start-Process -FilePath 'F:\UE_5.9\Engine\Binaries\Win64\UnrealEditor.exe' -ArgumentList '"F:\UE\EpicJam_ToolTest\ToolTest.uproject"'`, then poll `ue_mcp.py status` until it answers (about one to two minutes). The MCP server starts by itself.
- Headless editor build (editor closed): `powershell -NoProfile -ExecutionPolicy Bypass -File F:/UE/EpicJam/Tools/build-editor.ps1 -Project F:/UE/EpicJam_ToolTest/ToolTest.uproject` (about one minute).
- Engine source is on disk and readable: `F:\UE_5.9\Engine`. The toolset plugins live under `F:\UE_5.9\Engine\Plugins\Experimental\Toolsets` and `...\Experimental\ModelContextProtocol`. Reading them is the fastest way to learn a tool's real behaviour.
- The module source is `F:\UE\EpicJam_ToolTest\Source\ToolTest\` (`ToolTest.Build.cs`, `ToolTest.cpp`, `ToolTest.h`).

## 2. Goal
A capability map that jam briefs can copy from. For each capability in section 4: a verdict (works / works with limits / does not work), the exact calls that worked (toolset, tool, argument JSON), how long it took, and the pitfalls. Write it in English to `F:\UE\EpicJam\Docs\research\ue59-editor-link-capabilities.md`.
Not in scope: game design, any fishing, water, sea, fish, boat or rod content, any change in `F:\UE\EpicJam` other than the report file.

## 3. Technical decisions (revocable)
1. All test assets go to `/Game/ToolTest/` with neutral names that start with `T_` after the type prefix (for example `M_T_Red`, `WBP_T_Hello`, `L_T_Check`) — risk: none — fallback: any neutral name.
2. Captures are 960x540 — risk: small details unreadable — fallback: capture a larger frame once when needed.
3. If a capability fails after three different attempts, record the error text and move on — risk: a false "does not work" — fallback: list it under "untested ideas".
4. If a toolset cannot do something, try the Python scripting toolset (`editor_toolset.toolsets.python_scripting.PythonScriptingTools`) as the universal fallback and say so in the verdict.

## 4. Work by blocks (in priority order; stop when the time box ends)
### Block A. Level and actors
- Create and open a new level `/Game/ToolTest/L_T_Check`. Add: engine basic shapes (cube, sphere, plane), a directional light, a sky light, a sky atmosphere, exponential height fog, an unbound post process volume, a camera actor. Set transforms and a few properties (light intensity, fog density, a post process setting). Save. Capture the result and look at the image.
### Block B. Materials
- Create a material with colour, emissive and roughness parameters. Create a translucent material. Create a material whose vertices move with time (world position offset driven by a sine) and that reads per-instance custom data. Create a material instance and set its parameters. Assign materials to the shapes. Capture and confirm the change is visible.
- Report which graph nodes and settings the material toolset can author and which it cannot.
### Block C. Play in editor
- `StartPIE`, `IsPIERunning`, capture the game viewport, record three seconds of the viewport to MP4, `StopPIE`.
- Input: find out whether an agent can inject a mouse click and a key press into the running game (`InputToolset` or another way). This decides whether agents can playtest by themselves.
### Block D. C++ loop with the editor open
- Change an existing `.cpp` (add a log line in a function that runs), compile with `LiveCodingToolset`, prove the new code runs (logs toolset or console).
- Add a new `UCLASS` (an `AActor` subclass with one `UPROPERTY`). Find out whether Live Coding makes it usable, or whether the editor must be closed, rebuilt headless and reopened. Measure both paths.
### Block E. Automation tests
- Add one trivial automation test in C++. Run it in the editor through `AutomationTestToolset`.
- Run it headless with the editor closed (`UnrealEditor-Cmd.exe <uproject> -ExecCmds="Automation RunTests <filter>; Quit" -unattended -nullrdr -nosplash -log`, adjust as needed). Record the exact command, how to read pass or fail from the exit code or the log, and the run time.
### Block F. Interface
- Create a Widget Blueprint with a text block and a button through `UMGEditor.WidgetBlueprintEditorSubsystem`. If possible, parent it to a C++ `UUserWidget` subclass that binds the widgets. Show it during play and capture it.
- Report how far styling goes through the tools: fonts, colours, anchors, layout, widget animations (`WidgetAnimationToolset`).
### Block G. Particles
- Create a simple Niagara system (for example a sprite fountain), place it, capture it. Report the limits of the Niagara toolsets.
### Block H. Asset import
- Generate with Python, in a temp folder outside both projects: a small PNG, a trivial OBJ or FBX mesh (a box is enough), a one-second sine WAV. Import each into `/Game/ToolTest/`. Assign the texture to a material, place the mesh, check the sound asset exists. Report which formats import and through which tool.
### Block I. Many objects cheaply
- Find out whether an agent can create an instanced static mesh component with a few hundred instances and per-instance custom data, through a toolset or through Python. Capture it.
### Block J. Python scripting
- Show one example of running editor Python through the link (for example listing assets). State clearly whether arbitrary `unreal` module scripts run, because that is the fallback for everything above.

## 5. Done means
1. `Docs/research/ue59-editor-link-capabilities.md` exists. It has a verdict table (capability, verdict, tool used, time, pitfalls) and a "Recipes" section with copy-and-paste calls and argument JSON for everything that worked.
2. Every block was attempted in order, or is listed as "not reached" with the reason.
3. Failures carry the error text.
4. At the end the editor is running and answers `status`, play mode is stopped, the test level is saved.
5. Nothing under `F:\UE\EpicJam` changed except the report file. No git command was run there.
6. Report delivered (section 8).

## 6. Boundaries
- Neutral test content only. No fish, water, sea, fishing gear, boats, money, shops or any game mechanic, not even as a test example.
- Write only inside `F:\UE\EpicJam_ToolTest`, a temp folder of your own, and the one report file.
- Engine folder and `F:\UnrealCache` are read-only for you.
- No downloads and no network use besides `127.0.0.1`.
- Do not use computer-use or any screen control. If the editor shows a modal dialog and the link stops answering for more than two minutes, write down what you were doing and stop that block; the orchestrator will look.
- Stop only processes you started. Use `ExitEditor` to close the editor; force-kill `UnrealEditor.exe` only if it does not exit within 60 seconds.
- If the editor crashes, note what caused it, restart it and continue with the next block.
- No commit, no stash, no checkout, no reset anywhere.

## 7. Verification
```bash
python -X utf8 F:/UE/EpicJam/Tools/ue_mcp.py status
```
```bash
python -X utf8 F:/UE/EpicJam/Tools/check_all.py
```
The second command must still print `0 fail` when you finish.

## 8. Report and questions
Return as your final message, in English, under 400 words: the verdict per block in one line each, the five pitfalls that will cost the most time during the jam, what is untested, and the state you left the editor in. The full detail belongs in the report file, not in the message.
You report to the orchestrator, not to the owner. Put open questions under "Questions for the orchestrator". Do not ask the owner anything.
