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
* Filters: PlayerFilter, DecisionTypeFilter, MatchScoreFilter, ErrorRangeFilter, PositionTypeFilter
* Classifiers: RaceClassifier, ContactClassifier (= !Race), InnerBoard631Classifier, InnerBoard54321Classifier
* `IPositionClassifier.Matches` accepts `IReadOnlyList<int>`
* `ColumnSelector`: explicit column registry, no reflection
* `FilteredDecisionIterator`: owns XgIteratorState; supports early-exit via IMatchFilter

### ExtractFromXgToCsv

**Purpose:** Blazor web app — extracts decisions from .xg/.xgp files, applies filters, exports CSV.
**Branch:** main
**Solution:** `ExtractFromXgToCsv\ExtractFromXgToCsv.slnx`
**Depends on:** ConvertXgToJson_Lib, XgFilter_Lib

Key facts:

* WASM — all .xg parsing, filtering, CSV generation runs client-side; server is thin host
* Razor pages/components live in `ExtractFromXgToCsv.Client`
* FilterPanel.razor owns filter UI state; raises `OnFiltersChanged EventCallback<DecisionFilterSet>`
* Local mode: server processes in background, client polls status
* Azure/browser mode: file upload works, CSV download button not yet implemented

### XgAnalytics

**Purpose:** Ad-hoc analysis tools against .xg/.xgp files.
**Branch:** main

Analyses: player match count, NonStandardStarts, MatchScoreDistribution

### BackgammonDiagram_Lib

**Purpose:** Pure rendering library — board diagrams as SVG, PNG, PDF, PowerPoint. No user interaction, no game state.
**Branch:** main
**Solution:** `BackgammonDiagram_Lib\BackgammonDiagram_Lib.slnx`
**Depends on:** BgDataTypes_Lib

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
* Three entry points: GenerateStates, EnumerateStates, NextMove
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
| ExtractFromXgToCsv | 🔧 In progress — CSV download button, ColumnSelector UI deferred |
| XgAnalytics | 🔧 In progress |
| BackgammonDiagram_Lib | 🔧 In progress |
| BgDiag_Razor | 🔧 In progress |
| BgRLEngine | 🔧 In progress |
| BgQuiz_Blazor | 🔧 In progress — Milestone 1 done |

### Deferred

* CSV download button for Azure/browser mode
* ColumnSelector wired into UI
* Priming, Blitz, HoldingGame classifiers
* PlayTypeFilter
* ShouldAdvanceGame / ShouldAdvanceMatch implementations
* ExtractFromXgToCsv 0-rows bug diagnosis

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

BgMoveGen (standalone)
└── BgRLEngine (standalone, Python)

XgAnalytics (standalone)
```

## Pre-session verification

```powershell
cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git submodule foreach 'echo "$name $(git rev-parse --short HEAD) $(git log --oneline -1 origin/main)"'
```

Compare each line against `git log --oneline -1` for the umbrella. Any mismatch means stale pointers — resolve before starting work.