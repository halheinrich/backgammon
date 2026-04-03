# Backgammon Umbrella Project

Main repo: https://github.com/halheinrich/backgammon
Local root: `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\`
**Current umbrella commit:** `2c9dfbd`

## Stack (all subprojects)

C# / .NET 10 / Visual Studio 2026 / Windows
Python / PyTorch (BgRLEngine) — also in Visual Studio 2026

## Git submodules

The umbrella repo tracks each subproject as a git submodule. After committing in any submodule, always update the umbrella:
```
cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git add <SubmoduleFolderName>
git commit -m "Update <SubmoduleName> submodule to <short-hash> (<reason>)"
```

Current submodule pinned commits:

| Submodule folder | Repo | Pinned commit |
| --- | --- | --- |
| `ConvertXgToJson_Lib` | https://github.com/halheinrich/ConvertXgToJson_Lib | `d5c3ed6` |
| `XgFilter_Lib` | https://github.com/halheinrich/XgFilter_Lib | `acaabf5` |
| `ExtractFromXgToCsv` | https://github.com/halheinrich/ExtractFromXgToCsv | `132a723` |
| `XgAnalytics` | https://github.com/halheinrich/XgAnalytics | `a53089f` |
| `BackgammonDiagram_Lib` | https://github.com/halheinrich/BackgammonDiagram_Lib | `23fd569` |
| `BgDiag_Razor` | https://github.com/halheinrich/BgDiag_Razor | `cdb69aa` |
| `BgRLEngine` | https://github.com/halheinrich/BgRLEngine | `e14caa1` |
| `BgMoveGen` | https://github.com/halheinrich/BgMoveGen | `e8e8d06` |
| `BgQuiz_Blazor` | https://github.com/halheinrich/BgQuiz_Blazor | `019c8de` |

## Naming convention

Always specify which Program.cs is being modified:

* Server: `ExtractFromXgToCsv\ExtractFromXgToCsv\Program.cs`
* Client: `ExtractFromXgToCsv\ExtractFromXgToCsv.Client\Program.cs`

## Shared infrastructure

### TestData

* Location: `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\TestData`
* `BothWays/` subfolder: `ThisWay.xg` and `ThatWay.xg` — same match, perspectives reversed
* Shared across all projects — do NOT put TestData inside individual project directories

---

## Subprojects

### 1. ConvertXgToJson_Lib

**Repo:** https://github.com/halheinrich/ConvertXgToJson_Lib
**Branch:** main
**Purpose:** Reads .xg and .xgp files; produces DecisionRow records.
**Solution:** `ConvertXgToJson_Lib\ConvertXgToJson_Lib.slnx`
**Current commit:** `d5c3ed6`

Key files:

* DecisionRow.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/Models/DecisionRow.cs
* XgDecisionIterator.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/XgDecisionIterator.cs
* XgMatchInfo.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/XgMatchInfo.cs
* XgGameInfo.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/XgGameInfo.cs
* XgFileReader.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/XgFileReader.cs
* BackgammonConstants.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/BackgammonConstants.cs
* ConvertXgToJson_Lib.csproj: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib/ConvertXgToJson_Lib.csproj
* Tests.csproj: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib.Tests/ConvertXgToJson_Lib.Tests.csproj
* GlobalUsings.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib@d5c3ed6/ConvertXgToJson_Lib.Tests/GlobalUsings.cs

Key facts:

* `DecisionRow.Board` is `int[]` (26 elements): `board[0]` = opponent bar (never positive), `board[1–24]` = points 1–24 from player on roll's perspective, `board[25]` = player bar (never negative). Positive = player on roll; negative = opponent.
* `Board` is not exposed in CSV output.
* `ToBoard` / `FlipPosition` in XgDecisionIterator handle perspective normalization.
* Taker cube row board is always doubler POV — no flip. `FlipBoard` removed as dead code.
* `MatchScoreFor(int activePlayer)` — returns match score from active player's perspective.
* XGID encoding always normalized to bottom-player perspective. All tests pass.
* XgIteratorState early-exit mechanism in place.
* XgMatchInfo populates match-level metadata before first row.
* XgGameInfo populates game-level metadata before first row.
* XgFileReader.ReadMatchInfo — fast match header extraction without full file parse.
* XgFileReader.ReadGameHeaders — fast game-header extraction without full file parse.
* BackgammonConstants — shared constants extracted from XgFileReader.

---

### 2. XgFilter_Lib

**Repo:** https://github.com/halheinrich/XgFilter_Lib
**Branch:** main
**Purpose:** Filtering and column projection for DecisionRow records. Used by ExtractFromXgToCsv.
**Solution:** `XgFilter_Lib\XgFilter_Lib.slnx`
**Depends on:** ConvertXgToJson_Lib
**Current commit:** `acaabf5`

Key files:

* XgFilter_Lib.csproj: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/XgFilter_Lib.csproj
* Tests.csproj: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib.Tests/XgFilter_Lib.Tests.csproj
* GlobalUsings.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib.Tests/GlobalUsings.cs
* Enums/PositionType.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Enums/PositionType.cs
* Enums/PlayType.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Enums/PlayType.cs
* Filtering/IDecisionFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/IDecisionFilter.cs
* Filtering/IMatchFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/IMatchFilter.cs
* Filtering/DecisionFilterSet.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/DecisionFilterSet.cs
* Filtering/PlayerFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/PlayerFilter.cs
* Filtering/DecisionTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/DecisionTypeFilter.cs
* Filtering/MatchScoreFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/MatchScoreFilter.cs
* Filtering/ErrorRangeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/ErrorRangeFilter.cs
* Filtering/PositionTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/PositionTypeFilter.cs
* Filtering/PlayTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Filtering/PlayTypeFilter.cs
* Classification/IPositionClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Classification/IPositionClassifier.cs
* Classification/RaceClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Classification/RaceClassifier.cs
* Classification/ContactClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Classification/ContactClassifier.cs
* Classification/InnerBoard631Classifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Classification/InnerBoard631Classifier.cs
* Projection/ColumnSelector.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/Projection/ColumnSelector.cs
* FilteredDecisionIterator.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib/FilteredDecisionIterator.cs
* Tests/Helpers/DecisionRowBuilder.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib.Tests/Helpers/DecisionRowBuilder.cs
* Tests/Classification/RaceClassifierTests.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib.Tests/Classification/RaceClassifierTests.cs
* Tests/FilteredDecisionIteratorTests.cs: https://raw.githack.com/halheinrich/XgFilter_Lib@acaabf5/XgFilter_Lib.Tests/FilteredDecisionIteratorTests.cs

Key facts:

* `IDecisionFilter`: single method `bool Matches(DecisionRow row)`; default methods `ShouldAdvanceGame`, `ShouldAdvanceMatch` (both default false)
* `IMatchFilter`: `bool ShouldSkipMatch(XgMatchInfo match)`; `bool ShouldSkipGame(XgGameInfo game)`
* `DecisionFilterSet`: ordered list of IDecisionFilter, AND semantics; supports both IMatchFilter and IDecisionFilter
* Filters: PlayerFilter, DecisionTypeFilter, MatchScoreFilter, ErrorRangeFilter, PositionTypeFilter
* `PositionTypeFilter` uses `row.Board` via RaceClassifier / ContactClassifier — never parses Xgid
* Contact = !Race (exhaustive and mutually exclusive for now)
* Priming, Blitz, HoldingGame classifiers deferred
* PlayTypeFilter deferred
* `ColumnSelector`: explicit column registry, no reflection; drives CSV header and row serialization
* `FilteredDecisionIterator`: owns XgIteratorState; supports early-exit via IMatchFilter

---

### 3. ExtractFromXgToCsv

**Repo:** https://github.com/halheinrich/ExtractFromXgToCsv
**Branch:** main
**Purpose:** Blazor web app. Extracts decisions from .xg/.xgp files, applies XgFilter_Lib filters, exports CSV.
**Solution:** `ExtractFromXgToCsv\ExtractFromXgToCsv.slnx`
**Depends on:** ConvertXgToJson_Lib, XgFilter_Lib
**Current commit:** `132a723`

Key files:

* ExtractFromXgToCsv.csproj: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/ExtractFromXgToCsv.csproj
* Program.cs (server): https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Program.cs
* Program.cs (client): https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv.Client/Program.cs
* Services/XgProcessingService.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Services/XgProcessingService.cs
* Services/JobStore.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Services/JobStore.cs
* Controllers/ShutdownController.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Controllers/ShutdownController.cs
* Components/Pages/Home.razor: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Components/Pages/Home.razor
* Components/FilterPanel.razor: https://raw.githack.com/halheinrich/ExtractFromXgToCsv@132a723/ExtractFromXgToCsv/Components/FilterPanel.razor

Key facts:

* WASM refactor complete — all .xg parsing, filtering, and CSV generation runs in the client
* Server is a thin host only
* Home.razor: owns file input, raw row list, output path/filename; localStorage persists input/output folder and CSV filename
* FilterPanel.razor: separate component under `Components/`; owns filter state; raises `OnFiltersChanged` EventCallback returning `DecisionFilterSet`; Home.razor applies the filter set to rows before building CSV
* Local mode: scans folder for .xg/.xgp, writes CSV to disk
* Azure/browser mode: file upload, CSV download (download button not yet implemented)

---

### 4. XgAnalytics

**Repo:** https://github.com/halheinrich/XgAnalytics
**Branch:** main
**Purpose:** Ad-hoc analysis tools and queries against .xg/.xgp files.
**Current commit:** `a53089f`

Current analyses:

* Player match count — scans .xg/.xgp files, outputs CSV of players and match counts
* NonStandardStarts — identifies matches that don't start from the standard opening position
* MatchScoreDistribution — distribution of match scores across files

---

### 5. BackgammonDiagram_Lib

**Repo:** https://github.com/halheinrich/BackgammonDiagram_Lib
**Branch:** main
**Purpose:** Pure rendering library — returns board diagrams as SVG, PNG, PDF, PowerPoint, or Blazor RenderFragment. No user interaction, no game state.
**Solution:** `BackgammonDiagram_Lib\BackgammonDiagram_Lib.slnx`
**Depends on:** ConvertXgToJson_Lib
**Current commit:** `23fd569`

Key files:

* BackgammonDiagram_Lib.csproj: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/BackgammonDiagram_Lib.csproj
* Models/Enums.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/Enums.cs
* Models/DiagramSize.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/DiagramSize.cs
* Models/DiagramRequest.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/DiagramRequest.cs
* Models/DiagramOptions.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/DiagramOptions.cs
* Models/DiagramRequestExtensions.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/DiagramRequestExtensions.cs
* Models/PlayCandidate.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/PlayCandidate.cs
* Models/AnalysisDepthEntry.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/AnalysisDepthEntry.cs
* Models/BoardHitRegions.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Models/BoardHitRegions.cs
* Themes/ITheme.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Themes/ITheme.cs
* Themes/DefaultTheme.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Themes/DefaultTheme.cs
* Themes/GreyscaleTheme.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Themes/GreyscaleTheme.cs
* Themes/ThemeRegistry.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Themes/ThemeRegistry.cs
* Rendering/BoardLayout.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/BoardLayout.cs
* Rendering/ISvgRasterizer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/ISvgRasterizer.cs
* Rendering/SkiaSharpRasterizer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/SkiaSharpRasterizer.cs
* Rendering/DiagramRenderer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/DiagramRenderer.cs
* Rendering/PptxBuilder.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/PptxBuilder.cs
* Rendering/PdfBuilder.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib/Rendering/PdfBuilder.cs
* Tests/BoardLayoutTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/24a9a16/BackgammonDiagram_Lib.Tests/BoardLayoutTests.cs
* Tests/SvgStructureTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/24a9a16/BackgammonDiagram_Lib.Tests/SvgStructureTests.cs
* Tests/VisualOutputTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/24a9a16/BackgammonDiagram_Lib.Tests/VisualOutputTests.cs
* Tests/PptxConformanceTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib.Tests/PptxConformanceTests.cs
* Tests/HitRegionsTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib.Tests/HitRegionsTests.cs
* Tests/TestPaths.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib@24a9a16/BackgammonDiagram_Lib.Tests/TestPaths.cs
* CodeReview.md: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/23fd569/CodeReview.md

Key decisions:

* SVG is hand-rolled
* QuestPDF for PDF output (Community license)
* OpenXml for PowerPoint output
* `IsCube` drives diagram type (checker play vs cube decision)
* Analysis panel always shown in Solution mode, never in Problem mode
* Panel position (Left/Right) is caller-specified
* PPTX repair prompt fixed: `sldLayoutId` must be ≥ 2147483648 per OOXML spec — root cause was hardcoded value of 2199
* PPTX post-processing fixes five OpenXml SDK quirks (Content_Types, relative .rels paths, namespace hoisting, sequential rIds, XML declarations); six regression tests in `PptxConformanceTests.cs`
* PDF: each DiagramRequest → one page (PNG embedded via QuestPDF `FitArea()`); widescreen landscape 13.33" × 7.5" matching PPTX; optional title in page header
* PDF: `PdfBuilder` is internal static — same pattern as `PptxBuilder`
* `HomeBoardOnRight` mirror is purely geometric reflection about playing area centre in `ColumnCentreX`; `effectivePt` in `AppendPoints` reverted — triangle direction follows raw `pt`
* Hit regions: `GetHitRegions(DiagramRequest, DiagramOptions)` — `DiagramRequest` required so `HomeBoardOnRight` feeds into `ColumnCentreX`; 11 tests (10 existing + 1 orientation test)
* `DiagramOrientation` enum removed — replaced by `bool HomeBoardOnRight { get; init; } = true` on `DiagramRequest`; breaking change for any callers referencing `Orientation`/`DiagramOrientation`
* `DiagramRequest` converted from `record` to immutable `class` with inner `Builder`; validation enforced at `Build()` — Mop length 26, Dice length 2, IsCube/Dice consistency, CubeSize power-of-2 from 1–4096
* `DiagramOptions.ThemeName` string removed — replaced by `ITheme Theme` direct reference defaulting to `ThemeRegistry.Default`
* `ThemeRegistry` simplified to static instances `Default` and `Greyscale`; `Resolve(string)` removed
* Spec expected to evolve during implementation

---

### 6. BgDiag_Razor

**Repo:** https://github.com/halheinrich/BgDiag_Razor
**Branch:** main
**Purpose:** Thin Razor Class Library wrapper around BackgammonDiagram_Lib. Exposes a `BackgammonDiagram.razor` component that calls `DiagramRenderer.RenderSvg()` and injects the result as `MarkupString`. Kept separate so the core library has no Blazor dependency.
**Depends on:** BackgammonDiagram_Lib
**Current commit:** `cdb69aa`

Key files:

* BackgammonDiagram.razor: https://raw.githack.com/halheinrich/BgDiag_Razor@cdb69aa/BgDiag_Razor/Components/BackgammonDiagram.razor
* BackgammonDiagram.razor.cs: https://raw.githack.com/halheinrich/BgDiag_Razor@cdb69aa/BgDiag_Razor/Components/BackgammonDiagram.razor.cs
* BackgammonDiagramTests.cs: https://raw.githack.com/halheinrich/BgDiag_Razor@cdb69aa/BgDiag_Razor.Tests/BackgammonDiagramTests.cs

Key facts:

* Razor Class Library (`Microsoft.NET.Sdk.Razor`)
* Parameters: `DiagramRequest? Request`, `DiagramOptions Options`
* Transparent SVG click overlay rendered on top of diagram SVG
* EventCallbacks wired: `OnPointClicked(1–24)`, `OnBarClicked(25)`, `OnCubeClicked`, `OnTrayClicked`
* Consumes `DiagramRenderer.GetHitRegions(DiagramRequest, DiagramOptions)` from BackgammonDiagram_Lib
* Pure Razor event model — no JS interop needed
* bunit tests cover overlay rendering and all click callbacks

---

### 7. BgRLEngine

**Repo:** https://github.com/halheinrich/BgRLEngine
**Branch:** main
**Purpose:** Reinforcement learning engine for backgammon and variants. Trained via self-play (tabula rasa). Portfolio of specialist sub-engines coordinated by a router.
**Current commit:** `e14caa1`

Key facts:

* Python / PyTorch for training; ONNX export for future C# inference
* IDE: Visual Studio 2026 (Python workload)
* Hardware: i9-12900K, 64GB RAM, RTX 3060 12GB
* Phase 1: TD-Gammon style NN via TD(λ) self-play
* Phase 2: AlphaZero style NN + MCTS (deferred)
* One model per variant; variant encoded in state representation
* Sub-engine routing tree: checker play (Race / General) and cube decision (Race / Money / Match play pre-Crawford)
* Match play cube classified by gammon dynamics: Seeking / Averse / Indifferent / Balanced
* Variant skill measurement via frozen checkpoint curriculum (Level 0 = random, Level N beats Level N-1 at ~75%)
* All five open design questions resolved: SPRT spec, plateau detection, state encoding, gammon classification, seam handling
* 303-feature board state encoding
* ~145K parameter network, smoke test passes on CUDA
* Uncommitted planning work deferred

---

### 8. BgMoveGen

**Repo:** https://github.com/halheinrich/BgMoveGen
**Branch:** main
**Purpose:** C# move generation library for backgammon and all variants. Used by BgRLEngine.
**Solution:** `BgMoveGen\BgMoveGen.slnx`
**Current commit:** `e8e8d06`

Key files:

* BgMoveGen.csproj: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen/BgMoveGen.csproj
* BoardState.cs: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen/BoardState.cs
* Move.cs: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen/Move.cs
* MoveGenerator.cs: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen/MoveGenerator.cs
* Play.cs: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen/Play.cs
* Tests.csproj: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen.Tests/BgMoveGen.Tests.csproj
* Tests/MoveGeneratorTests.cs: https://raw.githack.com/halheinrich/BgMoveGen@e8e8d06/BgMoveGen.Tests/MoveGeneratorTests.cs

Key facts:

* Pure move generation — no evaluation, no RL
* Applies to all backgammon variants
* BgRLEngine depends on BgMoveGen
* 3.4 μs/call, avoidance-based dedup, no HashSet
* Three public entry points: GenerateStates, EnumerateStates, NextMove iterator
* 50 tests green

---

### 9. BgQuiz_Blazor

**Repo:** https://github.com/halheinrich/BgQuiz_Blazor
**Branch:** main
**Purpose:** Blazor app for quizzing backgammon decisions.
**Solution:** `BgQuiz_Blazor\BgQuiz_Blazor.slnx`
**Depends on:** BgDiag_Razor
**Current commit:** `019c8de`

Key files:

* BgQuiz_Blazor.csproj: https://raw.githack.com/halheinrich/BgQuiz_Blazor@019c8de/BgQuiz_Blazor/BgQuiz_Blazor.csproj
* Program.cs: https://raw.githack.com/halheinrich/BgQuiz_Blazor@019c8de/BgQuiz_Blazor/Program.cs
* Components/Pages/Home.razor: https://raw.githack.com/halheinrich/BgQuiz_Blazor@019c8de/BgQuiz_Blazor/Components/Pages/Home.razor
* Components/Pages/Home.razor.cs: https://raw.githack.com/halheinrich/BgQuiz_Blazor@019c8de/BgQuiz_Blazor/Components/Pages/Home.razor.cs

Key facts:

* Builds clean
* Milestone 1 functional: `BackgammonDiagram` renders with hardcoded position, orientation toggle, click reporting
* `CreateOpeningPosition()` is still a TODO — needs real Mop array and actual dice
* `Dice = [1, 1]` stub in place to satisfy `Validate()`

---

## Current status

| Subproject | Status |
| --- | --- |
| ConvertXgToJson_Lib | ✅ Complete — MatchScoreFor replaces MatchScore; taker row board always doubler POV; FlipBoard removed; all tests pass |
| XgFilter_Lib | ✅ Complete — all filters, classifiers, ColumnSelector, FilteredDecisionIterator with early-exit; all tests pass |
| ExtractFromXgToCsv | 🔧 In progress — polling-based progress display working end-to-end; 951,973 rows from 6,660 files in 447s |
| XgAnalytics | 🔧 In progress — player match count, NonStandardStarts, MatchScoreDistribution complete |
| BackgammonDiagram_Lib | ✅ Complete — full code review done; all findings resolved; CodeReview.md complete |
| BgDiag_Razor | 🔧 In progress — DiagramRequest Builder pattern adopted in tests; all bunit tests passing |
| BgRLEngine | 🔧 In progress — DMP long run: level 4 in 100K games; uncommitted planning work deferred |
| BgMoveGen | ✅ Complete — move generation library, all tests passing |
| BgQuiz_Blazor | 🔧 In progress — builds clean; Milestone 1 functional; CreateOpeningPosition() still TODO |

### Deferred

* CSV download button for Azure/browser mode
* ColumnSelector wired into UI (column projection)
* Priming, Blitz, HoldingGame classifiers
* PlayTypeFilter
* ExtractFromXgToCsv gets 0 rows after XGID fix — to be diagnosed from ExtractFromXgToCsv project
* ShouldAdvanceGame / ShouldAdvanceMatch implementations in XgFilter_Lib (MoveNumberFilter will be first consumer)

### Key decisions

* FilterPanel is a separate component under `ExtractFromXgToCsv/Components/`
* XgFilter_Lib is client-side (WASM refactor complete)
* Board not exposed in CSV/ColumnSelector
* Contact = !Race
* Taker cube row board is always doubler POV — no flip; FlipBoard removed
* `MatchScoreFor(int activePlayer)` replaces `MatchScore` property
* Always quote paths with spaces in PowerShell
* FilterPanel.razor owns all filter UI state; raises `OnFiltersChanged EventCallback<DecisionFilterSet>`
* Home.razor applies DecisionFilterSet to `_rows` before CSV output and displays filtered/total row count
* WASM refactor: all .xg parsing, filtering, and CSV generation moved to client; server is thin host only
* JobStore.cs added for polling-based progress display
* raw.githack.com used for all source file URLs (raw.githack.com rate-limits; raw.githubusercontent.com is DNS-blocked)
* BgDiag_Razor is a separate Razor Class Library — keeps BackgammonDiagram_Lib free of Blazor dependencies
* BgDiag_Razor click handling uses transparent SVG overlay + pure Razor EventCallbacks — no JS interop
* `DiagramOrientation` enum removed — replaced by `bool HomeBoardOnRight { get; init; } = true` on `DiagramRequest`
* `HomeBoardOnRight` mirror is purely geometric reflection about playing area centre in `ColumnCentreX`
* `GetHitRegions` signature changed to `(DiagramRequest, DiagramOptions)` — `DiagramRequest` required for correct orientation mapping
* `DiagramRequest` converted from `record` to immutable `class` with inner `Builder`; validation at `Build()`
* `DiagramOptions.ThemeName` string removed — replaced by `ITheme Theme` direct reference
* `ThemeRegistry` simplified to static instances `Default` and `Greyscale`; `Resolve(string)` removed
* `Builder.CubeOwner` defaults to `CubeOwner.Centered` — enum zero value (`OnRoll`) was wrong default; fix required explicit initializer
* `F()` locale-safe via `InvariantCulture` throughout DiagramRenderer
* `AppendCheckers` passes `request.HomeBoardOnRight` to `ColumnCentreX` — checker placement bug fixed
* `EnsureLicense` removed from `PdfBuilder`; `IsPdfSupported()` added to `DiagramRenderer`; callers set QuestPDF license themselves
* `Mop`, `Dice`, `Plays`, `AnalysisDepths` defensively copied, exposed as `IReadOnlyList<T>`
* `DiagramOptions` converted to `record`
* `BoardHitRegions.Points` → `IReadOnlyDictionary`
* Test files split: `BoardLayoutTests`, `SvgStructureTests`, `VisualOutputTests`; visual tests tagged `[Trait("Category", "Visual")]`
---

## Two-level Claude Project structure

This project (Backgammon Umbrella) is the **coordination layer** only. Heads-down coding happens in dedicated subproject Claude Projects.

### Claude Projects

| Claude Project | Purpose |
| --- | --- |
| **Backgammon Umbrella** ← you are here | Architecture, status, decisions, commit hashes, cross-cutting work, generating updated instruction docs |
| **ExtractFromXgToCsv** | All coding on the Blazor app |
| **XgFilter_Lib** | All coding on the filter/classifier library |
| **ConvertXgToJson_Lib** | All coding on the .xg/.xgp reader library |
| **XgAnalytics** | Ad-hoc analysis tools and queries against .xg/.xgp files |
| **BackgammonDiagram_Lib** | Rendering library — board diagrams as SVG, PNG, PDF, PowerPoint, Blazor component |
| **BgDiag_Razor** | Razor Class Library wrapper for BackgammonDiagram_Lib |
| **BgRLEngine** | RL engine for backgammon and variants — Python/PyTorch training, ONNX export |
| **BgMoveGen** | C# move generation library for backgammon and all variants |
| **BgQuiz_Blazor** | Blazor quiz app for backgammon decisions |

### Workflow

1. Start a coding session → go to the relevant subproject's Claude Project
2. Finish, commit, push → return here (Umbrella)
3. Update this instructions doc → download updated MD
4. Re-paste updated MD into Umbrella project instructions

---

## GitHub fetch workaround

`raw.githubusercontent.com` is DNS-blocked in Claude's container. `raw.githack.com` rate-limits under repeated fetches. Use `raw.githack.com` exclusively for all source file fetches.

URL format: `https://raw.githack.com/halheinrich/{repo}@{hash}/{path}`

**Standard workaround — always follow this pattern:**

1. Ask Claude: *"Give me the URLs I need to fetch"*
2. Claude lists the jsDelivr URLs
3. Paste those URLs back into the chat as a user message
4. Claude calls `web_fetch` on each URL

This applies in all subproject Claude Projects as well.

---

## Session handoff protocol

After every GitHub commit:

1. **Submodule commit** — run `git rev-parse --short HEAD` in the submodule dir; update the short hash in the subproject header and in every URL for that submodule
2. **Umbrella commit** — `cd` to umbrella dir, `git add <folder>`, commit, update **Current umbrella commit** and the submodule table
3. **Key files** — add jsDelivr URLs for any new files created this session
4. **Current status table** — update subproject status
5. **In progress / Deferred** — move items as appropriate
6. **Key decisions** — append any new decisions made this session
7. **Affected subproject instructions** — regenerate and re-paste into that subproject's Claude Project

**Other subproject instructions** — update only when about to start a session in that project. Check URLs are current against the Umbrella's pinned commit before starting work.