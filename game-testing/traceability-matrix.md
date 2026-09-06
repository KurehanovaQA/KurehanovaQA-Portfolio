---
date: 2026-09-06
topic: advanced-chess-kit-traceability
---

# Advanced Chess Kit — Traceability Matrix

Requirement → test case → automated test. Use it two ways: to prove a
requirement is covered before shipping, and to find what to re-run after a
change.

**Sources**
`BB-*` — [2026-09-05 bug batch requirements](../brainstorms/2026-09-05-chess-kit-bug-batch-requirements.md)
`SI-*` — [2026-09-06 search integrity requirements](../brainstorms/2026-09-06-chess-kit-search-integrity-requirements.md)

**Coverage column**
`Auto` — an EditMode test fails if the requirement regresses.
`Manual` — only a human pass catches it.
`Both` — automated at the logic level, manual for the presentation.

---

## Bug batch — session flow (BB-R1…R7)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| BB-R1 | Session opens on difficulty selection when no resumable game exists | SESS-01, SC-01 | — | Manual |
| BB-R2 | Restart from any screen routes through difficulty selection | SESS-04, SC-04, SC-05 step 5, SC-06 step 5 | — | Manual |
| BB-R3 | Board menu button opens a pause menu with Resume and Restart only | SESS-06, SESS-07, SC-02 | — | Manual |
| BB-R4 | The outcome screen offers Restart | SESS-08, SC-05 | — | Manual |
| BB-R5 | Continue only for an in-progress save; finished ⇒ visible and disabled | SESS-02, SESS-03, SC-03, SC-06 | `ChessGameStorageTests.TryPeekOutcome_*` (the gating read) | Both |
| BB-R6 | Starting a new game discards the previous save at that moment | SESS-05, SC-04 | — | Manual |
| BB-R7 | The kit wires the loop by default; removing any part stays supported | SESS-10, SESS-11, SESS-12, SC-07 | — | Manual |

> **Gap worth knowing.** The session loop is the least automated area in the kit
> and the one a buyer meets first. SC-01 through SC-07 are therefore mandatory
> per release, not sampled.

## Bug batch — engine strength and pacing (BB-R8…R12)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| BB-R8 | Every difficulty plays the strongest move its search found | DIFF-01, DIFF-03, DIFF-05, SRCH-18 | `DifficultyCalibrationTests.EachDifficulty_SearchesAtLeastAsDeepAsTheOneBelowIt`, `.HardDifficulty_NeverPassesUpAnAvailableMate`, `MinimalChessGameTests.HardDifficulty_PicksTheSameReplyEveryTime`, `RankMovesCharacterizationTests.RankedAndSorted_TopChoice_IsTheSearchsOwnBestMove_EvenWhenTheRankingPassTruncates` | Auto |
| BB-R9 | Each difficulty gets its published budget as real search time | SRCH-15, SRCH-14, SRCH-13 | `ChunkedSearchTests.Advance_AccumulatesOnlyTimeSpentInsideAdvance_NotWallClockBetweenCalls`, `.Advance_DoesNotRunFarPastItsRequestedSlice`, `.Advance_WithATotalBudget_EventuallyFinishesUnderManySmallSlices` | Auto |
| BB-R10 | Reply lands 1.5–2.5 s after the reveal finishes, waiting out the remainder | DIFF-M1, DIFF-M2, DIFF-M3 | — | Manual |
| BB-R11 | No on-screen "thinking" text | DIFF-M4, SC-05 | — | Manual |
| BB-R12 | The best-move hint names the engine's genuine top choice | SRCH-18, SRCH-19, SRCH-21, VIEW-08 | `RankMovesCharacterizationTests.RankedAndSorted_TopChoice_…`, `.BeginHint_CreatesExactlyOneSearch_RegardlessOfDifficulty`, `.Hint_FindsAForcedMate_AtEveryDifficultysBudget` | Both |

## Bug batch — frame budget and input feel (BB-R13…R16)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| BB-R13 | Applying a player move costs no perceptible frame hitch | SRCH-M1, ASYNC-11, SC-05 step 2 | `AsyncReplyContractTests.ReadingExpectedPlayerReply_TriggersNoNewSearch` (the fix that removed the blocking search) | Both |
| BB-R14 | Move-revealed still reports whether the player played the top choice, at no extra search cost | ASYNC-07…11 | `AsyncReplyContractTests.ExpectedPlayerReply_*` | Auto |
| BB-R15 | A piece released on a legal square animates from the release point | VIEW-04, SC-01 step 9 | — | Manual |
| BB-R16 | Hint markers clear when the move they point at is accepted | VIEW-08, SC-05 | — | Manual |

## Bug batch — editor tooling and docs (BB-R17…R20)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| BB-R17 | Test Launcher Play opens a configured scene, defaulting to the kit sample | TOOL-05, SC-09 | — | Manual |
| BB-R18 | The session starts a game on a pending test position, no host code needed | TOOL-02, TOOL-06, SC-09, SC-11 step 5 | — | Manual |
| BB-R19 | The published difficulty table describes what ships | PKG-M6, DIFF-M1, DIFF-M2, DIFF-01 | `DifficultyCalibrationTests.*` (guards the ordering the table claims) | Both |
| BB-R20 | Both READMEs describe the shipped session loop and how to unplug it | PKG-M6, SESS-11, SESS-12 | — | Manual |

