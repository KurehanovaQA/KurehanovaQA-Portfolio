---
date: 2026-09-06
topic: advanced-chess-kit-bug-reporting
---

# Advanced Chess Kit — Defect Reporting

## Severity ladder

Severity is about **impact on a buyer**, not about how hard the fix is.

| | Meaning | Examples | Release effect |
|---|---|---|---|
| **S1 — Critical** | The kit does not work as a product, or ships something it has no right to ship | Import fails to compile; the sample scene throws on Play; a wrong rules verdict (illegal move accepted, mate not detected); a paid third-party asset is inside the package; a save is corrupted or lost | Blocks release. No waiver. |
| **S2 — Major** | A documented feature does not do what the docs say | Continue resumes a finished game; Restart does not discard the save; a screen removed causes a null reference; the difficulty table overstates what ships; a claimed platform does not run | Blocks release. No waiver. |
| **S3 — Moderate** | Wrong or degraded behaviour with a workaround, or a defect on a secondary path | Reply pacing outside the published window on one platform; hint markers linger a ply too long; SFX jitter not audible; an editor tool refuses a valid input | Blocks unless waived in the release checklist. |
| **S4 — Minor** | Cosmetic, or affects only a rare configuration | Label colour slightly off in one skin; a screen's fade timing feels abrupt; a doc typo | Ship and fix later. |

**Priority** is separate from severity: an S4 typo in the store-page-visible
README may be P0 for a submission, while an S3 on an unclaimed platform may be P3.

## Report template

Copy this whole block per defect.

```markdown
### [S?] <one-line summary — what is wrong, not what you were doing>

**Found by:**   <case or scenario ID, e.g. SC-05 step 3 / SESS-03 / exploratory>
**Severity:**   S1 / S2 / S3 / S4
**Priority:**   P0 / P1 / P2 / P3
**Area:**       ENG / SRCH / DIFF / SAVE / ASYNC / SESS / VIEW / TOOL / PKG / PLAT
**Requirement:** BB-R? / SI-R? / none
**Regression:**  yes (last known good: <version or commit>) / no / unknown

**Environment**
- Kit version / commit:
- Unity:                6000.5.9f1
- Pipeline:             URP / Built-in
- Active Input Handling: New / Old / Both
- DOTween:              present / absent
- Platform:             Editor / Windows / WebGL / Android / iOS
- Scene:                sample scene / own scene / Test Launcher (FEN: ______)

**Steps to reproduce**
1.
2.
3.

**Expected**
<what the docs, the requirement, or the test case say should happen — quote it>

**Actual**
<what happened, precisely. Include the outcome, not just "it broke">

**Reproduction rate:** always / intermittent (__ of __ attempts) / once

**Evidence**
- Console output / stack trace:
- FEN or save string at the point of failure:
- Screenshot / clip:
- Test results XML (if an automated failure):

**Notes**
<workaround, suspected cause, related defects — optional, and keep speculation
labelled as speculation>
```

## Filing rules

**One defect per report.** If two things broke in one scenario, that is two
reports. They will be fixed and verified separately.

**Always capture a reproduction handle.** For anything chess-related that means
the **FEN or the save string** at the moment of failure — the Test Launcher then
puts anyone straight into the position. A defect without one costs far more to
fix than it did to find.

**Say whether it is a regression.** If the same case passed last release, say so
and name the last known good version. That single line usually points at the
change responsible.

**Quote the expectation, don't paraphrase it.** "The README says replies land
1.5–2.5 s after the reveal; I measured 0.3 s" is actionable. "Reply feels too
fast" is not.

**Report what you observed, not what you concluded.** A theory about the cause is
welcome in Notes, clearly marked. It does not belong in Actual.

## When a defect is found

1. File it from the template above.
2. If it is S1 or S2, **stop the release pass** — there is no point finishing a
   pass against a build that will not ship.
3. Check whether an automated test *should* have caught it. If yes, that is a
   second defect: the test gap. Note it in the report.
4. After the fix: re-run the originating case, the whole area's cases, and
   whatever the [reverse index](traceability-matrix.md#reverse-index--what-to-re-run-after-a-change)
   lists for the code that changed.
5. If the defect was catchable by an automated test that did not exist, **add
   that test as part of the fix** — the kit's suite grew this way and should keep
   growing this way.

## Verifying a fix

A fix is verified when:

- [ ] The originating case passes on the environment where it failed.
- [ ] The full EditMode suite is still green.
- [ ] A new automated test covers the defect, or there is a stated reason none can.
- [ ] The reverse-index re-runs for the changed code all pass.
- [ ] If the fix changed documented behaviour, the docs changed in the same commit.
