---
date: 2026-09-06
topic: advanced-chess-kit-test-plan
---

# Advanced Chess Kit — Test Plan

## 1. Purpose

Define what gets tested, how, and on what, before the kit is exported as a
`.unitypackage` and submitted to the Unity Asset Store — and before every update
after that.

The product under test is a **drop-in single-player chess game**, not a board
component. That framing sets the priority order: a buyer who imports the package
into an empty project must get a *playable game* without writing glue. Anything
that breaks that is P0 regardless of how narrow the underlying defect looks.

## 2. Scope

### In scope

| Area | What it covers |
|---|---|
| **Engine correctness** | Legal move generation, check/mate/stalemate, promotion, en passant, castling, threefold repetition, insufficient material, resignation |
| **Search integrity** | Transposition-table hygiene, sliced search determinism, depth accounting, root-move ranking |
| **Difficulty & pacing** | Budget per level, the depth ladder, the reply time window, unknown-id fallback |
| **Persistence** | `Save()`/`TryLoad()`, `IChessGameStorage`, PlayerPrefs default, flush at session boundaries, tampered/truncated/foreign save strings |
| **Session loop** | `ChessKitSession` entry routing, restart routing, Continue gating, pause, save lifecycle |
| **UI screens** | All six overlay screens, each present and each absent |
| **Board presentation & input** | Selection, drag, tap-tap, highlights, animation, hint markers, SFX, pointer backends |
| **Editor tools** | Skin editor window, Test Launcher, DOTween define setup |
| **Packaging** | Self-containment, third-party notices, folder layout, import into an empty project |
| **Environment matrix** | Unity version, render pipeline, input handling mode, optional DOTween/UniTask, target platforms |

### Out of scope

- The consuming application the kit is developed alongside — its scenes, content
  pipeline, localization, save slots, and progression systems. The kit is tested
  standalone, and proving that it *can* be tested standalone is itself part of
  the scope (see the PKG cases).
- Chess *strength* as a numeric rating (Elo). The kit ships no external reference
  to calibrate against; the only strength guards are ordinal (see §6).
- `EasyViewService` as a product. It is exercised only through the six screens.
- Multiplayer, networking, clocks, PGN import/export — none exist.
- Localization of screen text beyond "the key resolves and no `MISSING:` string
  reaches the screen".

### Assumptions

- Automated tests run in the Editor only. No PlayMode suite exists by design;
  presentation-level behaviour is covered manually and deliberately so.
- The reference machine for pacing measurements is the developer workstation.
  Timing cases state tolerances wide enough to survive slower hardware, since
  the budget is wall-clock (see [test-cases.md](test-cases.md) DIFF cases).

## 3. Test levels

| Level | Where | Gate |
|---|---|---|
| **L1 — Unit / contract** | EditMode suite, `AdvancedChessKit.Tests` | Every commit. 100% pass, no exceptions. |
| **L2 — Integration** | EditMode suite (host + storage + search together) | Every commit, same gate as L1. |
| **L3 — Manual functional** | Sample scene + Test Launcher, in-Editor Play mode | Every release candidate. |
| **L4 — Environment matrix** | Fresh empty projects, each matrix row | Every release candidate, and any change to assembly definitions, defines, or dependencies. |
| **L5 — Package acceptance** | Exported `.unitypackage` imported into an empty project | Every submission. |

L1 and L2 share one assembly and one runner; they are separated here only to
name the intent — `MinimalChessGameTests` is L1, `ChessGameStorageTests` (which
spins up real `ChessBoardHost` components) is L2.

## 4. Automated suite inventory

11 fixtures, 112 cases. What each one guards, and what it deliberately does not:

| Fixture | Cases | Guards |
|---|---:|---|
| `MinimalChessGameTests` | 35 | Rules end to end — legal moves, mate, draws, promotion, undo, save round-trip, opening book, FEN starts |
| `AsyncReplyContractTests` | 12 | The engine reply is computed across frames: what is rejected mid-computation, what cancels it, what `ExpectedPlayerReply` reports |
| `ChunkedSearchTests` | 11 | The sliced search itself — slice sizes, budget accounting, mate-in-one under any slicing, no TT corruption |
| `RankMovesCharacterizationTests` | 11 | One search per hint/reply regardless of difficulty; hint frame count is difficulty-independent; top choice really is the search's best move |
| `ChessGameContractTests` | 10 | `IChessGame` contract against the in-memory fake — header, outcome, undo limits, unreadable state |
| `AsyncHintContractTests` | 8 | Hint computation lifecycle and its mutual exclusion with replies |
| `ChessGameStorageTests` | 8 | `IChessGameStorage` substitution, mid-game round-trip through storage, outcome peek, truncated save rejection |
| `SearchIntegrityTests` | 6 | The one fixture that **never** clears the transposition table in setup — consecutive searches through a shared table must not degrade |
| `CheckContractTests` | 4 | Check reporting on delivery, on removal, on mate, and on restore |
| `DifficultyCalibrationTests` | 4 | The ordinal strength guards: depth ladder, hard never loses to easy, hard never passes up a mate, unknown id falls back |
| `KitSelfContainmentTests` | 3 | No kit asset references anything outside the kit; no file is a byte copy of a third-party asset; kit shaders compile |

**Deliberate isolation note.** Every fixture except `SearchIntegrityTests` clears
the transposition table in setup, and says so in a comment. `SearchIntegrityTests`
exists precisely to run *without* that clear — if you add a fixture, state which
side you are on and why (SI-R12).

