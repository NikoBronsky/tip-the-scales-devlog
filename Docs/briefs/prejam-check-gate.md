# Brief — pre-jam: check gate script

Date: 2026-11-12. Author: orchestrator, on the owner's decision of 2026-11-12. Executor: Sonnet, effort medium.
Project: `F:\UE\EpicJam`, Unreal Engine 5.9 jam game (2026 Epic AI MegaJam), Python 3.11 for tools. Contract: `CLAUDE.md`. Paths below are relative to `F:\UE\EpicJam` unless stated.

## 0. Why
The owner works alone and all production is done by agents («Я работаю один. Всё должно делаться аи.»). The owner asked the orchestrator to run the project with the usual harness («Помни, что ты можешь работать как оркестратор, у нас даже скиллы под это есть»). The harness needs one fast command that every agent runs before handing work over. Without it, a jam rule can be broken silently (for example Russian text in the game, or a submission archive over the size limit) and the entry is disqualified.

## 1. What exists now (verified facts)
- The folder is a fresh git repository on branch `main` with Git LFS initialised locally. Nothing is committed yet.
- `.gitignore` ignores `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`, `Packaged/`, `Builds/`, `*.zip`.
- `.gitattributes` sends `*.uasset`, `*.umap`, images, audio, video, `*.fbx`, `*.glb`, `*.blend`, fonts through LFS.
- `Tools/build-editor.ps1 -Project <uproject>` compiles the editor target. `Tools/package-win64.ps1 -Project <uproject> -OutDir <dir>` packages a Development Win64 build.
- The game project does not exist yet. It is created after the jam starts. Its `.uproject` will sit in the repository root or one folder below it.
- The orchestrator is writing these files in parallel with you: `CLAUDE.md`, `AGENTS.md` (must be a byte-identical copy of `CLAUDE.md`), `Docs/journal.md`, `Docs/decisions/*.answers.json`, `DEVLOG.md`, `Tools/llm_or.py`. They may or may not exist while you work.
- Reference implementation of the same idea in another project of the owner (read-only, do not copy project-specific checks): `F:\UE\SquadUE\Tools\check_all.py`.
- Jam rules that the gate guards: all in-game text and the public dev log are in English; the submission zip is under 1 GB; the build is a packaged Development build for Windows.

## 2. Goal
`python -X utf8 Tools/check_all.py` runs in a few seconds, prints one line per check with status `ok`, `fail` or `skip`, and exits with code 1 if any check failed, otherwise 0. A skipped check never fails the run. The script uses only the Python standard library and `git`.
Not in scope: Unreal automation tests, gameplay checks, any change outside `Tools/check_all.py`.

## 3. Technical decisions (revocable)
1. One file, `Tools/check_all.py`, no helper modules — risk: grows long — fallback: split later.
2. Optional argument `--root <path>` makes the script check another folder instead of the repository (used for your fixture tests) — risk: none — fallback: environment variable.
3. Optional flag `--ue` also compiles the editor target through `Tools/build-editor.ps1` for the first `.uproject` found at `<root>/*.uproject` or `<root>/*/*.uproject`; without the flag, or with no `.uproject`, the check is `skip` — risk: slow — fallback: keep it opt-in.
4. Output: a plain aligned text table, then a one-line summary `N ok, N fail, N skip` — risk: none.

## 4. Work by blocks
### Block A. Checks (`Tools/check_all.py`)
- `agents-copy`: `AGENTS.md` is byte-identical to `CLAUDE.md`. Either file missing → fail.
- `journal`: `Docs/journal.md` exists and is not empty → ok, else fail.
- `devlog`: `DEVLOG.md` exists and is not empty → ok, else fail.
- `decision-json`: every `Docs/decisions/*.json` parses as JSON (read with `utf-8-sig`). No files → ok with detail `0 files`.
- `build-scripts`: `Tools/build-editor.ps1` and `Tools/package-win64.ps1` exist.
- `english-only`: no Cyrillic characters (U+0400–U+052F) in files a player or a judge can see. Scan: `DEVLOG.md`; every `*.h`, `*.cpp`, `*.cs`, `*.ini`, `*.csv`, `*.json`, `*.txt`, `*.md`, `*.po` under any folder that contains a `.uproject` (the game project), excluding its `Binaries`, `Intermediate`, `Saved`, `DerivedDataCache` folders; every text file under `Docs/itch/` if that folder exists. Never scan `CLAUDE.md`, `AGENTS.md`, `Docs/` outside `Docs/itch/`, `Tools/`. If the `.uproject` sits in the repository root itself, the exclusions of the previous sentence still apply. Report up to 10 hits as `path:line`. Unreadable or binary files are ignored. No game project and no hits → ok.
- `git-ignored-paths`: no path tracked or staged in git lives under `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`, `Packaged/`, `Builds/` at any depth. Use `git ls-files` (it covers the index). If `<root>` is not a git repository → skip.
- `git-lfs`: every tracked or staged file whose extension is one of the LFS extensions in `.gitattributes` is an LFS pointer, and no tracked non-LFS file is larger than 50 MB. Parse the extensions from `.gitattributes` lines that contain `filter=lfs`. Use `git lfs ls-files -n` for the LFS set. Not a git repository → skip.
- `submission-zip`: every `Packaged/*.zip` is smaller than 1 000 000 000 bytes. No zip → skip.
- `ue-build` (only with `--ue`): see decision 3. Put the last 15 lines of the build output into the detail on failure.

### Block B. Tests
- Build throwaway fixture folders under your scratch folder (not inside the repository) and run the script with `--root` to prove each check can pass and can fail: a mismatching `AGENTS.md`; a broken decision JSON; a `.cpp` with a Cyrillic string inside a fake game project folder; a Cyrillic line in `DEVLOG.md`.
- For the zip limit, do not create a 1 GB file. Either state the fail path as untested or add a hidden `--zip-limit <bytes>` argument and test with a small file.
- For the git checks, make a scratch git repository (`git init`), stage a file under `Saved/` with `git add -f` and confirm the fail.
- Run the script on the real repository last and paste its output.

## 5. Done means
1. `python -X utf8 Tools/check_all.py` runs on `F:\UE\EpicJam` without a traceback and prints the table.
2. Each check in Block A was shown to pass and to fail on fixtures (list them in the report), except the ones you state as untested.
3. Exit code is 1 when a check fails and 0 otherwise.
4. Only `Tools/check_all.py` was created or changed inside the repository.
5. Report delivered (section 8).

## 6. Boundaries
- Do not create or change any file in `F:\UE\EpicJam` except `Tools/check_all.py`.
- Do not touch `F:\UE\EpicJam_ToolTest` (another task is running the Unreal editor and a packaging run on it). Do not start Unreal, UnrealBuildTool or the build scripts.
- Do not run `--ue` on the real repository.
- No commit, no stash, no checkout, no reset, no `git add` in `F:\UE\EpicJam`.
- Messages printed by the script are in English.

## 7. Verification
```bash
python -X utf8 F:/UE/EpicJam/Tools/check_all.py
```
```bash
python -X utf8 F:/UE/EpicJam/Tools/check_all.py --root <fixture folder>
```

## 8. Report and questions
Return the report as your final message, in English, under 300 words: what each check does in one line, which section-3 decisions you changed and why, what is untested, the fixture results, the output of the run on the real repository, the list of changed files.
You report to the orchestrator, not to the owner. Put open questions in a "Questions for the orchestrator" list in the report. Do not ask the owner anything.
