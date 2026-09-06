---
date: 2026-09-06
topic: advanced-chess-kit-test-scenarios
---

# Advanced Chess Kit — Test Scenarios

End-to-end journeys. Where [test-cases.md](test-cases.md) checks one behaviour in
isolation, a scenario walks a real path through the product and fails if any
seam along it is wrong.

Run these **in order** for a release candidate — SC-01 through SC-06 form a
continuous session that builds its own preconditions. SC-07 onward are
independent.

**Timing.** A full pass is roughly 60–90 minutes per environment row. Budget for
SC-05 (the full game) being the long one.

---

## SC-01 — First launch, a buyer's very first minute

*The most important scenario in this document. If this fails, nothing else
matters.*

| | |
|---|---|
| **Covers** | BB-R1, BB-R7 · SESS-01, VIEW-01, VIEW-02, VIEW-03 |
| **Pre** | Empty project with the kit imported. PlayerPrefs cleared. Sample scene open. |

1. Press Play without changing a single setting.
2. **Expect:** `DifficultySelectScreen` appears. No console errors. No missing
   sprites, no pink materials, no `MISSING:` text.
3. Choose **Normal**.
4. **Expect:** The screen fades out and a board appears with all 32 pieces
   correctly placed, rank/file labels legible.
5. Click a pawn.
6. **Expect:** It highlights as selected; its one or two legal destinations
   highlight too.
7. Click a highlighted destination.
8. **Expect:** The pawn animates there, the move SFX plays, the player cannot act
   for a moment, and the engine's reply animates in 1.5–2.5 s after the player's
   reveal finished.
9. Drag a second piece and release it on a legal square.
10. **Expect:** The piece follows the pointer and settles from the release point,
    not from its origin square.

**Fail conditions:** any console error; the board not appearing; a move being
accepted while the engine is computing; a reply that lands instantly or after
more than ~3 s.

---

## SC-02 — Pause and resume mid-game

| | |
|---|---|
| **Covers** | BB-R3 · SESS-06, SESS-07, VIEW-07 |
| **Pre** | Continues from SC-01 — a game in progress, player's turn. |

1. Press the board's menu button.
2. **Expect:** `PauseMenuScreen` opens showing Resume and Restart, and nothing
   else.
3. Click through the panel onto a piece behind it.
4. **Expect:** The board takes no input.
5. Choose **Resume**.
6. **Expect:** The exact same position, same side to move, same selection state.
   Play continues normally.
7. Make a move, and while the engine is computing, press the menu button again.
8. **Expect:** Nothing opens. The reply still lands normally afterwards.

---

## SC-03 — Interrupted session resumes exactly

| | |
|---|---|
| **Covers** | BB-R5 · SESS-02, SESS-13, SAVE-M1, SAVE-M2, DIFF-M5 |
| **Pre** | Continues from SC-02. Note the position and ply count before stopping. |

1. Stop play mode without restarting or finishing the game.
2. Press Play again.
3. **Expect:** `ContinueRestartScreen` appears with **Continue enabled**.
4. Choose **Continue**.
5. **Expect:** The exact noted position, the correct side to move, and the
   difficulty chosen in SC-01 — verify by timing one reply against that level's
   published window.
