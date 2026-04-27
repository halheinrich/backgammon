# Backgammon Umbrella Project

Main repo: https://github.com/halheinrich/backgammon
Local root: `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\`

## Stack

C# / .NET 10 / Visual Studio 2026 / Windows
Python / PyTorch (BgRLEngine only)

## Git submodules

| Folder | Repo |
| --- | --- |
| `BgDataTypes_Lib` | https://github.com/halheinrich/BgDataTypes_Lib |
| `ConvertXgToJson_Lib` | https://github.com/halheinrich/ConvertXgToJson_Lib |
| `XgFilter_Lib` | https://github.com/halheinrich/XgFilter_Lib |
| `ExtractFromXgToCsv` | https://github.com/halheinrich/ExtractFromXgToCsv |
| `XgAnalytics` | https://github.com/halheinrich/XgAnalytics |
| `BackgammonDiagram_Lib` | https://github.com/halheinrich/BackgammonDiagram_Lib |
| `BgDiag_Razor` | https://github.com/halheinrich/BgDiag_Razor |
| `BgRLEngine` | https://github.com/halheinrich/BgRLEngine |
| `BgMoveGen` | https://github.com/halheinrich/BgMoveGen |
| `BgQuiz_Blazor` | https://github.com/halheinrich/BgQuiz_Blazor |

## Naming convention

Always specify which Program.cs is being modified:

* Server: `ExtractFromXgToCsv\ExtractFromXgToCsv\Program.cs`
* Client: `ExtractFromXgToCsv\ExtractFromXgToCsv.Client\Program.cs`

## TestData

* Location: `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\TestData`
* `BothWays/` subfolder: `ThisWay.xg` and `ThatWay.xg` — same match, perspectives reversed
* Shared across all projects — do NOT put TestData inside individual project directories

---

## Subprojects

### BgDataTypes_Lib

**Purpose:** Shared type layer — BgDecisionData, DecisionRow, and constituent types. No parsing or rendering logic.
**Branch:** main
**Solution:** `BgDataTypes_Lib\BgDataTypes_Lib.slnx`

Key facts:

* Types: BgDecisionData (= PositionData + DecisionData + DescriptiveData), DecisionRow, PlayCandidate, AnalysisDepthEntry, CubeOwner enum
* `IDecisionFilterData` interface — implemented by both DecisionRow and BgDecisionData; enables XgFilter_Lib to filter either type
* `DecisionRow` migrated from ConvertXgToJson_Lib — CSV methods kept for pragmatic reasons
* `DecisionRow.Board` is `IReadOnlyList<int>` (26 elements); `MatchScore` computed from needs/Crawford/length
* All properties `init`-only; `CubeOwner` serializes as string

### ConvertXgToJson_Lib

**Purpose:** Reads .xg and .xgp files; produces DecisionRow and BgDecisionData records.
**Branch:** main
**Solution:** `ConvertXgToJson_Lib\ConvertXgToJson_Lib.slnx`
**Depends on:** BgDataTypes_Lib

Key facts:

* `XgDecisionIterator` yields DecisionRow; `IterateDiagramRequests` yields BgDecisionData
* Board: `board[0]` = opponent bar, `board[1–24]` = points, `board[25]` = player bar. Positive = player on roll; negative = opponent.
* XGID always normalized to bottom-player perspective
* Taker cube row board is always doubler POV — no flip
* XgIteratorState early-exit mechanism for match/game-level filtering
* XgFileReader.ReadMatchInfo / ReadGameHeaders — fast extraction without full parse

### XgFilter_Lib

**Purpose:** Filtering and classification for decision records.
**Branch:** main
**Solution:** `XgFilter_Lib\XgFilter_Lib.slnx`
**Depends on:** BgDataTypes_Lib, ConvertXgToJson_Lib

Key facts:

* All filters accept `IDecisionFilterData` — works with both DecisionRow and BgDecisionData
* `IDecisionFilter.Matches(IDecisionFilterData)`; `IMatchFilter` for match/game-level early-exit
* `DecisionFilterSet`: ordered list, AND semantics
* Filters: PlayerFilter, DecisionTypeFilter, MatchScoreFilter, ErrorRangeFilter, PositionTypeFilter, PlayTypeFilter
* Classifiers: RaceClassifier, ContactClassifier, InnerBoard631Classifier, InnerBoard54321Classifier. Multi-membership — a position can satisfy several classifiers at once.
* PlayType classifiers via `IPlayTypeClassifier`: Make20PtClassifier (only PlayType currently implemented).
* `IPositionClassifier.Matches` accepts `IReadOnlyList<int>`
* `ColumnSelector`: explicit column registry, no reflection
* `FilteredDecisionIterator`: owns XgIteratorState; supports early-exit via IMatchFilter

### ExtractFromXgToCsv

**Purpose:** Blazor web app — extracts decisions from .xg/.xgp files, applies filters, exports CSV or PPTX.
**Branch:** main
**Solution:** `ExtractFromXgToCsv\ExtractFromXgToCsv.slnx`
**Depends on:** ConvertXgToJson_Lib, XgFilter_Lib, BackgammonDiagram_Lib (server-side only, for PPTX)

Key facts:

* WASM — all .xg parsing, filtering, CSV generation runs client-side; server is thin host
* Razor pages/components live in `ExtractFromXgToCsv.Client`
* FilterPanel.razor owns filter UI state; raises `OnFiltersChanged EventCallback<DecisionFilterSet>`
* Local mode: server processes in background, client polls status
* Azure/browser mode: file upload works, CSV download button not yet implemented
* PPTX output: Local mode only — BackgammonDiagram_Lib is referenced from the server csproj so SkiaSharp stays out of the WASM payload. Web/Azure PPTX deferred.

### XgAnalytics

**Purpose:** Ad-hoc analysis tools against .xg/.xgp files.
**Branch:** main

Analyses: player match count, NonStandardStarts, MatchScoreDistribution

### BackgammonDiagram_Lib

**Purpose:** Pure rendering library — board diagrams as SVG, PNG, PDF, PowerPoint. No user interaction, no game state.
**Branch:** main
**Solution:** `BackgammonDiagram_Lib\BackgammonDiagram_Lib.slnx`
**Depends on:** BgDataTypes_Lib; ConvertXgToJson_Lib (test-only, for fixture-driven visual tests)

Key facts:

* `DiagramRequest` = BgDecisionData + rendering options; immutable class with inner Builder; validation at Build()
* SVG hand-rolled; QuestPDF for PDF; OpenXml for PPTX
* `HomeBoardOnRight` (bool, default true) — geometric reflection in ColumnCentreX
* `DiagramOptions` is a record; `ITheme Theme` direct reference (no string lookup)
* `ThemeRegistry`: static instances Default and Greyscale

### BgDiag_Razor

**Purpose:** Thin Razor Class Library wrapper — exposes `BackgammonDiagram.razor` component. Keeps BackgammonDiagram_Lib free of Blazor dependencies.
**Branch:** main
**Depends on:** BackgammonDiagram_Lib

Key facts:

* Click handling via transparent SVG overlay + pure Razor EventCallbacks — no JS interop
* EventCallbacks: OnPointClicked(1–24), OnBarClicked(25), OnCubeClicked, OnTrayClicked

### BgRLEngine

**Purpose:** Reinforcement learning engine — TD-Gammon style self-play training, ONNX export for C# inference.
**Branch:** main
**Language:** Python / PyTorch (files under `BgRLEngine/` subfolder)

Key facts:

* Portfolio-of-experts architecture with routing tree
* 303-feature board state encoding, ~145K parameter network
* Phase 2 (AlphaZero + MCTS) deferred

### BgMoveGen

**Purpose:** C# move generation library for backgammon and all variants.
**Branch:** main
**Solution:** `BgMoveGen\BgMoveGen.slnx`

Key facts:

* 3.4 μs/call, avoidance-based dedup
* Move generation: GenerateStates, EnumerateStates, NextMove
* Move formatting: MoveNotationFormatter.Format(Play) — standard notation
* BgRLEngine depends on this

### BgQuiz_Blazor

**Purpose:** Blazor quiz app for backgammon decisions.
**Branch:** main
**Solution:** `BgQuiz_Blazor\BgQuiz_Blazor.slnx`
**Depends on:** BgDiag_Razor

Key facts:

* Milestone 1 functional — diagram renders, orientation toggle, click reporting
* `CreateOpeningPosition()` still TODO

---

## Current status

| Subproject | Status |
| --- | --- |
| BgDataTypes_Lib | ✅ Complete |
| ConvertXgToJson_Lib | ✅ Complete |
| XgFilter_Lib | ✅ Complete — IDecisionFilterData migration done |
| BgMoveGen | ✅ Complete |
| ExtractFromXgToCsv | 🔧 In progress — Web/Azure PPTX, CSV download, ColumnSelector UI all deferred |
| XgAnalytics | 🔧 In progress |
| BackgammonDiagram_Lib | 🔧 In progress |
| BgDiag_Razor | 🔧 In progress |
| BgRLEngine | 🔧 In progress |
| BgQuiz_Blazor | 🔧 In progress — Milestone 1 done |

### Next up

**BgQuiz_Blazor multi-phase build-out** — Phase 0 architectural
review complete (read-only subproject session, 2026-04-27).
Substrate location decided: new `BgGame_Lib` subproject (not
BgQuiz_Blazor-internal), so the four eventual modes (scored quiz,
user-vs-user, user-vs-bot, bot-vs-bot tournament) share
scaffolding from day one. Concrete sessions, rank-ordered:

1. **BgDiag_Razor / BackgammonDiagram_Lib — orientation-flip
   respects hit regions.** `GetHitRegions` / `BoardHitRegions`
   must honour `HomeBoardOnRight` so the click overlay aligns
   with the rendered SVG after flip. Already documented as a
   known pitfall in `BgQuiz_Blazor/INSTRUCTIONS.md`. Lower-layer
   fix preferred (in `DiagramRenderer.GetHitRegions` / the
   `BoardHitRegions` model), so any future consumer of hit
   regions inherits the fix. Unblocks every click-driven feature
   past Milestone 1.

2. **BackgammonDiagram_Lib — `DiagramRequest.FromDecision`
   factory.** Adapter from `BgDecisionData` + rendering options
   to `DiagramRequest`. Sibling to the already-planned
   `FromBoard` / `FromXgid` factories on the subproject's own
   next-steps list. Eliminates the consumer adapter Phase 1
   would otherwise grow.

3. **BgMoveGen — `BoardState.FromMop` / `ToMop` bridge.** Bridge
   between `BoardState.Points` (`int[26]`) and
   `IReadOnlyList<int>` (the shape used by `IDecisionFilterData`
   and `BgDecisionData.Position.Mop`). Both layouts already
   match — 26 elements, on-roll-relative, `[0]` opponent bar,
   `[25]` player bar. Tiny; precondition for any quiz-side
   legal-move generation against XG-sourced positions.

4. **XgFilter_Lib — `FilteredDecisionIterator` diagram-shape
   variant.** Add `IterateXgDirectoryDiagrams` /
   `IterateJsonDirectoryDiagrams` yielding `BgDecisionData`
   (already supported on the parser side via
   `XgDecisionIterator.IterateDiagramRequests`; not yet exposed
   through the filter wrapper). Phase 1 problem-set source.

5. **New subproject `BgGame_Lib` — scaffold + substrate.** Set up
   csproj / slnx / INSTRUCTIONS.md / test project; register as
   umbrella submodule; add to umbrella status table and
   dependency graph. Add `GameState` (board + match + cube
   state), `IPlayAgent`, `ICubeAgent`, `CubeAction` enum,
   skeletal `Referee` (turn sequencing, end-of-game),
   `Transcript` (state / decision / outcome tuples). The
   substrate scored-quiz, user-vs-user, user-vs-bot, and
   bot-vs-bot all share — built ahead of the modes per
   CLAUDE.md "Best-practice bias" (extensible shapes when
   multi-mode is the known target).

6. **BgMoveGen — `MoveEntryState` for click-driven Play
   assembly.** Incremental-click state machine against
   `MoveGenerator.GeneratePlays(state, d1, d2)`: tracks partial
   from/to clicks, exposes current partial play, legal-next-
   clicks, and completed `Play` once dice are exhausted.
   Handles doubles ordering ambiguity and undo of partial
   entries. Consumed by every quiz mode that takes human input.

7. **BgDataTypes_Lib — `SubmittedPlay` + `QuizScore`.** Init-only
   records: chosen `Play` + matched candidate index + equity
   loss + correctness flag; running-total record. (`CubeAction`
   lives in `BgGame_Lib` per #5.)

8. **BgQuiz_Blazor — Phase 1 implementation.** Wires #1–#7 into
   problem-set selection, click-to-Play, submit-and-score,
   running total. Uses `BgGame_Lib`'s substrate (single-position
   game with the quiz-grader as the second agent) so Phase 2+
   modes reuse the scaffolding without rewrite.

**Parallelism.** Items 2 and 3 are independent of item 1 and can
run in any order. Item 4 depends only on XgFilter_Lib itself.
Items 5–8 form the chain that delivers Phase 1 — 5 before 6
because `MoveEntryState` may want `IPlayAgent`-shaped hooks; 6
before 8; 7 anywhere before 8.

**Phase 2+** (answer tracking with weighted re-recurrence on
wrong answers, the three two-agent modes) builds on items 1–8
and gets queued after Phase 1 ships.

### Deferred

* CSV download button for Azure/browser mode
* PPTX download for Azure/browser mode (SkiaSharp native isn't available under Blazor WASM)
* ColumnSelector wired into UI
* ExtractFromXgToCsv 0-rows bug diagnosis (regression after XGID perspective fix)
* BgDiag_Razor: verify Blazor component layout under new 16:9 aspect default; adapt or pass `AspectPreset.Natural` if needed
* BgDataTypes_Lib: reinstate 2 cube-error adapter tests (`UsesDoubleError`, `FallsBackToTakeError`) dropped from XgFilter_Lib in `d8fac0d` when the filter-test suite was consolidated onto `DecisionFilterAsserts`. The fallback logic being tested lives in `BgDataTypes_Lib.BgDecisionData.FilterError`, not in the filter, so the tests belong with the type.
* XgFilter_Lib: `FilteredDecisionIterator` exception-swallowing. Catch block discards exceptions silently; either handle something real or delete the catch. Correctness concern masquerading as style.
* ExtractFromXgToCsv: two encapsulation leaks in `FilterPanel.razor` — PositionType checkbox label renders `@pt` (bare identifier) instead of `@pt.ToLabel()` around line 97; local `DecisionTypeLabel` switch around lines 253-259 should use `value.ToLabel()`. Same pattern Session 4 fixed for PlayType. Small single-session cleanup whenever the subproject next opens.
* ConvertXgToJson_Lib: dance-sentinel notation glitch. XG's no-move sentinel for a "dance" (closed-out roll) reaches the formatter and renders as `"1/1"`. Latent pre-existing bug surfaced during the BgMoveGen migration. Documented in subproject INSTRUCTIONS.md Pitfalls. Likely fix sites: `XgMoveTranslator` (filter sentinel before constructing `Play`) or upstream in `BuildMoveDiagramRequest`. Single-session ConvertXgToJson_Lib follow-up.

---

## Claude Project structure

| Claude Project | Purpose |
| --- | --- |
| **Backgammon Umbrella** ← you are here | Coordination, status, cross-cutting decisions |
| **BgDataTypes_Lib** | Shared type layer |
| **ConvertXgToJson_Lib** | .xg/.xgp reader library |
| **XgFilter_Lib** | Filter/classifier library |
| **ExtractFromXgToCsv** | Blazor CSV extraction app |
| **XgAnalytics** | Ad-hoc analysis tools |
| **BackgammonDiagram_Lib** | Rendering library |
| **BgDiag_Razor** | Razor wrapper for diagram lib |
| **BgRLEngine** | RL training engine |
| **BgMoveGen** | Move generation library |
| **BgQuiz_Blazor** | Blazor quiz app |

## Architecture — dependency graph

```
BgDataTypes_Lib (shared types)
├── ConvertXgToJson_Lib (parsing)
│   └── XgFilter_Lib (filtering)
│       └── ExtractFromXgToCsv (app)
├── BackgammonDiagram_Lib (rendering)
│   └── BgDiag_Razor (Blazor wrapper)
│       └── BgQuiz_Blazor (app)
├── BgPositionRouter (planned — position routing)
└── BgInference (planned — ONNX inference)

BgMoveGen (move generation + notation)
└── BgRLEngine (standalone, Python)

XgAnalytics (standalone)
```

Cross-edges not shown in the tree:

- ExtractFromXgToCsv also consumes BackgammonDiagram_Lib server-side for PPTX output.
- BackgammonDiagram_Lib's test project references ConvertXgToJson_Lib for fixture-driven visual tests.
- ConvertXgToJson_Lib consumes BgMoveGen for move-notation formatting (`MoveNotationFormatter.Format(Play)`).

## Pre-session verification

```powershell
cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git submodule foreach 'echo "$name $(git rev-parse --short HEAD) $(git log --oneline -1 origin/main)"'
```

Compare each line against `git log --oneline -1` for the umbrella. Any mismatch means stale pointers — resolve before starting work.