## Search integrity — correctness (SI-R1…R4)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| SI-R1 | Every search begins with a table holding no entries from a previous search | SRCH-04 | `SearchIntegrityTests.ConstructingASearch_EmptiesTheTranspositionTable` | Auto |
| SI-R2 | Two consecutive searches of the same position return the same move | SRCH-01, SRCH-02, DIFF-05 | `SearchIntegrityTests.TwoConsecutiveSearches_OfTheSamePosition_AgreeOnTheBestMove`, `.ASearchOfTheNextPosition_MatchesACleanSearchOfThatPosition` | Auto |
| SI-R3 | An interrupted iteration resumes at the same depth, not deeper | SRCH-05 | `SearchIntegrityTests.AnInterruptedIteration_DoesNotAdvanceTheDepthTarget` | Auto |
| SI-R4 | A completed search reports a move from a finished iteration | SRCH-05, SRCH-08, SRCH-11 | `SearchIntegrityTests.AnInterruptedIteration_…`, `ChunkedSearchTests.Advance_FindsMateInOne_AcrossASpreadOfSliceSizes`, `.Constructor_OnAnAlreadyCheckmatedPosition_ScoresItAsCheckmateNotZero` | Auto |

## Search integrity — frame budget (SI-R5…R7)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| SI-R5 | The per-frame cost leaves the majority of a 60 fps frame to the game | SRCH-M1, SRCH-14, PLAT-02 | `ChunkedSearchTests.Advance_DoesNotRunFarPastItsRequestedSlice` | Both |
| SI-R6 | The slice is an inspector-visible, documented setting with a stated trade-off | PKG-M6 | — | Manual |
| SI-R7 | Each difficulty's think window holds that difficulty's own budget | DIFF-M1, DIFF-M2, DIFF-01 | `DifficultyCalibrationTests.EachDifficulty_SearchesAtLeastAsDeepAsTheOneBelowIt` | Both |

## Search integrity — strength and docs (SI-R8…R9)

| Req | Requirement (abbreviated) | Cases | Automated test | Coverage |
|---|---|---|---|---|
| SI-R8 | Each difficulty reaches a depth distinguishable from its neighbours | DIFF-01, DIFF-02 | `DifficultyCalibrationTests.EachDifficulty_SearchesAtLeastAsDeepAsTheOneBelowIt`, `.EasyDoesNotBeatHard_OneQuickGame` | Auto |
| SI-R9 | The kit README's difficulty table describes measured post-fix behaviour | PKG-M6, DIFF-M1, DIFF-M2 | — | Manual |

## Search integrity — test-suite requirements (SI-R10…R12)

These are requirements *on the test suite itself*. They are verified by reading
the fixtures, not by running them.

| Req | Requirement (abbreviated) | Verified by | Coverage |
|---|---|---|---|
| SI-R10 | A test drives consecutive searches through one shared table without clearing, failing if a later search degrades | `SearchIntegrityTests` as a whole — the fixture has no `ClearTranspositionTable` setup | Auto |
| SI-R11 | A test asserts the search actually explores — a node/depth floor | `SearchIntegrityTests.ASearchFollowingAnotherSearch_ActuallyRuns`, `ChunkedSearchTests.Advance_DrivenInManySmallSlices_EventuallyReachesAMeaningfulDepth` | Auto |
| SI-R12 | Any fixture that clears the table in setup states why | Every fixture except `SearchIntegrityTests` carries the isolation comment. **Check this when adding a fixture.** | Review |

---

## Reverse index — what to re-run after a change

| If you change… | Re-run |
|---|---|
| `MinimalChessGame`, `OpeningBook`, rules code | `MinimalChessGameTests`, `CheckContractTests`, `ChessGameContractTests` + SC-05, SC-09 |
| `ChunkedSearch`, budgets, slice size, move selection | `ChunkedSearchTests`, `SearchIntegrityTests`, `RankMovesCharacterizationTests`, `DifficultyCalibrationTests` + DIFF-M1/M2, SRCH-M1 — **and update the README difficulty table if it moved** (BB-R19, SI-R9) |
| `ChessBoardHost`, async reply/hint lifecycle | `AsyncReplyContractTests`, `AsyncHintContractTests`, `ChessGameStorageTests` + ASYNC-M1, SC-05 |
| Save format, `IChessGameStorage`, PlayerPrefs storage | `ChessGameStorageTests`, `ChessGameContractTests` + SAVE-M1…M4, SC-03, SC-12 step 5 |
| `ChessKitSession`, routing, any screen | SC-01…SC-07 in full — automated coverage here is thin |
| `ChessBoardView`, input, tweening | SC-01, SC-05, SC-08, VIEW-* + SC-10 (both DOTween states) |
| Assembly definitions, defines, optional dependencies | Full EditMode suite + SC-10 + SC-11, on every Core environment row |
| Anything under `Runtime/MinimalChess/` | Full suite + PKG-M4 (fork markers and notices must still match) |
| Prefabs, skin, SFX, localization asset | SC-01, SC-08, SESS-14, SESS-15 |
| Documentation | PKG-M6, and re-read it against whatever DIFF-M1/M2 actually measured |