6. Undo two plies (the sample's Undo button), then stop and re-enter and Continue
   once more.
7. **Expect:** The resumed game reflects the undo, not the pre-undo history.

---

## SC-04 — Restart discards, and cannot be backed out of

| | |
|---|---|
| **Covers** | BB-R2, BB-R6, BB-AE2 · SESS-04, SESS-05 |
| **Pre** | Continues from SC-03 — a game in progress. |

1. Open the pause menu and choose **Restart**.
2. **Expect:** `DifficultySelectScreen` appears immediately.
3. **Without choosing a level**, stop play mode. Press Play again.
4. **Expect:** Difficulty selection again — *not* continue-or-restart. The
   abandoned game is gone; it was discarded the moment Restart was chosen.
5. Choose **Easy** and play two moves.
6. **Expect:** A fresh game at Easy, with Easy's pacing.

---

## SC-05 — A full game to a real outcome

*The long one. Do not shortcut it with the Test Launcher — the point is that
nothing degrades over a hundred plies.*

| | |
|---|---|
| **Covers** | ENG-*, SESS-08, SESS-09, VIEW-08, VIEW-09, VIEW-10, DIFF-M4, SRCH-M1 |
| **Pre** | A new game at **Normal**, played to a terminal outcome. |

1. Play the game out to mate, resignation, or a draw. Along the way, deliberately
   hit each of these at least once:
   - a capture (capture SFX, captured piece removed cleanly);
   - a check, in either direction (`CheckNoticeScreen` shows and clears);
   - the **Best Move** button (markers appear; they clear when that move is
     played — BB-R16);
   - the **Undo** button (both plies revert, no ghost pieces);
   - a promotion if one arises (picker opens; chosen piece is placed).
2. Throughout, watch for: any frame hitch when a move is applied, any on-screen
   "thinking" text, any drift in reply pacing, and the demo buttons greying out
   whenever the player cannot act.
3. Play the final move.
4. **Expect:** The winning move's reveal **finishes first**, then `OutcomeScreen`
   appears — never simultaneously. It states the correct result and offers
   Restart.
5. Choose **Restart**.
6. **Expect:** Difficulty selection.

**Fail conditions:** a wrong rules verdict at any point; the outcome screen
racing the final animation; a hitch on move application; console errors
accumulating over the game.

---

## SC-06 — A finished save cannot be resumed

| | |
|---|---|
| **Covers** | BB-R5, BB-AE1 · SESS-03, SAVE-16 |
| **Pre** | Play a game to a **loss** and stop play mode from the outcome screen *without* pressing Restart. Fast path: Test Launcher preset *Mate to me* and let the engine mate you. |

1. Press Play.
2. **Expect:** `ContinueRestartScreen` with **Continue visible and disabled** —
   not hidden, not absent. Restart enabled.
3. Attempt to activate Continue by every means available (click, keyboard, gamepad
   navigation if present).
4. **Expect:** Nothing happens. No path leads back into the finished position.
5. Choose **Restart** and confirm a new game starts through difficulty selection.

---

## SC-07 — Every screen removed, the game still plays

| | |
|---|---|
| **Covers** | BB-R7 · SESS-10, SESS-11, SESS-12 |
| **Pre** | A copy of the sample scene. |

1. Remove all six screen objects and clear all six references on
   `ChessKitSession`. Play.
2. **Expect:** A game starts immediately at the default difficulty (`normal`).
   Promotion auto-queens. Play to an outcome — **zero** null-reference
   exceptions, including at game end.
3. Restore the scene, then run six short passes, each removing exactly one
   screen, and confirm the documented fallback for that screen (kit README's
   *UI Screens* table).
4. Finally, remove the `ChessKitSession` component entirely and start the game
   from a one-line script:
   `GetComponent<ChessBoardHost>().NewGame("normal", playerPlaysWhite: true);`
5. **Expect:** A fully playable game. `SaveToStorage()` / `TryLoadFromStorage()`
   still work when called by hand.

---

## SC-08 — Reskin and re-sound without touching code

| | |
|---|---|
| **Covers** | VIEW-11, VIEW-12, TOOL-01 |
| **Pre** | Sample scene. |

1. Duplicate `Art/DefaultBoardSkin`. Open
   `Ostryzhnyi ▸ Advanced Chess Kit ▸ Edit` and change the board sprite, two
   piece sprites, and the label colour on the copy.
2. **Expect:** The window's preview updates live. The original asset is untouched.
3. Point the board's skin field at the copy and Play.
4. **Expect:** The new art and label colour render. No code was changed, no
   prefab was edited.
5. Duplicate `Audio/DefaultBoardSfx`, swap the capture clip, point the board at
   it, play a capture.
6. **Expect:** The new clip plays, with the configured volume/pitch jitter.
7. Clear the board's `sfx` reference entirely and play a full exchange.
8. **Expect:** Complete silence, no warning, no exception.

---

## SC-09 — Rules edge cases through the Test Launcher

| | |
|---|---|
| **Covers** | ENG-M1, ENG-M2, ENG-M3, TOOL-02, TOOL-03, TOOL-05, TOOL-06, BB-R18 |
| **Pre** | Test Launcher open. |

1. For each of the seven presets: preview, press Play, and confirm the board
   shows exactly that position with the right side to move — and that **both**
   entry screens were skipped.
2. On *En passant*, execute the capture and confirm the captured pawn is removed
   from d5.
3. On *Castling*, castle each side and confirm the rook animates to its square.
4. On *Promotion*, promote to a knight through the picker and confirm a knight
   lands.
5. On *Mate to enemy*, deliver mate and confirm the outcome screen path.
6. Paste a malformed FEN.
7. **Expect:** A readable refusal, no exception, no half-loaded board.
8. Exit play mode, then press Play normally from the Editor.
9. **Expect:** Normal entry routing. The test position does **not** reappear.

---

## SC-10 — The optional-dependency round-trip

| | |
|---|---|
| **Covers** | TOOL-04, PKG-M2 · environment rows E1/E2 |
| **Pre** | An empty project with the kit imported and **no** DOTween. |

1. Confirm `CHESS_DOTWEEN_SUPPORT` is absent from the scripting defines. Play a
   few moves.
2. **Expect:** Clean compile, and pieces animate (through the bundled tween).
3. Import DOTween. Wait for the domain reload.
4. **Expect:** The define now appears for the active build target, without anyone
   editing it. Pieces still animate.
5. Remove DOTween.
6. **Expect:** The define is removed, the project still compiles, animation still
   works.

**Fail conditions:** a compile error in any of the three states; a manual define
edit being required; animation stopping in any state.

---

## SC-11 — Fresh-import package acceptance

*Run against the exported `.unitypackage`, not the working repo.*

| | |
|---|---|
| **Covers** | PKG-M1, PKG-M2, PKG-M3, PKG-M4, PKG-M5 · exit criterion 4 |
| **Pre** | A newly created empty Unity 6 project (URP), nothing else installed. |

1. Import the exported package.
2. **Expect:** Exactly `Ostryzhnyi/Advanced Chess Kit/` and
   `Ostryzhnyi/EasyViewService/`, plus `Third-Party Notices.txt` at the package
   root — nothing else anywhere in the project.
3. **Expect:** Zero compile errors and zero kit-originated warnings.
4. Open `Samples/AdvancedChessKitSample.unity` and run SC-01 in full.
5. Open both editor tools and confirm they work in this bare project.
6. Verify the third-party notices: MinimalChessEngine MIT text and © line
   present, fork changes described, and every file carrying an
   `Advanced Chess Kit fork` comment accounted for.
7. Repeat the import into a second project that **already has**
   EasyViewService installed at its canonical path.
8. **Expect:** No duplicate folder, no duplicate-type compile errors.

---

## SC-12 — On-device session

| | |
|---|---|
| **Covers** | PLAT-01…04, VIEW-15, SAVE-M3, DIFF-M1/M2 on device |
| **Pre** | A build for each platform the store listing claims. |

1. Install and launch. Play through difficulty selection into a game.
2. **Expect:** Touch (or mouse) input selects, drags, and releases correctly. A
   second simultaneous touch does not hijack a drag.
3. Play at least 15 plies, timing several replies.
4. **Expect:** Pacing inside the published window, or a documented slower-hardware
   caveat that is actually honest about what you observe. The UI stays responsive
   while the engine computes.
5. Background the app, force-close it from the task switcher, relaunch.
6. **Expect:** Continue-or-restart offers the game at the last played ply.
7. On WebGL specifically: reload the page mid-game and confirm the save persists;
   confirm the browser never shows a "page unresponsive" prompt during a reply.
8. Check the platform log (logcat / browser console) for errors across the whole
   session.

---

## Scenario-to-environment grid

Which scenarios run on which environment rows (see
[test-plan.md](test-plan.md#5-environment-matrix)):

| Scenario | E1 | E2 | E3 | E4 | E5 | E6 |
|---|:--:|:--:|:--:|:--:|:--:|:--:|
| SC-01 first launch | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| SC-02 pause/resume | ✔ | ✔ | ✔ | | | |
| SC-03 interrupted session | ✔ | ✔ | ✔ | | | |
| SC-04 restart discards | ✔ | ✔ | ✔ | | | |
| SC-05 full game | ✔ | ✔ | ✔ | | | |
| SC-06 finished save | ✔ | ✔ | | | | |
| SC-07 screens removed | ✔ | | | | | |
| SC-08 reskin/re-sound | ✔ | | | | | |
| SC-09 rules edge cases | ✔ | ✔ | | | | |
| SC-10 dependency round-trip | ✔ | ✔ | | ✔ | | |
| SC-11 package acceptance | ✔ | | ✔ | | | |
| SC-12 on-device | — platform builds, not editor rows — | | | | | |
