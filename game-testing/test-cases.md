---
date: 2026-09-06
topic: advanced-chess-kit-test-cases
---

# Advanced Chess Kit — Test Cases

Atomic, independently executable cases. IDs are stable — retire, never reuse.

**Legend**
`Pri` — P0 blocks release / P1 blocks unless waived / P2 polish.
`Type` — `Auto` (an EditMode test owns it) / `Manual` / `Auto+Manual`.
`Covered by` — the automating fixture and test method, where one exists.

Unless a case says otherwise, the environment is: sample scene
`Samples/AdvancedChessKitSample.unity`, Editor Play mode, default skin, default
SFX, `PlayerPrefsChessGameStorage`, no pending test position.

**Resetting between cases.** Many cases depend on there being no save. Clear it
with `Edit ▸ Clear All PlayerPrefs`, or from a script:
`PlayerPrefs.DeleteAll(); PlayerPrefs.Save();`. Cases that need a save present
say so in their precondition.

---

## ENG — Engine and rules

Automated. These are the correctness floor; a failure here is P0 by definition.

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| ENG-01 | Legal moves from a square list exactly the piece's legal destinations | P0 | Auto | `MinimalChessGameTests.LegalMovesFrom_ListsTheKnightsTwoOpeningMoves` |
| ENG-02 | An empty square yields no legal moves | P0 | Auto | `MinimalChessGameTests.LegalMovesFrom_IsEmptyForAnEmptySquare` |
| ENG-03 | The opponent's piece yields no legal moves for the player | P0 | Auto | `MinimalChessGameTests.LegalMovesFrom_IsEmptyForTheOpponentsPiece` |
| ENG-04 | An illegal move is rejected and leaves the game unchanged | P0 | Auto | `MinimalChessGameTests.IllegalMove_IsRejectedAndGameUnchanged` |
| ENG-05 | Garbage move text is rejected without throwing | P0 | Auto | `MinimalChessGameTests.GarbageMoveText_IsRejectedWithoutThrowing` |
| ENG-06 | A legal move applies and the engine replies automatically | P0 | Auto | `MinimalChessGameTests.LegalMove_AppliesAndEngineRepliesAutomatically` |
| ENG-07 | Board FEN reflects the played move | P0 | Auto | `MinimalChessGameTests.BoardFen_ReflectsThePlayedMove` |
| ENG-08 | Fool's mate ends the game as a player loss | P0 | Auto | `MinimalChessGameTests.FoolsMate_EndsTheGameWithPlayerLost` |
| ENG-09 | Resign ends the game as a player loss | P0 | Auto | `MinimalChessGameTests.Resign_EndsGameAsPlayerLost` |
| ENG-10 | Undo reverts both the board and the history | P0 | Auto | `MinimalChessGameTests.Undo_RevertsBoardAndHistory` |
| ENG-11 | A checkmated position reports no legal moves | P0 | Auto | `MinimalChessGameTests.CheckmatePosition_HasNoLegalMoves` |
| ENG-12 | Threefold repetition ends the game as a draw | P0 | Auto | `MinimalChessGameTests.ThreefoldRepetition_EndsTheGameAsADraw` |
| ENG-13 | Insufficient material is a draw — bare kings, K+B, K+N, same-colour bishops | P0 | Auto | `MinimalChessGameTests.InsufficientMaterial_IsADraw` (4 `[TestCase]`s) |
| ENG-14 | Sufficient material keeps the game in progress — opposite-colour bishops, two knights, a promotable pawn, a rook | P0 | Auto | `MinimalChessGameTests.SufficientMaterial_KeepsTheGameInProgress` (5 `[TestCase]`s) |
| ENG-15 | Capturing the last piece into a bare-kings position ends the game as a draw | P1 | Auto | `MinimalChessGameTests.CapturingTheLastPiece_EndsTheGameAsADraw` |
| ENG-16 | A player promotion is accepted and places the chosen piece | P0 | Auto | `MinimalChessGameTests.PlayerPromotion_IsAcceptedAndPlacesAQueen` |
| ENG-17 | `IsPromotion` is true only for a legal promoting move | P0 | Auto | `MinimalChessGameTests.IsPromotion_TrueOnlyForALegalPromotingMove` |
| ENG-18 | Starting as Black, the engine auto-plays White's opening move | P0 | Auto | `MinimalChessGameTests.NewGame_AsBlack_EngineAutoPlaysWhitesOpeningMove` |
| ENG-19 | A FEN-started game with Black to move at ply 0 reports the side to move correctly | P1 | Auto | `MinimalChessGameTests.IsPlayerToMove_IsCorrect_ForAFenStartedGameWithBlackToMoveAtPlyZero` |
| ENG-20 | `LastMove` is null before any move and reflects the engine's reply after one | P1 | Auto | `MinimalChessGameTests.LastMove_IsNullBeforeAnyMovesArePlayed`, `LastMove_ReflectsTheEnginesReply` |
| ENG-21 | The opening book supplies the known reply to 1.e4 | P1 | Auto | `MinimalChessGameTests.OpeningBook_SuppliesTheKnownReplyToE4` |
| ENG-22 | Check is reported with the correct side to move when delivered | P0 | Auto | `CheckContractTests.MoveDeliveringCheck_ReportsItWithTheCorrectSideToMove` |
| ENG-23 | Check clears when the move that removes it is played | P0 | Auto | `CheckContractTests.MoveRemovingCheck_ReportsNoCheck` |
| ENG-24 | A mating move reports both check and the outcome | P0 | Auto | `CheckContractTests.MoveDeliveringMate_ReportsBothCheckAndOutcome` |
| ENG-25 | A game restored into a check position reports check immediately | P1 | Auto | `CheckContractTests.RestoredGame_InACheckPosition_ReportsCheckImmediately` |
| ENG-26 | Header tracks ply count and side to move | P1 | Auto | `ChessGameContractTests.Header_TracksPlyCountAndSideToMove` |
| ENG-27 | Undo is refused for more plies than were played | P1 | Auto | `ChessGameContractTests.Undo_RejectsMorePliesThanPlayed` |
| ENG-28 | A finished game reports its outcome and refuses undo | P0 | Auto | `ChessGameContractTests.FinishedGame_ReportsOutcomeAndRefusesUndo` |

