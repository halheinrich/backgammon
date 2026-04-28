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
| `BgGame_Lib` | https://github.com/halheinrich/BgGame_Lib |
| `XgFilter_Razor` | https://github.com/halheinrich/XgFilter_Razor |

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
* Charter scope: types and pure data-shape translations between data types or to primitives. A translation whose result type comes from another subproject is a true cross-subproject dependency and lives in the consumer, not here. Game-mode-specific records (e.g., `SubmittedPlay`, `QuizScore`) live in BgGame_Lib.

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

### BgGame_Lib

**Purpose:** Game/play substrate — `GameState`, `IPlayAgent`, `ICubeAgent`, `IProblemSetSource`, skeletal `Referee`, `Transcript`, plus quiz-result records `SubmittedPlay` and `QuizScore`. Multi-mode scaffolding shared by scored quiz, user-vs-user, user-vs-bot, and bot-vs-bot tournament.
**Branch:** main
**Solution:** `BgGame_Lib\BgGame_Lib.slnx`
**Depends on:** BgDataTypes_Lib, BgMoveGen (planned — substrate types follow the scaffold)

Key facts:

* Initial commit ships scaffold only (Class Library + xUnit test project, mirroring BgMoveGen's layout); substrate types come in follow-up commits
* Razor-free — the "human via clicks" `IPlayAgent` implementation lives in `BgQuiz_Blazor`, not here, keeping the substrate Blazor-free for non-UI consumers (future bot-vs-bot loops, replay analytics, etc.)

### XgFilter_Razor

**Purpose:** Razor Class Library for shared filter UI — `FilterPanel.razor` and supporting types. Razor-side counterpart to `XgFilter_Lib`, paralleling `BgDiag_Razor`'s relationship with `BackgammonDiagram_Lib`.
**Branch:** main
**Solution:** `XgFilter_Razor\XgFilter_Razor.slnx`
**Depends on:** XgFilter_Lib, BgDataTypes_Lib (planned — extraction follows the scaffold)

Key facts:

* Initial commit ships scaffold only (Razor Class Library + xUnit/bUnit test project, mirroring BgDiag_Razor); `FilterPanel.razor` + supporting types extraction from `ExtractFromXgToCsv.Client` follows in the next commit
* `Microsoft.AspNetCore.Components.Web` pinned at 10.0.7 (newer than BgDiag_Razor's 10.0.5; intentional — kept on the latest)
* Eventual consumers: `ExtractFromXgToCsv.Client` (after a follow-up ExtractFromXgToCsv session removes the originals and adds a project reference) and `BgQuiz_Blazor` (Phase 1)

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
| BgGame_Lib | 🔧 In progress — scaffold only |
| XgFilter_Razor | 🔧 In progress — scaffold only |

### Next up

**BgQuiz_Blazor multi-phase build-out** — Phase 0 architectural
review complete (read-only subproject session, 2026-04-27).
Substrate location decided: new `BgGame_Lib` subproject (not
BgQuiz_Blazor-internal), so the four eventual modes (scored quiz,
user-vs-user, user-vs-bot, bot-vs-bot tournament) share
scaffolding from day one.

Concrete sessions, rank-ordered:

1. **BgGame_Lib — substrate types.** Now that the scaffold is
   registered, populate the initial type set: multi-mode play
   primitives (`GameState`, `IPlayAgent`, `ICubeAgent`,
   `CubeAction` enum, skeletal `Referee` for turn sequencing /
   end-of-game, `Transcript` of state / decision / outcome
   tuples); the problem-set abstraction `IProblemSetSource` —
   iterable of `BgDecisionData` for any quiz / replay loop,
   keeping Phase 1's source choice (server-disk now; upload,
   deployed sets, curated library later) replaceable as
   alternative implementations; and the quiz-result records
   `SubmittedPlay` (chosen `Play` + matched candidate index +
   equity loss + correctness flag) and `QuizScore`
   (running-total). Adds `BgMoveGen` and `BgDataTypes_Lib`
   ProjectReferences. `INSTRUCTIONS.md` for the subproject lands
   in this commit (or its own follow-up if too much for one
   session). Built ahead of the modes per CLAUDE.md
   "Best-practice bias". `SubmittedPlay` / `QuizScore` live here
   rather than BgDataTypes_Lib because they're game-mode-specific
   records, not shared-data-layer types.

2. **BgDiag_Razor — `BackgammonPlayEntry` component.** New
   stateful Razor component wrapping the existing
   `BackgammonDiagram` and holding a `BgMoveGen.MoveEntryState`.
   Hooks the inner diagram's click events to the state machine;
   derives the displayed `Mop` from `state.Current` after each
   click; optionally overlays `state.LegalNextClicks` as hover
   hints. Exposes `OnPlayCompleted` (fires when `IsComplete`
   flips true), `OnPlayProgress` (per legal click), and
   `UndoLast` / `UndoAll` methods/callbacks so the consumer
   wires its own buttons. The existing `BackgammonDiagram` stays
   view-only — used directly by replay viewers, bot-vs-bot,
   analytics. Adds BgMoveGen as a new BgDiag_Razor dependency;
   the implementing session updates the umbrella dependency
   graph.

3. **XgFilter_Razor — extract `FilterPanel.razor` and supporting
   types from `ExtractFromXgToCsv.Client`.** Now that the
   scaffold is registered, populate it: read the existing
   `FilterPanel.razor`, code-behind partial class, supporting
   types (enum-label extension methods like `PlayType.ToLabel()`,
   `PositionType.ToLabel()`, etc.), small DTOs / state types,
   scoped CSS from `../ExtractFromXgToCsv/ExtractFromXgToCsv.Client/`
   (cross-boundary read is allowed), and recreate the equivalents
   inside `XgFilter_Razor/`. Adds `XgFilter_Lib` and
   `BgDataTypes_Lib` ProjectReferences. Don't modify
   `ExtractFromXgToCsv.Client` — that's a separate
   `ExtractFromXgToCsv` subproject session that follows. Preserve
   existing behaviour — don't fold in the encapsulation-leak fix
   on the umbrella Deferred list.

4. **BgQuiz_Blazor — Phase 1 implementation.** Wires #1–#3 into
   problem-set selection (filter UI from #3 + a server-disk
   `IProblemSetSource` implementation against the diagram-shape
   iterator), click-to-Play (via `BackgammonPlayEntry`),
   submit-and-score, running total. Uses `BgGame_Lib`'s
   substrate (single-position game with the quiz-grader as the
   second agent) so Phase 2+ modes reuse the scaffolding without
   rewrite. Server-disk source is one of several
   `IProblemSetSource` implementations anticipated; alternatives
   (upload, deployed sets, curated library) plug in via the
   same interface.

5. **Decision identification scheme.** No stable way today to
   reference a specific decision within an `.xg`/`.xgp` file.
   Phase 2+ requires it — answer tracking with weighted
   re-recurrence on wrong answers; resume / skip / re-do with
   persistent reference. Candidate ID schemes for the
   implementing session to weigh: XGID (XG's canonical
   position+state form, already in the data; needs dice
   extension for play decisions); a logical address tuple
   (file stem, match index, game index, move number, IsCube);
   or a content hash. Affects BgDataTypes_Lib (where the ID type
   lives), ConvertXgToJson_Lib (emits the ID per decision),
   XgFilter_Lib (passes through), and Phase 2+ consumers.
   Decide and ship before Phase 2+ work begins.

**Parallelism.** Items 1, 2, and 3 are independent of each
other and can run in any order. `BackgammonPlayEntry` (item 2)
consumes only the already-shipped `BgMoveGen.MoveEntryState`;
the filter-UI library (item 3) depends on the existing
`XgFilter_Lib` + `BgDataTypes_Lib` only; the substrate (item 1)
doesn't depend on either since the "human via clicks"
`IPlayAgent` implementation lives in BgQuiz_Blazor (item 4),
keeping BgGame_Lib Razor-free. Item 4 (Phase 1) pulls in all
three. Item 5 (decision IDs) is logically pre-Phase-2+; can
run in parallel with the Phase 1 chain or after, but before
any Phase 2+ work.

**Phase 2+** (answer tracking with weighted re-recurrence on
wrong answers, the three two-agent modes) builds on items 1–5
and gets queued after Phase 1 ships and item 5 lands.

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
* BgQuiz_Blazor: clear stale BgDiag_Razor-fingering pitfall at `INSTRUCTIONS.md:137-140`. The click-handling bug it describes was fixed upstream in `BackgammonDiagram_Lib.DiagramRenderer.GetHitRegions` (coordinate-system alignment with `RenderSvg`); the pitfall text should reflect that the fix shipped in the lib, not the Razor wrapper. Single-session BgQuiz_Blazor INSTRUCTIONS.md edit.
* BgMoveGen `MoveEntryState`: revisit click-semantics contract after Phase 1 ships. The α two-click model (source-then-destination, intermediate `ClickOutcome.SourceSelected`) is a first-pass commit without real UX feedback; xmldoc in `MoveEntryState.cs` documents current behaviour. Once Phase 1 ships and BgQuiz_Blazor has been click-tested in earnest, evaluate whether refinements (e.g., destination-only with inferred source for unambiguous cases) better fit observed UX.
* XgFilter_Lib: directory iteration enumerates `*.xg` only; the parser side (`XgDecisionIterator.IterateXgDirectory`) enumerates both `*.xg` and `*.xgp` via a private `EnumerateXgFormatFiles` helper. Filter-side iteration silently drops `.xgp` (XG position-file) inputs that unfiltered parser iteration would surface. Real concern for Phase 1 quiz problem-set production — `.xgp` is a likely problem-set source. Small fix; folds naturally into the next XgFilter_Lib touch (or the same session that addresses the existing exception-swallowing entry above).

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
| **BgGame_Lib** | Game/play substrate |
| **XgFilter_Razor** | Razor wrapper for filter UI |

## Architecture — dependency graph

```
BgDataTypes_Lib (shared types)
├── ConvertXgToJson_Lib (parsing)
│   └── XgFilter_Lib (filtering)
│       ├── ExtractFromXgToCsv (app)
│       └── XgFilter_Razor (Razor wrapper for filter UI)
├── BackgammonDiagram_Lib (rendering)
│   └── BgDiag_Razor (Razor wrapper for diagram)
│       └── BgQuiz_Blazor (app)
├── BgGame_Lib (game/play substrate)
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
- BgGame_Lib will consume BgMoveGen for `Play`, `BoardState`, `MoveEntryState` once substrate types ship (currently scaffold only — csproj reference lands with the substrate types).
- XgFilter_Razor will be consumed by `ExtractFromXgToCsv.Client` once a follow-up ExtractFromXgToCsv session removes the original `FilterPanel.razor` and adds a project reference (currently `.Client` still owns the original).

## Pre-session verification

```powershell
cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git submodule foreach 'echo "$name $(git rev-parse --short HEAD) $(git log --oneline -1 origin/main)"'
```

Compare each line against `git log --oneline -1` for the umbrella. Any mismatch means stale pointers — resolve before starting work.