## 5. Environment matrix

Every row is a full L3 manual pass in a fresh project. Rows marked **Core** are
mandatory for any release; the rest are mandatory for a first submission and for
any change that could plausibly affect them.

| # | Unity | Pipeline | Active Input Handling | DOTween | Priority |
|---|---|---|---|---|---|
| E1 | 6000.5.9f1 | URP | New Input System | Absent | **Core** |
| E2 | 6000.5.9f1 | URP | New Input System | Present | **Core** |
| E3 | 6000.5.9f1 | Built-in | Input Manager (old) | Absent | **Core** |
| E4 | 6000.5.9f1 | URP | Both | Present | Secondary |
| E5 | 6000.5.9f1 | Built-in | Both | Absent | Secondary |
| E6 | Latest Unity 6 patch ≠ dev version | URP | New Input System | Absent | Secondary |

**Target platforms** for build verification (PLAT cases in
[test-cases.md](test-cases.md)):

| Platform | Why it is on the list |
|---|---|
| Windows Standalone | Primary development target |
| WebGL | The default `PlayerPrefsChessGameStorage` is claimed to work there; the sliced search exists so a single-threaded target stays smooth |
| Android | Touch input path through `BoardPointerSource`; slower hardware exposes budget-vs-wall-clock assumptions |
| iOS *(if published as supported)* | Same as Android; only test if the store listing claims it |

**Do not** claim a platform on the store page that has not passed its PLAT case.

## 6. Strength testing policy

The kit deliberately makes **no numeric strength claim**, so no test asserts one.
The only automated strength guards are ordinal and permanent:

- `"hard"` never loses outright to `"easy"` in a self-play game.
- Each level reaches at least the depth of the level below it.
- `"hard"` never passes up an available mate.

Any change to search, budgets, pacing, or move selection **must** re-run
`DifficultyCalibrationTests` and, if the published table in the kit README moves,
update that table in the same change (BB-R19, SI-R9). A difficulty table that
overstates what ships is treated as a store-compliance defect, not a doc nit.

## 7. Entry criteria

A build enters a release-candidate test pass only when:

1. The EditMode suite is green — 100%, no ignored or skipped cases.
2. The Editor console is clean on scene load and on a full game: no errors, no
   new warnings from kit code.
3. `Documentation/README.md` and `Samples/README.md` describe the build under
   test (not the previous one).
4. Every change since the last pass has an entry in the traceability matrix, or
   a stated reason it needs none.

## 8. Exit criteria

A release candidate ships when:

1. All **P0** cases pass on every **Core** environment row.
2. All **P1** cases pass, or carry a written waiver naming the risk accepted.
3. Every target platform on the store listing has a passing PLAT case.
4. The package acceptance pass (L5) succeeds: exported package imports into an
   empty project, the sample scene plays a full game, and nothing lands outside
   the two permitted root folders.
5. No open defect at **S1** or **S2** (see [bug-report-template.md](bug-report-template.md)).
6. [release-checklist.md](release-checklist.md) is fully ticked and archived with
   the submission.

## 9. Risks

| Risk | Why it matters here | Mitigation |
|---|---|---|
| **Wall-clock budgets on slow hardware** | Difficulty is a time budget; a slow device searches less within it. Depth claims in the README are measured on the dev machine. | Depth assertions are ordinal, never absolute. PLAT-Android case checks the ladder still holds on device. |
| **Shared transposition table** | The most recently found class of defect. A search that reuses dirty state degrades silently — it still returns a legal move. | `SearchIntegrityTests` runs without clearing; node/depth floors mean a search that does not search *fails* (SI-R11). |
| **Optional dependencies flip a define** | `CHESS_DOTWEEN_SUPPORT` is set by an editor script. A project that adds or removes DOTween mid-flight changes which tween backend compiles. | E1/E2 pair, plus TOOL-04 (define add/remove round-trip). |
| **Screens are individually removable** | Every screen absent is a supported configuration; six screens means many combinations. | SESS cases test each screen absent individually, plus the all-absent case, rather than the full power set. |
| **No PlayMode suite** | Presentation, input, and animation regressions can only be caught by a human. | The manual scenarios in [test-scenarios.md](test-scenarios.md) are mandatory per release, not optional. |
| **EasyViewService duplication** | It ships at its own canonical path so an existing owner gets no duplicate — an import that lands it elsewhere breaks that promise. | PKG-03 verifies the folder layout of the exported package. |
| **Store compliance drift** | Extra files, stale notices, or a doc claim that no longer matches shipping behaviour are rejection causes. | PKG cases plus the documentation section of the release checklist. |

## 10. Roles and cadence

| When | What runs | Who |
|---|---|---|
| Every commit | L1 + L2 (EditMode suite) | Developer, before commit |
| Every PR touching engine/search/budgets | L1 + L2 + `DifficultyCalibrationTests` re-run, plus doc table check | Developer |
| Release candidate | L3 on Core rows + L4 on all rows + PLAT cases | Whoever cuts the release |
| Submission | L5 + [release-checklist.md](release-checklist.md) | Whoever submits |

## 11. Deliverables

- Test results XML from the headless run, archived per release.
- A completed copy of [release-checklist.md](release-checklist.md) per submission.
- Defect reports filed from [bug-report-template.md](bug-report-template.md).
- An updated [traceability-matrix.md](traceability-matrix.md) whenever a
  requirement is added or a case changes what it covers.