### Manual rules cases

These need the board, not just the game object — they verify the *player-facing*
result of a rule, which no EditMode test sees.

**ENG-M1 — En passant is offered and executes on the board** · P0 · Manual
- **Pre:** Test Launcher, preset *En passant* (`4k3/8/8/3pP3/8/8/8/4K3 w - d6 0 1`), Play.
- **Steps:** Select the e5 pawn.
- **Expected:** d6 is highlighted as a legal destination. Releasing on d6 moves the pawn to d6 **and removes the black pawn from d5**, with the capture SFX, not the move SFX.

**ENG-M2 — Castling both sides executes as a two-piece move** · P0 · Manual
- **Pre:** Test Launcher, preset *Castling (both sides)* (`r3k2r/8/8/8/8/8/8/R3K2R w KQkq - 0 1`), Play.
- **Steps:** Select the king on e1. Move it to g1. Restart the position and repeat to c1.
- **Expected:** Both g1 and c1 are highlighted. Kingside: king lands g1, rook animates e1→f1 (h1→f1). Queenside: king lands c1, rook a1→d1. The rook animates; it does not teleport.

**ENG-M3 — Promotion opens the picker and places the chosen piece** · P0 · Manual
- **Pre:** Test Launcher, preset *Promotion* (`4k3/1P6/8/8/8/8/8/4K3 w - - 0 1`), Play. `PromotionPickerScreen` assigned.
- **Steps:** Move the b7 pawn to b8. Choose Knight in the picker.
- **Expected:** The picker opens *before* the move resolves, shows four pieces in the player's colour, and the square ends with a knight — not a queen.

**ENG-M4 — Stalemate and draw outcomes reach the outcome screen** · P1 · Manual
- **Pre:** A position that will draw (Test Launcher, any insufficient-material FEN, e.g. `8/8/4k3/8/8/4K3/8/5B2 w - - 0 1`).
- **Steps:** Play until the game ends.
- **Expected:** `OutcomeScreen` shows a draw wording — not a win or loss — and offers Restart.

---

## SRCH — Search integrity

