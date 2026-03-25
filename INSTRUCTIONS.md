# Backgammon Umbrella Project

Main repo: https://github.com/halheinrich/backgammon
Local root: `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\`
**Current umbrella commit:** `4b90fd1`

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
| `ConvertXgToJson_Lib` | https://github.com/halheinrich/ConvertXgToJson_Lib | `f25850d` |
| `XgFilter_Lib` | https://github.com/halheinrich/XgFilter_Lib | `4af20df` |
| `ExtractFromXgToCsv` | https://github.com/halheinrich/ExtractFromXgToCsv | `132a723` |
| `XgAnalytics` | https://github.com/halheinrich/XgAnalytics | `a53089f` |
| `BackgammonDiagram_Lib` | https://github.com/halheinrich/BackgammonDiagram_Lib | `fbfdf4a` |
| `BgRLEngine` | https://github.com/halheinrich/BgRLEngine | `4eb6d10` |
| `BgMoveGen` | https://github.com/halheinrich/BgMoveGen | `cecc1f8` |

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

### 1. ConvertXgToJson\_Lib

**Repo:** https://github.com/halheinrich/ConvertXgToJson_Lib
**Branch:** main
**Purpose:** Reads .xg and .xgp files; produces DecisionRow records.
**Solution:** `ConvertXgToJson_Lib\ConvertXgToJson_Lib.slnx`
**Current commit:** `f25850d`

Key files:

* DecisionRow.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/Models/DecisionRow.cs
* XgDecisionIterator.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/XgDecisionIterator.cs
* XgMatchInfo.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/XgMatchInfo.cs
* XgGameInfo.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/XgGameInfo.cs
* XgFileReader.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/XgFileReader.cs
* BackgammonConstants.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/BackgammonConstants.cs
* ConvertXgToJson\_Lib.csproj: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib/ConvertXgToJson_Lib.csproj
* Tests.csproj: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib.Tests/ConvertXgToJson_Lib.Tests.csproj
* GlobalUsings.cs: https://raw.githack.com/halheinrich/ConvertXgToJson_Lib/f25850d/ConvertXgToJson_Lib.Tests/GlobalUsings.cs

Key facts:

* `DecisionRow.Board` is `int[]` (26 elements): `board[0]` = opponent bar (never positive), `board[1–24]` = points 1–24 from player on roll's perspective, `board[25]` = player bar (never negative). Positive = player on roll; negative = opponent.
* `Board` is not exposed in CSV output.
* `ToBoard` / `FlipBoard` in XgDecisionIterator handle player perspective normalization.
* XGID encoding always normalized to bottom-player perspective. All tests pass.
* XgIteratorState early-exit mechanism in place.
* XgMatchInfo populates match-level metadata before first row.
* XgGameInfo populates game-level metadata before first row.
* XgFileReader.ReadMatchInfo — fast match header extraction without full file parse.
* XgFileReader.ReadGameHeaders — fast game-header extraction without full file parse.
* BackgammonConstants — shared constants extracted from XgFileReader.

---

### 2. XgFilter\_Lib

**Repo:** https://github.com/halheinrich/XgFilter_Lib
**Branch:** main
**Purpose:** Filtering and column projection for DecisionRow records. Used by ExtractFromXgToCsv.
**Solution:** `XgFilter_Lib\XgFilter_Lib.slnx`
**Depends on:** ConvertXgToJson\_Lib
**Current commit:** `4af20df`

Key files:

* XgFilter\_Lib.csproj: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/XgFilter_Lib.csproj
* Tests.csproj: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib.Tests/XgFilter_Lib.Tests.csproj
* GlobalUsings.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib.Tests/GlobalUsings.cs
* Enums/PositionType.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Enums/PositionType.cs
* Enums/PlayType.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Enums/PlayType.cs
* Filtering/IDecisionFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/IDecisionFilter.cs
* Filtering/IMatchFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/IMatchFilter.cs
* Filtering/DecisionFilterSet.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/DecisionFilterSet.cs
* Filtering/PlayerFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/PlayerFilter.cs
* Filtering/DecisionTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/DecisionTypeFilter.cs
* Filtering/MatchScoreFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/MatchScoreFilter.cs
* Filtering/ErrorRangeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/ErrorRangeFilter.cs
* Filtering/PositionTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/PositionTypeFilter.cs
* Filtering/PlayTypeFilter.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Filtering/PlayTypeFilter.cs
* Classification/IPositionClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Classification/IPositionClassifier.cs
* Classification/RaceClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Classification/RaceClassifier.cs
* Classification/ContactClassifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Classification/ContactClassifier.cs
* Classification/InnerBoard631Classifier.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Classification/InnerBoard631Classifier.cs
* Projection/ColumnSelector.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/Projection/ColumnSelector.cs
* FilteredDecisionIterator.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib/FilteredDecisionIterator.cs
* Tests/Helpers/DecisionRowBuilder.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib.Tests/Helpers/DecisionRowBuilder.cs
* Tests/Classification/RaceClassifierTests.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib.Tests/Classification/RaceClassifierTests.cs
* Tests/FilteredDecisionIteratorTests.cs: https://raw.githack.com/halheinrich/XgFilter_Lib/4af20df/XgFilter_Lib.Tests/FilteredDecisionIteratorTests.cs

Key facts:

* `IDecisionFilter`: single method `bool Matches(DecisionRow row)`
* `IMatchFilter`: match-level early-exit filter interface
* `DecisionFilterSet`: reworked — supports both IMatchFilter and IDecisionFilter, AND semantics
* Filters: PlayerFilter, DecisionTypeFilter (uses `IsCube`), MatchScoreFilter, ErrorRangeFilter, PositionTypeFilter
* `PositionTypeFilter` uses `row.Board` via RaceClassifier / ContactClassifier — never parses Xgid
* Contact = !Race (exhaustive and mutually exclusive for now)
* Priming, Blitz, HoldingGame classifiers deferred
* PlayTypeFilter deferred
* `ColumnSelector`: explicit column registry, no reflection; drives CSV header and row serialization
* `FilteredDecisionIterator`: reworked to support early-exit via IMatchFilter

---

### 3. ExtractFromXgToCsv

**Repo:** https://github.com/halheinrich/ExtractFromXgToCsv
**Branch:** main
**Purpose:** Blazor web app. Extracts decisions from .xg/.xgp files, applies XgFilter\_Lib filters, exports CSV.
**Solution:** `ExtractFromXgToCsv\ExtractFromXgToCsv.slnx`
**Depends on:** ConvertXgToJson\_Lib, XgFilter\_Lib
**Current commit:** `132a723`

Key files:

* ExtractFromXgToCsv.csproj: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/ExtractFromXgToCsv.csproj
* Program.cs (server): https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Program.cs
* Program.cs (client): https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv.Client/Program.cs
* Services/XgProcessingService.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Services/XgProcessingService.cs
* Services/JobStore.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Services/JobStore.cs
* Controllers/ShutdownController.cs: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Controllers/ShutdownController.cs
* Components/Pages/Home.razor: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Components/Pages/Home.razor
* Components/FilterPanel.razor: https://raw.githack.com/halheinrich/ExtractFromXgToCsv/132a723/ExtractFromXgToCsv/Components/FilterPanel.razor

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

### 5. BackgammonDiagram\_Lib

**Repo:** https://github.com/halheinrich/BackgammonDiagram_Lib
**Branch:** main
**Purpose:** Pure rendering library — returns board diagrams as SVG, PNG, PDF, PowerPoint, or Blazor RenderFragment. No user interaction, no game state.
**Solution:** `BackgammonDiagram_Lib\BackgammonDiagram_Lib.slnx`
**Depends on:** ConvertXgToJson\_Lib
**Current commit:** `fbfdf4a`

Key files:

* BackgammonDiagram\_Lib.csproj: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/BackgammonDiagram_Lib.csproj
* Models/DiagramRequest.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Models/DiagramRequest.cs
* Rendering/DiagramRenderer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Rendering/DiagramRenderer.cs
* Rendering/BoardLayout.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Rendering/BoardLayout.cs
* Rendering/ISvgRasterizer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Rendering/ISvgRasterizer.cs
* Rendering/SkiaSharpRasterizer.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Rendering/SkiaSharpRasterizer.cs
* Themes/ThemeRegistry.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Themes/ThemeRegistry.cs
* Themes/GreyscaleTheme.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib/Themes/GreyscaleTheme.cs
* Tests.csproj: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib.Tests/BackgammonDiagram_Lib.Tests.csproj
* Tests/DiagramRendererTests.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib.Tests/DiagramRendererTests.cs
* Tests/TestPaths.cs: https://raw.githack.com/halheinrich/BackgammonDiagram_Lib/fbfdf4a/BackgammonDiagram_Lib.Tests/TestPaths.cs

Key decisions:

* SVG is hand-rolled
* QuestPDF for PDF output
* OpenXml for PowerPoint output
* `IsCube` drives diagram type (checker play vs cube decision)
* Analysis panel always shown in Solution mode, never in Problem mode
* Panel position (Left/Right/Below) is caller-specified
* Spec expected to evolve during implementation

---

### 6. BgRLEngine

**Repo:** https://github.com/halheinrich/BgRLEngine
**Branch:** main
**Purpose:** Reinforcement learning engine for backgammon and variants. Trained via self-play (tabula rasa). Portfolio of specialist sub-engines coordinated by a router.
**Current commit:** `4eb6d10`

Key files:

* main.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/main.py
* engine/state.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/engine/state.py
* engine/network.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/engine/network.py
* engine/dice.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/engine/dice.py
* engine/game.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/engine/game.py
* engine/setup_generator.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/engine/setup_generator.py
* training/td_trainer.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/training/td_trainer.py
* utils/sprt.py: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/utils/sprt.py
* configs/default.yaml: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/configs/default.yaml
* configs/dmp.yaml: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/configs/dmp.yaml
* configs/gammon_avoiding.yaml: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/configs/gammon_avoiding.yaml
* configs/gammon_seeking.yaml: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/configs/gammon_seeking.yaml
* configs/money.yaml: https://raw.githack.com/halheinrich/BgRLEngine/4eb6d10/configs/money.yaml

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

---

### 7. BgMoveGen

**Repo:** https://github.com/halheinrich/BgMoveGen
**Branch:** main
**Purpose:** C# move generation library for backgammon and all variants. Used by BgRLEngine.
**Solution:** `BgMoveGen\BgMoveGen.slnx`
**Current commit:** `cecc1f8`

Key files:

* BgMoveGen.csproj: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen/BgMoveGen.csproj
* BoardState.cs: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen/BoardState.cs
* Move.cs: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen/Move.cs
* MoveGenerator.cs: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen/MoveGenerator.cs
* Play.cs: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen/Play.cs
* Tests.csproj: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen.Tests/BgMoveGen.Tests.csproj
* Tests/MoveGeneratorTests.cs: https://raw.githack.com/halheinrich/BgMoveGen/cecc1f8/BgMoveGen.Tests/MoveGeneratorTests.cs

Key facts:

* Pure move generation — no evaluation, no RL
* Applies to all backgammon variants
* BgRLEngine depends on BgMoveGen
* 3.4 μs/call, avoidance-based dedup, no HashSet
* Three public entry points: GenerateStates, EnumerateStates, NextMove iterator
* 59 tests green
* Pass returns flipped state; get_starting_position export added

---

## Current status

| Subproject | Status |
| --- | --- |
| ConvertXgToJson\_Lib | ✅ Complete — XGID encoding always normalized to bottom-player perspective. XgIteratorState early-exit mechanism in place. XgMatchInfo and XgGameInfo added. XgFileReader.ReadMatchInfo and ReadGameHeaders added. All tests pass. |
| XgFilter\_Lib | ✅ Complete — all filters, classifiers, ColumnSelector, FilteredDecisionIterator, full test suite passing; TestData path updated to shared location; IMatchFilter added; early-exit optimization in place; InnerBoard631Classifier added |
| ExtractFromXgToCsv | 🔧 In progress — polling-based progress display working end-to-end; 951,973 rows from 6,660 files in 447s; exit button, InnerBoard631 filter added |
| XgAnalytics | 🔧 In progress — player match count, NonStandardStarts, MatchScoreDistribution complete |
| BackgammonDiagram\_Lib | 🔧 In progress — BoardLayout, DiagramRenderer geometry, test scaffold; PNG rendering, greyscale theme added |
| BgRLEngine | 🔧 In progress — 88% throughput improvement via BgMoveGen NativeAOT integration |
| BgMoveGen | ✅ Complete — Pass returns flipped state; get_starting_position export added; 59 tests passing |

### In progress

* End-to-end testing and refinement (ExtractFromXgToCsv)

### Deferred

* CSV download button for Azure/browser mode
* ColumnSelector wired into UI (column projection)
* Priming, Blitz, HoldingGame classifiers
* PlayTypeFilter
* ExtractFromXgToCsv gets 0 rows after XGID fix — to be diagnosed from ExtractFromXgToCsv project
* XgAnalytics — add key files section to INSTRUCTIONS.md

### Key decisions

* FilterPanel is a separate component under `ExtractFromXgToCsv/Components/`
* All rendering is InteractiveServer — no WASM components yet
* XgFilter\_Lib is server-side only
* Board not exposed in CSV/ColumnSelector
* Contact = !Race
* FlipBoard kept for cube rows (responder perspective)
* Always quote paths with spaces in PowerShell
* FilterPanel.razor owns all filter UI state; raises `OnFiltersChanged EventCallback<DecisionFilterSet>`
* Home.razor applies DecisionFilterSet to `_rows` before CSV output and displays filtered/total row count
* WASM refactor: all .xg parsing, filtering, and CSV generation moved to client; server is thin host only
* JobStore.cs added for polling-based progress display
* raw.githack.com adopted as standard URL format for all source file fetches

---

## Two-level Claude Project structure

This project (Backgammon Umbrella) is the **coordination layer** only. Heads-down coding happens in dedicated subproject Claude Projects.

### Claude Projects

| Claude Project | Purpose |
| --- | --- |
| **Backgammon Umbrella** ← you are here | Architecture, status, decisions, commit hashes, cross-cutting work |
| **ExtractFromXgToCsv** | All coding on the Blazor app |
| **XgFilter\_Lib** | All coding on the filter/classifier library |
| **ConvertXgToJson\_Lib** | All coding on the .xg/.xgp reader library |
| **XgAnalytics** | Ad-hoc analysis tools and queries against .xg/.xgp files |
| **BackgammonDiagram\_Lib** | Rendering library — board diagrams as SVG, PNG, PDF, PowerPoint, Blazor component |
| **BgRLEngine** | RL engine for backgammon and variants — Python/PyTorch training, ONNX export |
| **BgMoveGen** | C# move generation library for backgammon and all variants |

### Workflow

1. Start a coding session → go to the relevant subproject's Claude Project
2. Finish, commit, push → return here (Umbrella)
3. Update INSTRUCTIONS.md and push to GitHub

---

## Session handoff protocol

After every GitHub commit:

1. **Submodule commit** — run `git rev-parse --short HEAD` in the submodule dir; update the short hash in the subproject header and in every raw URL for that submodule
2. **Umbrella commit** — `cd` to umbrella dir, `git add <folder>`, commit, update **Current umbrella commit** and the submodule table
3. **Key files** — add raw.githack.com URLs for any new files created this session
4. **Current status table** — update subproject status
5. **In progress / Deferred** — move items as appropriate
6. **Key decisions** — append any new decisions made this session
7. **Push** — push updated INSTRUCTIONS.md to GitHub

**Other subproject instructions** — update only when about to start a session in that project. Check URLs are current against the Umbrella's pinned commit before starting work.

---

## GitHub fetch workaround

See AGENTS.md. Use `raw.githack.com` URLs exclusively — raw.githubusercontent.com is blocked, blob URLs are flaky.
