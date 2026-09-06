---
date: 2026-09-06
topic: advanced-chess-kit-release-checklist
---

# Advanced Chess Kit — Release Checklist

Copy this file per submission (`release-checklist-<version>.md`), tick as you go,
and archive the completed copy with the submission. An unticked box is a blocker
unless it carries a written waiver in §9.

```
Version:          ______________________
Branch / commit:  ______________________
Unity version:    6000.5.9f1
Date:             ______________________
Run by:           ______________________
Submission type:  [ ] first submission   [ ] update
```

---

## 1. Entry gate

- [ ] Working tree is clean; the release commit is tagged or its SHA is recorded above.
- [ ] `Documentation/README.md` and `Samples/README.md` describe **this** build, not the previous one.
- [ ] Every change since the last release appears in [traceability-matrix.md](traceability-matrix.md), or has a stated reason it needs no entry.
- [ ] Version number bumped consistently everywhere it appears.

## 2. Automated suite

- [ ] EditMode suite green — **100%**, zero failures, zero ignored, zero skipped.
- [ ] Test count is at or above the last release's count (a drop means tests were deleted — justify it).
- [ ] Headless run reproduces the Editor result:
      ```bash
      Unity.exe -batchmode -projectPath . -runTests -testPlatform EditMode -testResults TestResults.xml -logFile -
      ```
- [ ] `TestResults.xml` archived with this checklist.
- [ ] Editor console clean on scene load and across a full game: no errors, no new kit-originated warnings.

## 3. Manual functional pass — Core environments

Per [test-scenarios.md](test-scenarios.md). Tick the row only when every scenario
listed for it in the scenario-to-environment grid has passed.

- [ ] **E1** — URP · New Input System · no DOTween
- [ ] **E2** — URP · New Input System · DOTween present
- [ ] **E3** — Built-in · old Input Manager · no DOTween

Secondary rows (mandatory for a first submission; for an update, mandatory if
anything touched input, defines, or dependencies):

- [ ] **E4** — URP · Both · DOTween present
- [ ] **E5** — Built-in · Both · no DOTween
- [ ] **E6** — a Unity 6 patch version other than the development one

## 4. Engine and pacing verification

- [ ] `DifficultyCalibrationTests` re-run **after** the final engine/search change, not before.
- [ ] DIFF-M1 — `easy` and `normal` replies land 1.5–2.5 s after the reveal finishes.
- [ ] DIFF-M2 — `hard` replies land 2.5–3.5 s after the reveal finishes.
- [ ] DIFF-M3 — an opening-book reply still waits out the window.
- [ ] SRCH-M1 — no frame hitch on move application; UI stays responsive during a reply.
- [ ] The README difficulty table (budgets, depth ranges, reply windows) matches what was just measured. **If the numbers moved, the table moved with them** (BB-R19, SI-R9).
- [ ] No strength claim anywhere in the docs or store copy that these tests do not support.

## 5. Platform builds

Tick only platforms the store listing will claim.

- [ ] **Windows Standalone** — PLAT-01 + SC-12
- [ ] **WebGL** — PLAT-02 + SC-12 (page stays responsive; save survives a reload)
- [ ] **Android** — PLAT-03 + SC-12 (touch, force-close survival, ladder still ordered)
- [ ] **iOS** — PLAT-04 + SC-12
- [ ] The store listing claims **no** platform without a passing box above.
- [ ] Platform logs (logcat / browser console) reviewed for errors across a whole session.

## 6. Package composition

- [ ] Exported `.unitypackage` contains exactly `Ostryzhnyi/Advanced Chess Kit/` and `Ostryzhnyi/EasyViewService/`, plus `Third-Party Notices.txt` at the package root.
- [ ] Nothing else ships: no `.csproj`, no `Library/`, no `docs/`, no `TestResults.xml`, no `.unitypackage`, no asset belonging to a consuming application, no IDE folders, no editor-only scratch files.
- [ ] Decision recorded — the test assembly **[ ] ships / [ ] does not ship** with the package. (Reason: ______________________)
- [ ] `KitSelfContainmentTests` green — no kit asset points outside the kit, no byte-copy of a third-party asset, shaders compile.
- [ ] Every `.meta` file is present and committed; no missing-GUID references after import.
- [ ] Demo scene included and named as the docs say it is.

## 7. Legal and compliance

- [ ] `Third-Party Notices.txt` carries the MinimalChessEngine MIT text and © 2021 Thomas Jahn.
- [ ] The fork is declared: what changed, in which files, and why — in the notices **and** in `Runtime/MinimalChess/THIRD-PARTY-NOTICES.md`.
- [ ] Every forked file carries its in-place `Advanced Chess Kit fork` comment, and the marked set matches the notices exactly.
- [ ] No paid third-party asset, and no asset the project has no redistribution right to, is anywhere in the package (paid UI, inspector, tweening or shader packages; and any art, audio or content-pipeline asset belonging to a consuming application).
- [ ] DOTween and UniTask are **not** bundled; both remain optional.
- [ ] Store listing text makes no claim the package does not deliver — supported Unity versions, pipelines, platforms, and what "no hard third-party dependencies" means.
- [ ] Screenshots and the store video show this build, not an older one.

## 8. Fresh-import acceptance (L5)

Run SC-11 against the **exported package**, in a brand-new empty project.

- [ ] Imports with zero compile errors and zero kit-originated warnings.
- [ ] Nothing lands outside the two permitted root folders.
- [ ] `Samples/AdvancedChessKitSample.unity` opens and plays a full game (SC-01 in full).
- [ ] Both editor tools work in that bare project — including the Test Launcher, whose default target scene is the kit's own sample (BB-R17, BB-R18).
- [ ] Second import test: a project that already has EasyViewService gets no duplicate and no duplicate-type errors.

## 9. Exit gate

- [ ] All **P0** cases pass on every Core environment row.
- [ ] All **P1** cases pass, or are waived below.
- [ ] No open **S1** or **S2** defect ([bug-report-template.md](bug-report-template.md)).
- [ ] `TestResults.xml` and this completed checklist archived together.

**Waivers** — one line each: case ID, what fails, why shipping anyway is
acceptable, who accepted it.

```
1.
2.
3.
```

## 10. Post-submission

- [ ] Release commit tagged and pushed *(pushing is the developer's own step)*.
- [ ] Exported `.unitypackage` archived alongside the tag.
- [ ] Any defect found during this pass but not fixed is filed, not just remembered.
- [ ] Anything this pass taught you that the docs did not say — add it to [test-cases.md](test-cases.md) or [test-plan.md](test-plan.md) now, while it is fresh.