Automated. This area exists because a broken search still returns a legal move —
these are the cases that make silence fail loudly.

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| SRCH-01 | Two consecutive searches of the same position agree on the best move | P0 | Auto | `SearchIntegrityTests.TwoConsecutiveSearches_OfTheSamePosition_AgreeOnTheBestMove` |
| SRCH-02 | A search of the next position matches a clean search of that position | P0 | Auto | `SearchIntegrityTests.ASearchOfTheNextPosition_MatchesACleanSearchOfThatPosition` |
| SRCH-03 | A search following another search actually runs (node/depth floor) | P0 | Auto | `SearchIntegrityTests.ASearchFollowingAnotherSearch_ActuallyRuns` |
| SRCH-04 | Constructing a search empties the transposition table | P0 | Auto | `SearchIntegrityTests.ConstructingASearch_EmptiesTheTranspositionTable` |
| SRCH-05 | An interrupted iteration does not advance the depth target | P0 | Auto | `SearchIntegrityTests.AnInterruptedIteration_DoesNotAdvanceTheDepthTarget` |
| SRCH-06 | Ranking root moves when no iteration ever completed does not throw | P1 | Auto | `SearchIntegrityTests.RankingRootMoves_WhenNoIterationEverCompleted_DoesNotThrow` |
| SRCH-07 | Given no time at all, a search still returns a legal move | P0 | Auto | `ChunkedSearchTests.Advance_GivenNoTimeAtAll_StillReturnsALegalMove` |
| SRCH-08 | Mate in one is found across a spread of slice sizes | P0 | Auto | `ChunkedSearchTests.Advance_FindsMateInOne_AcrossASpreadOfSliceSizes` |
| SRCH-09 | An interrupted search does not corrupt the shared table for a later search | P0 | Auto | `ChunkedSearchTests.InterruptedSearch_DoesNotCorruptTheSharedTranspositionTableForALaterSearch` |
| SRCH-10 | A single legal move returns without needing the full budget | P1 | Auto | `ChunkedSearchTests.Advance_SingleLegalMove_ReturnsItWithoutNeedingTheFullBudget` |
| SRCH-11 | An already-checkmated position scores as checkmate, not zero | P0 | Auto | `ChunkedSearchTests.Constructor_OnAnAlreadyCheckmatedPosition_ScoresItAsCheckmateNotZero` |
| SRCH-12 | Root ranking on a position with no legal moves is empty, not an exception | P1 | Auto | `ChunkedSearchTests.RankedRootMoves_OnAPositionWithNoLegalMoves_IsEmptyNotAnException` |
| SRCH-13 | A total budget completes under many small slices | P0 | Auto | `ChunkedSearchTests.Advance_WithATotalBudget_EventuallyFinishesUnderManySmallSlices` |
| SRCH-14 | A slice does not run far past its requested size | P0 | Auto | `ChunkedSearchTests.Advance_DoesNotRunFarPastItsRequestedSlice` |
| SRCH-15 | Budget accumulates time spent *inside* Advance, not wall clock between calls | P0 | Auto | `ChunkedSearchTests.Advance_AccumulatesOnlyTimeSpentInsideAdvance_NotWallClockBetweenCalls` |
| SRCH-16 | Many small slices still reach a meaningful depth | P0 | Auto | `ChunkedSearchTests.Advance_DrivenInManySmallSlices_EventuallyReachesAMeaningfulDepth` |
| SRCH-17 | After a slice-limited search, most root moves score distinct and non-zero | P1 | Auto | `ChunkedSearchTests.RankedRootMoves_AfterASliceLimitedSearch_ScoresDistinctNonZeroForMostRootMoves` |
| SRCH-18 | The ranked top choice is the search's own best move even when the ranking pass truncates | P0 | Auto | `RankMovesCharacterizationTests.RankedAndSorted_TopChoice_IsTheSearchsOwnBestMove_EvenWhenTheRankingPassTruncates` |
| SRCH-19 | A hint creates exactly one search regardless of difficulty | P1 | Auto | `RankMovesCharacterizationTests.BeginHint_CreatesExactlyOneSearch_RegardlessOfDifficulty` (3 `[TestCase]`s) |
| SRCH-20 | An engine reply creates exactly one search regardless of difficulty | P1 | Auto | `RankMovesCharacterizationTests.EngineReply_AlsoCreatesExactlyOneSearch_RegardlessOfDifficulty` (3 `[TestCase]`s) |
| SRCH-21 | A hint finds a forced mate at every difficulty's budget | P1 | Auto | `RankMovesCharacterizationTests.Hint_FindsAForcedMate_AtEveryDifficultysBudget` (3 `[TestCase]`s) |
| SRCH-22 | A hint takes the same frame count regardless of difficulty | P1 | Auto | `RankMovesCharacterizationTests.Hint_TakesTheSameFrameCount_RegardlessOfDifficulty` |

**SRCH-M1 — The search leaves the frame usable** · P0 · Manual · *covers SI-R5*
- **Pre:** Sample scene, difficulty `hard`, Profiler open (or a visible frame-rate readout).
- **Steps:** Make a move and watch the frame while the engine computes its reply.
- **Expected:** No hitch on the frame the player's move is applied (BB-R13). During computation the frame rate stays at or near target — the board's own animations run smoothly and the pause menu opens without stutter. The search is visibly *not* consuming a whole frame.

---

## DIFF — Difficulty and pacing

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| DIFF-01 | Each difficulty searches at least as deep as the one below it | P0 | Auto | `DifficultyCalibrationTests.EachDifficulty_SearchesAtLeastAsDeepAsTheOneBelowIt` |
| DIFF-02 | `easy` does not beat `hard` in a self-play game | P0 | Auto | `DifficultyCalibrationTests.EasyDoesNotBeatHard_OneQuickGame` |
| DIFF-03 | `hard` never passes up an available mate | P0 | Auto | `DifficultyCalibrationTests.HardDifficulty_NeverPassesUpAnAvailableMate` |
| DIFF-04 | An unknown difficulty id uses the default budget rather than throwing | P0 | Auto | `DifficultyCalibrationTests.UnknownDifficultyId_UsesTheDefaultBudget_NotAnException` |
| DIFF-05 | `hard` picks the same reply every time for the same position | P0 | Auto | `MinimalChessGameTests.HardDifficulty_PicksTheSameReplyEveryTime` |

**DIFF-M1 — Reply lands inside the published window (easy / normal)** · P0 · Manual · *covers BB-R10*
- **Pre:** Sample scene, difficulty `easy`. Repeat for `normal`.
- **Steps:** Play 5 moves in the middlegame. Time from the moment the player's move *finishes animating* to the moment the engine's piece starts moving.
- **Expected:** Every reply lands **1.5–2.5 s** after the reveal finishes. Consistent across the five, not tracking how hard the position is.

**DIFF-M2 — Reply lands inside the published window (hard)** · P0 · Manual
- **Pre:** Same, difficulty `hard`.
- **Expected:** Every reply lands **2.5–3.5 s** after the reveal finishes.

**DIFF-M3 — Opening-book reply still waits out the window** · P1 · Manual · *covers BB-AE3*
- **Pre:** New game as White, difficulty `normal`.
- **Steps:** Play 1.e4 — a move the opening book answers with no search at all. Time the reply.
- **Expected:** The reply still lands no sooner than 1.5 s after the reveal finishes. It does not snap back instantly.

**DIFF-M4 — No "thinking" text anywhere** · P1 · Manual · *covers BB-R11*
- **Steps:** Play a full game at `hard`, watching for on-screen status text during every reply.
- **Expected:** No label, spinner, or text announces that the engine is thinking. The only feedback is that the player cannot act.

