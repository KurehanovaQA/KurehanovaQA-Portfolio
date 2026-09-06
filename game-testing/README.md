---
date: 2026-09-06
topic: advanced-chess-kit-test-documentation
---

# Advanced Chess Kit — Test Documentation

QA documentation for the **Advanced Chess Kit** Unity Asset Store package
(`Assets/Ostryzhnyi/Advanced Chess Kit/`, plus `Assets/Ostryzhnyi/EasyViewService/`,
which ships alongside it).

These documents live in the repository, **not** in the shipped package — the kit
folders stay clean for store submission, and `KitSelfContainmentTests` would flag
anything added under them that points outside the kit.

---

## The documents

| File | What it is | Read it when |
|---|---|---|
| [test-plan.md](test-plan.md) | Scope, test levels, environment matrix, entry/exit criteria, risks | Planning a release or onboarding onto the kit's QA |
| [test-cases.md](test-cases.md) | 158 atomic cases with IDs, preconditions, steps, expected results | Executing a pass, or writing a new automated test |
| [test-scenarios.md](test-scenarios.md) | 12 end-to-end journeys that string cases together | Running a manual regression or a pre-submission smoke |
| [traceability-matrix.md](traceability-matrix.md) | Requirement → case → automated test mapping | Proving a requirement is covered, or finding a gap |
| [release-checklist.md](release-checklist.md) | The gate before exporting/uploading a `.unitypackage` | Cutting a store submission or an update |
| [bug-report-template.md](bug-report-template.md) | Defect template + severity ladder | Filing anything found during a pass |

---

## Running the automated suite

The whole automated suite is **EditMode only** — the test assembly
(`Assets/Ostryzhnyi/Advanced Chess Kit.Tests/AdvancedChessKit.Tests.asmdef`)
is `includePlatforms: ["Editor"]` and constrained to `UNITY_INCLUDE_TESTS`.
There is no PlayMode suite; everything that needs a running scene is covered
manually (see the `Manual` cases in [test-cases.md](test-cases.md)).

**In the Editor:** `Window ▸ General ▸ Test Runner ▸ EditMode ▸ Run All`.

**Headless (CI or a clean check):**

```bash
Unity.exe -batchmode -projectPath . -runTests -testPlatform EditMode -testResults TestResults.xml -logFile -
```

As of this writing the suite is **11 fixtures / 112 cases** (counted from the
`[Test]` and `[TestCase]` attributes in source). Fixture inventory and what each
one guards is in [test-plan.md](test-plan.md#automated-suite-inventory).

---

## Manual test aids

- **Test Launcher** — `Ostryzhnyi ▸ Advanced Chess Kit ▸ Test Launcher`. Paste a
  FEN or pick a preset (starting position, promotion, en passant, castling, in
  check, mate in one either way), then Play to drop straight into that position.
  This is the primary tool for the rules and edge-case manual cases; the preset
  FENs are listed in [test-cases.md](test-cases.md#appendix-a--test-positions-fen).
- **Skin editor** — `Ostryzhnyi ▸ Advanced Chess Kit ▸ Edit`. Live board preview
  for the `ChessBoardSkin` cases.
- **Sample scene** — `Assets/Ostryzhnyi/Advanced Chess Kit/Samples/AdvancedChessKitSample.unity`.
  The canonical manual-test scene: board, camera, six screens, `ChessKitSession`,
  and the Best Move / Undo demo buttons — and nothing from any consuming
  application.

---

## Conventions used across these docs

- **Case IDs** are stable. Never renumber; retire an ID rather than reuse it.
- **Priority** — `P0` blocks a release, `P1` blocks unless explicitly waived,
  `P2` is polish.
- **Type** — `Auto` (covered by a named EditMode test), `Manual`, or `Auto+Manual`
  (automated at the logic level, still needs eyes on the presentation).
- Requirement references (`BB-R1`, `SI-R5`) point at
  [`docs/brainstorms/2026-09-05-chess-kit-bug-batch-requirements.md`](../brainstorms/2026-09-05-chess-kit-bug-batch-requirements.md)
  and [`docs/brainstorms/2026-09-06-chess-kit-search-integrity-requirements.md`](../brainstorms/2026-09-06-chess-kit-search-integrity-requirements.md)
  respectively.