**DIFF-M5 — Difficulty selection is honoured** · P0 · Manual
- **Steps:** Start three games via `DifficultySelectScreen`, one per level. In each, save mid-game, quit play mode, re-enter, Continue.
- **Expected:** The resumed game's reply pacing matches the level that was chosen, not the default (the header carries difficulty through the round-trip — see SAVE-05).

---

## SAVE — Persistence

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| SAVE-01 | A saved game round-trips position and turn | P0 | Auto | `ChessGameContractTests.SavedGame_RoundTripsPositionAndTurn` |
| SAVE-02 | A restored game allows undo of two plies | P0 | Auto | `ChessGameContractTests.RestoredGame_AllowsUndoOfTwoPlies` |
| SAVE-03 | Save/load round-trips and the game remains playable | P0 | Auto | `MinimalChessGameTests.SaveLoad_RoundTripsAndRemainsPlayable` |
| SAVE-04 | Undo after loading a saved game works | P1 | Auto | `MinimalChessGameTests.Undo_WorksAfterLoadingASavedGame` |
| SAVE-05 | The header carries difficulty through a round-trip | P0 | Auto | `ChessGameContractTests.Header_CarriesDifficultyThroughRoundTrip` |
| SAVE-06 | `playerPlaysBlack` survives a round-trip | P1 | Auto | `MinimalChessGameTests.SaveLoad_PreservesPlayerPlaysBlack` |
| SAVE-07 | The outcome survives a round-trip | P1 | Auto | `ChessGameContractTests.Outcome_SurvivesRoundTrip` |
| SAVE-08 | Undo truncates history for subsequent saves | P1 | Auto | `ChessGameContractTests.Undo_TruncatesHistoryForSubsequentSaves` |
| SAVE-09 | An unreadable state fails without throwing | P0 | Auto | `ChessGameContractTests.UnreadableState_FailsWithoutThrowing` |
| SAVE-10 | A tampered move sequence is rejected | P0 | Auto | `MinimalChessGameTests.TryLoad_RejectsATamperedMoveSequence` |
| SAVE-11 | An old-format save is rejected | P0 | Auto | `MinimalChessGameTests.TryLoad_RejectsOldFormatSave` |
| SAVE-12 | A truncated save string is rejected, not partially restored | P0 | Auto | `ChessGameStorageTests.TruncatedSaveString_IsRejected_NotPartiallyRestored` |
| SAVE-13 | A substituted `IChessGameStorage` receives the writes, not the default | P0 | Auto | `ChessGameStorageTests.SubstitutedStorage_ReceivesWriteCalls_NotTheDefault` |
| SAVE-14 | Save-then-load through storage mid-game continues the game | P0 | Auto | `ChessGameStorageTests.SaveThenLoadThroughStorage_MidGame_Continues` |
| SAVE-15 | `TryLoadFromStorage` with nothing ever saved fails cleanly | P0 | Auto | `ChessGameStorageTests.TryLoadFromStorage_WithNothingEverSaved_Fails` |
| SAVE-16 | Restoring a checkmated game reports the outcome immediately and refuses input | P0 | Auto | `ChessGameStorageTests.RestoringACheckmatedGame_ReportsTheOutcomeImmediately_AndRefusesInput` |
| SAVE-17 | Outcome peek reads the outcome without replaying the move list | P1 | Auto | `ChessGameStorageTests.TryPeekOutcome_ReadsTheOutcome_WithoutReplayingTheMoveList` |
| SAVE-18 | Outcome peek on an in-progress game reports in-progress | P1 | Auto | `ChessGameStorageTests.TryPeekOutcome_OnAnInProgressGame_ReportsInProgress` |
| SAVE-19 | Outcome peek on an unreadable or foreign string returns false | P1 | Auto | `ChessGameStorageTests.TryPeekOutcome_OnAnUnreadableOrForeignFormatString_ReturnsFalse` |
| SAVE-20 | Save during reply computation restores to before the reply and resumes computing | P0 | Auto | `AsyncReplyContractTests.SaveDuringComputation_RestoresToBeforeTheReply_AndResumesComputing` |

**SAVE-M1 — The save survives quitting play mode** · P0 · Manual
- **Steps:** Start a game, play 4 plies, exit play mode, re-enter.
- **Expected:** Continue-or-restart appears with Continue **enabled**. Continue restores the exact position, side to move, and difficulty.

**SAVE-M2 — The save is flushed at session boundaries, not only per ply** · P0 · Manual · *covers BB-R6 flush half*
- **Steps:** Start a game, play 2 plies, open the pause menu, then kill the Editor's play mode hard (Stop) — or on device, force-quit the app.
- **Expected:** Re-entering finds the save at the position it was at when the menu opened. No lost plies.

**SAVE-M3 — Application pause/quit flushes on device** · P1 · Manual
- **Pre:** An Android or iOS build.
- **Steps:** Play a few plies, background the app via the home gesture, force-close it from the task switcher, relaunch.
- **Expected:** The game resumes at the last played ply.

**SAVE-M4 — A corrupted PlayerPrefs entry degrades to a new game** · P1 · Manual
- **Steps:** Save a game, then overwrite the stored save string with garbage (via a small editor script writing to the same PlayerPrefs key), enter play mode.
- **Expected:** No exception in the console. The session routes to difficulty selection as if there were no save.

---

## ASYNC — Reply and hint lifecycle

Everything about what the player is and is not allowed to do while the engine
works. Automated at the contract level; ASYNC-M1 covers what it looks like.

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| ASYNC-01 | Playing a move leaves the game computing until advanced | P0 | Auto | `AsyncReplyContractTests.TryPlayMove_LeavesTheGameComputing_UntilAdvanced` |
| ASYNC-02 | A move submitted during computation is rejected, game unchanged | P0 | Auto | `AsyncReplyContractTests.TryPlayMove_DuringComputation_IsRejectedAndGameUnchanged` |
| ASYNC-03 | Undo during computation is rejected | P0 | Auto | `AsyncReplyContractTests.TryUndo_DuringComputation_IsRejected` |
| ASYNC-04 | A player-delivered mate starts no reply computation | P0 | Auto | `AsyncReplyContractTests.PlayerDeliveredMate_DoesNotStartAReplyComputation` |
| ASYNC-05 | Resign during computation cancels it and reports the outcome | P0 | Auto | `AsyncReplyContractTests.Resign_DuringComputation_CancelsItAndReportsTheOutcome` |
| ASYNC-06 | A new game as Black starts with White's opening move computing | P0 | Auto | `AsyncReplyContractTests.NewGame_AsBlack_StartsWithWhitesOpeningMoveComputing` |
| ASYNC-07 | `ExpectedPlayerReply` is null before any reply has landed | P1 | Auto | `AsyncReplyContractTests.ExpectedPlayerReply_IsNull_BeforeAnyReplyHasLanded` |
| ASYNC-08 | `ExpectedPlayerReply` is set after a search-produced reply | P1 | Auto | `AsyncReplyContractTests.ExpectedPlayerReply_IsSetAfterASearchProducedReply` |
| ASYNC-09 | `ExpectedPlayerReply` is null after an opening-book reply | P1 | Auto | `AsyncReplyContractTests.ExpectedPlayerReply_IsNull_AfterAnOpeningBookReply` |
| ASYNC-10 | `ExpectedPlayerReply` is null after an undo | P1 | Auto | `AsyncReplyContractTests.ExpectedPlayerReply_IsNull_AfterAnUndo` |
| ASYNC-11 | Reading `ExpectedPlayerReply` triggers no new search | P0 | Auto | `AsyncReplyContractTests.ReadingExpectedPlayerReply_TriggersNoNewSearch` |
| ASYNC-12 | A hint leaves the game computing until advanced | P0 | Auto | `AsyncHintContractTests.BeginHint_LeavesTheGameComputing_UntilAdvanced` |
| ASYNC-13 | A hint requested while a reply is pending is rejected | P0 | Auto | `AsyncHintContractTests.BeginHint_WhileAReplyIsPending_IsRejected` |
| ASYNC-14 | A hint requested while another hint computes is rejected | P0 | Auto | `AsyncHintContractTests.BeginHint_WhileAnotherHintIsAlreadyComputing_IsRejected` |
| ASYNC-15 | A hint on a finished game does nothing | P1 | Auto | `AsyncHintContractTests.BeginHint_OnAFinishedGame_DoesNothing` |
| ASYNC-16 | A move during hint computation is rejected, game unchanged | P0 | Auto | `AsyncHintContractTests.TryPlayMove_DuringHintComputation_IsRejectedAndGameUnchanged` |
| ASYNC-17 | Undo during hint computation is rejected | P0 | Auto | `AsyncHintContractTests.TryUndo_DuringHintComputation_IsRejected` |
| ASYNC-18 | Resign during hint computation cancels it and reports the outcome | P0 | Auto | `AsyncHintContractTests.Resign_DuringHintComputation_CancelsItAndReportsTheOutcome` |
| ASYNC-19 | `HintMoves` is empty before any hint was ever requested | P2 | Auto | `AsyncHintContractTests.HintMoves_IsEmpty_BeforeAnyHintHasEverBeenRequested` |

**ASYNC-M1 — `CanPlayerAct` visibly gates the demo buttons** · P1 · Manual
- **Pre:** Sample scene (Best Move / Undo buttons present).
- **Steps:** Watch both buttons across: idle player turn → player move reveal → engine computing → engine reveal → game over.
- **Expected:** Both are interactable **only** on the player's idle turn. They grey out the instant the player's move is submitted and stay out until the engine's reveal finishes; once the game ends they stay out permanently. No click is silently swallowed.

---

## SESS — Session loop and screens

Almost entirely manual — this is the layer a buyer meets first.

**SESS-01 — No save routes to difficulty selection** · P0 · Manual · *BB-R1*
- **Pre:** PlayerPrefs cleared.
- **Expected:** On Play, `DifficultySelectScreen` shows. Choosing a level starts a game at that level. Continue-or-restart never appears.

**SESS-02 — An in-progress save routes to continue-or-restart, Continue enabled** · P0 · Manual · *BB-R1, BB-R5*
- **Pre:** A save from an unfinished game.
- **Expected:** `ContinueRestartScreen` shows. Continue is **enabled**. Choosing it resumes the exact position.

**SESS-03 — A finished save leaves Continue visible and disabled** · P0 · Manual · *BB-R5, BB-AE1*
- **Pre:** Play a game to a loss (Test Launcher preset *Mate to me*, let the engine mate you), then exit play mode without restarting.
- **Expected:** On the next Play, continue-or-restart shows with Continue **visible but disabled** — not hidden, not missing. Restart is enabled.

**SESS-04 — Restart from continue-or-restart routes through difficulty selection** · P0 · Manual · *BB-R2*
- **Steps:** From SESS-02's state, choose Restart.
- **Expected:** Difficulty selection shows. The old game is not resumable afterwards by any route.

**SESS-05 — Restart discards the save at that moment** · P0 · Manual · *BB-R6, BB-AE2*
- **Steps:** From a game in progress, open the pause menu, choose Restart, then **stop play mode from the difficulty screen without choosing a level**. Re-enter play mode.
- **Expected:** Difficulty selection shows again — the abandoned game is gone. It was discarded when Restart was chosen, not on the next applied ply.

**SESS-06 — The menu button opens a pause menu with Resume and Restart only** · P0 · Manual · *BB-R3*
- **Steps:** Mid-game, press the board's menu button.
- **Expected:** `PauseMenuScreen` shows exactly two actions: Resume and Restart. Resume returns to the live board with the position, selection state, and side to move unchanged.

**SESS-07 — The pause menu does not open while the player cannot act** · P1 · Manual
- **Steps:** Submit a move, and press the menu button while the engine is computing its reply.
- **Expected:** Nothing opens (`OpenPauseMenu` is a no-op while `CanPlayerAct` is false). Nothing breaks; the reply still lands normally.

**SESS-08 — The outcome screen shows after the winning move's reveal, and offers Restart** · P0 · Manual · *BB-R4*
- **Pre:** Test Launcher, preset *Mate to enemy* (`6k1/5ppp/8/8/8/8/8/R6K w - - 0 1`).
- **Steps:** Play the mating move.
- **Expected:** The mating piece finishes animating **first**; only then does `OutcomeScreen` appear — never simultaneously with the reveal. It names a win and offers Restart, which routes to difficulty selection.

**SESS-09 — The check notice appears and clears** · P1 · Manual
- **Pre:** Test Launcher, preset *In check* (`4k3/8/8/8/8/8/8/r3K3 w - - 0 1`).
- **Expected:** `CheckNoticeScreen` shows on entering the position and hides the moment the check is answered.

**SESS-10 — All six screens absent still plays a game** · P0 · Manual · *BB-R7*
- **Pre:** Sample scene, all six screen references on `ChessKitSession` cleared and the screen objects removed.
- **Expected:** On Play the board starts a game at the session's default difficulty (`normal`) with no screen at all. Promotion auto-queens. No null-reference exceptions in the console, at any point, including at game end.

**SESS-11 — Each screen absent individually** · P1 · Manual · *BB-R7*
- **Steps:** Six passes; each removes exactly one screen and plays a short game.
- **Expected:** Per the kit README's *UI Screens* table — no difficulty screen ⇒ default difficulty applies; no continue/restart ⇒ same fallback; no pause menu ⇒ menu button does nothing; no promotion picker ⇒ auto-queen; no check notice ⇒ no on-screen check; no outcome screen ⇒ no banner and no Restart button. No exceptions in any pass.

**SESS-12 — `ChessKitSession` removed entirely** · P0 · Manual · *BB-R7*
- **Pre:** Remove the `ChessKitSession` component; drive the board from a one-line script calling `NewGame("normal", true)`.
- **Expected:** A full game is playable. `SaveToStorage()` / `TryLoadFromStorage()` still work when called by hand. Nothing in the board depends on the session component.

**SESS-13 — The save is written every ply** · P1 · Manual
- **Steps:** Play a ply, then stop play mode without touching pause/outcome/restart. Re-enter and Continue.
- **Expected:** The resumed position includes that ply.

**SESS-14 — Screen text comes from `ChessKitStrings`** · P1 · Manual
- **Steps:** Edit an entry in `Localization/ChessKitStrings.asset` (e.g. the outcome win line) and re-enter play mode.
- **Expected:** The new text appears with no code change and no rebuild.

**SESS-15 — A missing localization key degrades, never crashes** · P1 · Manual
- **Steps:** Delete a key from `ChessKitStrings.asset` and trigger the screen that uses it.
- **Expected:** The screen shows `MISSING: <key>` and stays functional. No exception.

---

## VIEW — Board presentation, input, audio

**VIEW-01 — Board places all 32 pieces on a new game** · P0 · Manual
- **Expected:** Correct starting layout, correct orientation for the player's colour, rank/file labels legible in the skin's label colour.

**VIEW-02 — Click-select then click-destination moves a piece** · P0 · Manual
- **Expected:** Press on a piece selects it and highlights every legal destination. A press on a highlighted square plays the move.

**VIEW-03 — Drag and release plays a move** · P0 · Manual
- **Expected:** Press-drag shows the piece following the pointer; release on a legal square plays the move.

**VIEW-04 — A released piece animates from the release point** · P1 · Manual · *BB-R15*
- **Steps:** Drag a piece well past the centre of a legal destination square, then release.
- **Expected:** It settles from **where it was released**, not by snapping back to its origin square and re-animating from there.

**VIEW-05 — Release on the same square keeps the selection** · P1 · Manual
- **Expected:** The piece stays selected, highlights stay up. No move is attempted.

**VIEW-06 — Release off the board or on an illegal square snaps back** · P0 · Manual
- **Expected:** The piece returns to its origin square with no move played, no SFX beyond select, and no exception.

**VIEW-07 — A press over UI does not reach the board** · P0 · Manual
- **Steps:** With a screen open (pause menu), press through the panel onto a piece behind it.
- **Expected:** The board takes no input. Closing the screen restores normal input.

**VIEW-08 — Hint markers appear and clear when the move is played** · P1 · Manual · *BB-R16*
- **Steps:** Press Best Move, then play the suggested move.
- **Expected:** From/to markers appear on the suggested pair; they clear the moment that move is accepted — not on the next ply, not on the engine's reply.

**VIEW-09 — Undo reverts both plies visually** · P1 · Manual
- **Steps:** Press Undo (which calls `TryUndo(2)`).
- **Expected:** Both the player's move and the engine's reply are reverted on the board, pieces animate or reposition cleanly, no ghosts left behind.

**VIEW-10 — SFX fire on the right events** · P1 · Manual
- **Expected:** A distinct select clip on selection, move clip on a quiet move, capture clip on a capture — each with slight volume/pitch variation across repeats, not identical every time.

**VIEW-11 — An empty `sfx` field plays silently and never errors** · P0 · Manual
- **Steps:** Clear the board's `sfx` reference and play a game.
- **Expected:** Silence. No warning, no exception.

**VIEW-12 — A duplicated skin reskins the board with no code change** · P1 · Manual
- **Steps:** Duplicate `Art/DefaultBoardSkin`, replace the board sprite and two piece sprites and the label colour, point the board's field at the copy.
- **Expected:** The board renders with the new art and label colour immediately. The original asset is untouched.

**VIEW-13 — Board input can be switched off externally** · P2 · Manual
- **Steps:** Call `ChessBoardView.SetInteractable(false)` from a test script mid-game.
- **Expected:** Presses do nothing; `true` restores input, with no stuck selection.

**VIEW-14 — Board camera override** · P1 · Manual
- **Pre:** A scene where the board is *not* under `Camera.main`.
- **Steps:** Leave `boardCamera` unassigned, then assign it.
- **Expected:** Assigned, input maps to the right squares. (Unassigned in this setup is the documented misconfiguration — it should fail visibly, not silently mis-map, and must not throw.)

**VIEW-15 — Touch input drives the board** · P0 · Manual
- **Pre:** An Android build, or the Editor with Device Simulator and the new Input System.
- **Expected:** First active touch selects, drags, and releases exactly as the mouse does. A second simultaneous touch does not hijack the drag.

---

## TOOL — Editor tooling

**TOOL-01 — Skin editor shows a live board preview and writes to the asset** · P1 · Manual
- **Steps:** `Ostryzhnyi ▸ Advanced Chess Kit ▸ Edit`, change a piece sprite and the label colour.
- **Expected:** The preview updates live; the change is persisted to the `ChessBoardSkin` asset and survives a domain reload.

**TOOL-02 — Test Launcher presets each load the position they name** · P0 · Manual
- **Steps:** For each of the seven presets, preview it and press Play.
- **Expected:** The board shows exactly that position, playable, with the right side to move. The session skips **both** entry screens (BB-R18).

**TOOL-03 — Test Launcher accepts a pasted FEN and rejects a malformed one** · P1 · Manual
- **Steps:** Paste a valid FEN, then paste garbage.
- **Expected:** Valid FEN previews and launches. Garbage is refused with a readable message — no exception, no half-loaded board.

**TOOL-04 — DOTween define round-trip** · P0 · Manual
- **Steps:** In a project **without** DOTween, confirm `CHESS_DOTWEEN_SUPPORT` is *not* in the scripting defines and the board animates. Import DOTween; confirm the define appears for the active build target and the board still animates. Remove DOTween; confirm the define is removed and animation still works.
- **Expected:** All three states compile and animate. No manual define editing is ever required.

**TOOL-05 — Test Launcher target scene is configurable** · P1 · Manual · *BB-R17*
- **Steps:** Leave Target Scene at the default (the kit sample scene) and Play. Then point it at a different scene and Play.
- **Expected:** Play opens the configured scene before entering play mode, in both cases.

**TOOL-06 — A pending test position does not leak into a normal session** · P1 · Manual
- **Steps:** Launch a position from the Test Launcher, exit play mode, then press Play normally from the Editor.
- **Expected:** The normal entry routing applies (difficulty selection or continue/restart). The board does **not** silently reopen the test position.

---

## PKG — Packaging and store compliance

| ID | Case | Pri | Type | Covered by |
|---|---|---|---|---|
| PKG-01 | No kit asset depends on anything outside the kit | P0 | Auto | `KitSelfContainmentTests.NoKitAssetDependsOnAnythingOutsideTheKit` |
| PKG-02 | No kit file is a byte copy of a third-party asset | P0 | Auto | `KitSelfContainmentTests.NoKitFileIsAByteCopyOfAThirdPartyAsset` |
| PKG-03 | Kit shaders exist and compile | P0 | Auto | `KitSelfContainmentTests.KitShadersExistAndCompile` |

**PKG-M1 — The exported package installs into exactly two root folders** · P0 · Manual
- **Steps:** Export the package, import into an empty project, inspect what appeared.
- **Expected:** `Ostryzhnyi/Advanced Chess Kit/` and `Ostryzhnyi/EasyViewService/`, plus `Third-Party Notices.txt` at the package root — and nothing else, anywhere.

**PKG-M2 — Import into an empty project compiles clean** · P0 · Manual
- **Expected:** Zero compile errors, zero kit-originated warnings, on a project with no DOTween, no UniTask, and nothing else installed.

**PKG-M3 — An existing EasyViewService owner gets no duplicate** · P1 · Manual
- **Pre:** An empty project with EasyViewService already installed at its canonical path.
- **Expected:** Importing the kit overlays that path rather than creating a second copy elsewhere. No duplicate-type compile errors.

**PKG-M4 — Third-party notices are complete and accurate** · P0 · Manual
- **Expected:** `Third-Party Notices.txt` carries the MinimalChessEngine MIT text and © line, and describes the fork changes. `Runtime/MinimalChess/THIRD-PARTY-NOTICES.md` declares the fork. Every forked file is marked in place with an `Advanced Chess Kit fork` comment, and the marked set matches the notices.

**PKG-M5 — No stray editor state ships** · P1 · Manual
- **Expected:** No `.csproj`, no `Library/`, no test results, no `docs/` folder, no `.unitypackage`, and no asset belonging to a consuming application inside the exported package. The tests folder ships only if that is a deliberate decision recorded in the release checklist.

**PKG-M6 — Documentation matches shipping behaviour** · P0 · Manual · *BB-R19, BB-R20, SI-R9*
- **Expected:** The difficulty table's budgets, depth ranges, and reply windows match what DIFF-M1/M2 and `DifficultyCalibrationTests` actually produce. The README's public API table lists every public member and no member it no longer has. The screen inventory table matches `Prefabs/UI/`.

---

## PLAT — Platform verification

Each case is: build, install, play a full game to a terminal outcome, save and
resume once, and check the console/logcat for errors.

**PLAT-01 — Windows Standalone** · P0 · Manual
- **Expected:** Full game, correct pacing, save survives a restart of the executable.

**PLAT-02 — WebGL** · P0 · Manual
- **Expected:** Builds and runs single-threaded. The sliced search keeps the page responsive during replies (no browser "page unresponsive"). `PlayerPrefsChessGameStorage` persists across a page reload. Reply pacing is inside the published windows or the README's slower-hardware caveat is honest about it.

**PLAT-03 — Android** · P0 · Manual
- **Expected:** Touch input per VIEW-15, save survives a force-close (SAVE-M3), and the difficulty ladder ordering still holds on device even if absolute depths are lower.

**PLAT-04 — iOS** · P1 · Manual · *only if the store listing claims iOS*
- **Expected:** As PLAT-03.

---

## Appendix A — Test positions (FEN)

The Test Launcher's built-in presets:

| Preset | FEN | Use it for |
|---|---|---|
| Starting position | `rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1` | Baseline, opening book |
| Promotion | `4k3/1P6/8/8/8/8/8/4K3 w - - 0 1` | ENG-M3, promotion picker |
| En passant | `4k3/8/8/3pP3/8/8/8/4K3 w - d6 0 1` | ENG-M1 |
| Castling (both sides) | `r3k2r/8/8/8/8/8/8/R3K2R w KQkq - 0 1` | ENG-M2 |
| In check (must respond) | `4k3/8/8/8/8/8/8/r3K3 w - - 0 1` | SESS-09 |
| Mate to enemy (you mate in 1) | `6k1/5ppp/8/8/8/8/8/R6K w - - 0 1` | SESS-08, DIFF-03 |
| Mate to me (engine mates in 1) | `r6k/8/8/8/8/8/5PPP/6K1 b - - 0 1` | SESS-03, outcome-loss path |

Draw and insufficient-material positions used by the automated suite, useful by
hand too:

| Position | FEN |
|---|---|
| Bare kings | `8/8/4k3/8/8/4K3/8/8 w - - 0 1` |
| King + bishop v king | `8/8/4k3/8/8/4K3/8/5B2 w - - 0 1` |
| King + knight v king | `8/8/4k3/8/8/4K3/8/5N2 w - - 0 1` |
| Same-colour bishop each | `5b2/8/4k3/8/8/4K3/8/2B5 w - - 0 1` |
| Two bishops, opposite colours (**not** a draw) | `8/8/4k3/8/8/4K3/8/2B2B2 w - - 0 1` |
| Pawn can still promote (**not** a draw) | `8/8/4k3/8/8/4K3/4P3/8 w - - 0 1` |
| Rook mates (**not** a draw) | `8/8/4k3/8/8/4K3/8/5R2 w - - 0 1` |

## Appendix B — Coverage gaps (known, accepted)

Recorded so nobody re-derives them mid-pass:

- **No PlayMode automated tests.** Everything scene-level is manual by design.
  If a presentation regression escapes twice, that decision should be revisited.
- **No Elo calibration.** Ordinal guards only — see [test-plan.md](test-plan.md#6-strength-testing-policy).
- **Screen combinations are not exhaustively tested.** Each screen absent
  individually (SESS-11) and all absent (SESS-10); the 64-way power set is not
  covered.
- **`ChessKitEventSystemModuleSetup`** is verified through the environment matrix
  rows, not by a dedicated case per input-handling mode.
- **Localization** is verified only for "the key resolves"; no translated locale
  ships with the kit.
