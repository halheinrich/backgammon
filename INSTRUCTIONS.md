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

**Purpose:** Shared-data foundation — atomic by design. Hosts the data types and primitive operations every other in-tree library rests on. No subproject dependencies and must never gain any.
**Branch:** main
**Solution:** `BgDataTypes_Lib\BgDataTypes_Lib.slnx`

Key facts:

* **Atomic by design.** BgDataTypes_Lib has no subproject dependencies, by architectural invariant — not just current state. Adding one would either create a circular reference or force the dependency on every consumer transitively. `System.Text.Json` (+ `JsonStringEnumConverter`) is the only runtime dependency.
* Data types: BgDecisionData (= PositionData + DecisionData + DescriptiveData + PlayOutcomeData), DecisionRow, PlayCandidate, AnalysisDepthEntry, CubeOwner enum.
* Move/Play/BoardState — primitive types with full instance API. `Move` and `Play` are value types; `BoardState` is mutable with `HighPointOccupied` tracking maintained inside the type via instance `ApplyMove`/`UndoMove`/`ApplyPlay`. `ApplyPlay` is the public turn-boundary primitive (applies all moves, then flips perspective internally — `Flip()` is private). Derived properties: `PipCount`, `OpponentPipCount`, `IsRace`. Factories: `Standard()`, `Nackgammon()`, `Bg960(seed?)`. Bridges: `FromMop()`, `ToMop()`.
* **Mutability exception.** All types are init-only **except `BoardState`**, which is mutable for hot-path move-generation efficiency. The type encapsulates its own state-management; external mutation of `Points` is supported but `HighPointOccupied` will desync if mutated outside the apply/undo helpers. Hot-path consumers (BgMoveGen) use the apply/undo primitives; non-hot-path consumers should treat `BoardState` as advancing only via `ApplyPlay`.
* `IDecisionFilterData` interface — implemented by both DecisionRow and BgDecisionData; enables XgFilter_Lib to filter either type.
* `DecisionRow` migrated from ConvertXgToJson_Lib — CSV methods kept for pragmatic reasons.
* `DecisionRow.Board` is `IReadOnlyList<int>` (26 elements); `MatchScore` computed from needs/Crawford/length.
* `PlayCandidate.Play` field carries the structural play (populated by ConvertXgToJson_Lib); `PlayCandidate.EquityLoss` is non-nullable `double` — `0.0` means "this candidate is itself a best play"; multiple candidates may share zero loss. `DecisionData.BestPlayIndex` is the canonical-best signal when one representative is needed.
* `CubeOwner` serializes as string; `Play` serializes via `PlayJsonConverter` (custom converter — fixed-buffer struct doesn't round-trip via default System.Text.Json).
* Charter scope: types and pure data-shape translations between data types or to primitives. A translation whose result type comes from another subproject is a true cross-subproject dependency and lives in the consumer, not here. Game-mode-specific records (e.g., `SubmittedPlay`, `QuizScore`) live in BgGame_Lib.

### ConvertXgToJson_Lib

**Purpose:** Reads .xg and .xgp files; produces DecisionRow and BgDecisionData records.
**Branch:** main
**Solution:** `ConvertXgToJson_Lib\ConvertXgToJson_Lib.slnx`
**Depends on:** BgDataTypes_Lib (data types incl. Move/Play/BoardState), BgMoveGen (MoveNotationFormatter only — algorithm)

Key facts:

* `XgDecisionIterator` yields DecisionRow; `IterateDiagramRequests` yields BgDecisionData.
* Populates `PlayCandidate.Play` from a single hoisted `XgMoveTranslator.Translate` call so the same `Play` value drives both `MoveNotation` (via `MoveNotationFormatter.Format`) and the structural `Play` field — single producer, no risk of drift.
* `EquityLoss` producer convention: emit `bestEquity - equity` unconditionally — best plays produce `0.0`, ties for best naturally share zero. Replaces the old `null = best` sentinel; matches `PlayCandidate`'s "multiple may share zero" contract.
* Board: `board[0]` = opponent bar, `board[1–24]` = points, `board[25]` = player bar. Positive = player on roll; negative = opponent.
* XGID always normalized to bottom-player perspective.
* Taker cube row board is always doubler POV — no flip.
* XgIteratorState early-exit mechanism for match/game-level filtering.
* XgFileReader.ReadMatchInfo / ReadGameHeaders — fast extraction without full parse.

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
**Depends on:** ConvertXgToJson_Lib, XgFilter_Lib, XgFilter_Razor (Client — for FilterPanel + FilterConfig), BackgammonDiagram_Lib (server-side only, for PPTX)

Key facts:

* WASM — all .xg parsing, filtering, CSV generation runs client-side; server is thin host
* Razor pages/components live in `ExtractFromXgToCsv.Client`
* `FilterPanel` UI is consumed from `XgFilter_Razor` (was in-tree until this swap); component contract unchanged — owns filter UI state, raises `OnFiltersChanged EventCallback<DecisionFilterSet>`
* `FilterConfig` (JSON DTO) is also consumed from `XgFilter_Razor`; `ProcessRequest` (host-app-specific, wraps `OutputFormat`) stays in `Client/Shared/`
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
* Play panel "Eq Loss" column renders blank for `EquityLoss == 0.0` (any best play in the equivalence class) and numeric for non-best — matches `PlayCandidate`'s "multiple may share zero" contract; preserves existing user-facing rendering after the EquityLoss-flip arc.

### BgDiag_Razor

**Purpose:** Razor Class Library hosting backgammon UI components — view-only `BackgammonDiagram` plus the stateful `BackgammonPlayEntry` for click-by-click play assembly. Keeps BackgammonDiagram_Lib free of Blazor dependencies.
**Branch:** main
**Depends on:** BackgammonDiagram_Lib, BgMoveGen (for `MoveEntryState`), BgDataTypes_Lib (explicit reference for Move/Play/BoardState)

Key facts:

* Two components, smart-vs-dumb split:
  * `BackgammonDiagram` — view-only. Click handling via transparent SVG overlay + pure Razor EventCallbacks (`OnPointClicked` 1–24, `OnBarClicked` 25, `OnCubeClicked`, `OnTrayClicked`); no JS interop. Used by replay viewers, bot-vs-bot, analytics, anything not taking human input.
  * `BackgammonPlayEntry` — stateful. Wraps `BackgammonDiagram` and holds an internal `BgMoveGen.MoveEntryState`. Public surface kept minimal: `Request`, `Options`, `AdditionalAttributes`, `OnPlayCompleted`, `UndoLast()`, `UndoAll()`. No `OnPlayProgress`, `OnUndo`, `ShowLegalHints`, or read-only state-query properties — those were considered and deferred (track in Deferred for the legal-hints overlay).
* `BackgammonPlayEntry` reset key is value-equality on `(Mop, Dice)` — swapping in a structurally different `Request` resets the internal state machine; identical request shape preserves it.
* Cube decisions are out of scope for `BackgammonPlayEntry` — `Decision.IsCube == true` rejected with `NotImplementedException`. A future cube-entry sibling component owns that path (Deferred).

### BgRLEngine

**Purpose:** Reinforcement learning engine — TD-Gammon style self-play training, ONNX export for C# inference.
**Branch:** main
**Language:** Python / PyTorch (files under `BgRLEngine/` subfolder)

Key facts:

* Portfolio-of-experts architecture with routing tree
* 303-feature board state encoding, ~145K parameter network
* Phase 2 (AlphaZero + MCTS) deferred

### BgMoveGen

**Purpose:** Move-generation algorithms over BgDataTypes_Lib's primitive types. C# library for backgammon and all variants.
**Branch:** main
**Solution:** `BgMoveGen\BgMoveGen.slnx`
**Depends on:** BgDataTypes_Lib (Move, Play, BoardState — primitive types and their instance API)

Key facts:

* 3.4 μs/call, avoidance-based dedup.
* Move-generation algorithms: `GeneratePlays`, `GenerateStates`, `EnumerateStates`, `NextMove`. Operate on `BoardState` (in BgDataTypes_Lib) via its instance `ApplyMove`/`UndoMove` primitives.
* Public turn-boundary apply: `MoveGenerator.ApplyPlay(state, play, die1, die2)` validates legality (re-enumerates via `GeneratePlays` + `DeduplicationKey` match) then delegates to `state.ApplyPlay(play)` (which applies all moves and flips perspective internally). `MoveGenerator.IsLegalPlay` exposes the validation alone for callers that want to validate without applying. Throw-before-mutate semantics — illegal play leaves state untouched.
* Move formatting: `MoveNotationFormatter.Format(Play)` — standard notation. Same shape as before; `Play` now lives in BgDataTypes_Lib but the formatter's algorithm stays here.
* `MoveEntryState` (click-driven Play assembly) lives here — algorithm-side state machine over BoardState.
* BgRLEngine consumes via NativeAOT interop (`BgBoardState` blittable layout, separate from BgDataTypes_Lib's BoardState).

### BgQuiz_Blazor

**Purpose:** Blazor quiz app for backgammon decisions. Phase 1 ships scored-quiz mode against a server-disk problem-set source; multi-mode (user-vs-user, user-vs-bot, bot-vs-bot tournament) lands in Phase 2+.
**Branch:** main
**Solution:** `BgQuiz_Blazor\BgQuiz_Blazor.slnx`
**Depends on:** BgGame_Lib (substrate), XgFilter_Razor (FilterPanel UI), XgFilter_Lib (DecisionFilterSet), BgDiag_Razor (BackgammonDiagram + BackgammonPlayEntry), BgMoveGen (Play / DeduplicationKey), BgDataTypes_Lib (BgDecisionData / PlayCandidate), ConvertXgToJson_Lib (transitively via XgFilter_Lib)

Key facts:

* Phase 1 flow: filter selection on `/`, problem stream from `ServerDiskProblemSetSource` (factory-injected `IProblemSetSource` implementation over `FilteredDecisionIterator.IterateXgDirectoryDiagrams`), click-to-Play via `BackgammonPlayEntry`, structural matching in `QuizController.SubmitPlayAsync` via `Play.DeduplicationKey()`, scoring against `PlayCandidate.EquityLoss` (`0.0 = best`).
* `QuizController` is scoped per circuit (Interactive Server render mode); state lost on reload — pre-Azure-deployment session will revisit render mode and persistence.
* Cube policy: Phase 1 is checker-plays-only. `QuizController.StartAsync` appends `DecisionTypeFilter(CheckerPlaysOnly)` to the user's filter set; `Home.razor` shows a permanent banner. Cube-only filter selection produces an empty quiz (banner sets the expectation).
* Off-list submission semantics: a user play whose `DeduplicationKey()` matches no candidate counts as a skip (excluded from history and score), not a low-scoring entry.
* `IProblemSetSource` is factory-injected (`Func<DecisionFilterSet, IProblemSetSource>`); Program.cs registers a closure that captures the configured `Quiz:ProblemSetDirectory`. Keeps the controller decoupled from the concrete source implementation; tests inject a fake.
* Routes: `/` (filter selection), `/quiz` (in-progress), `/done` (final summary, restart options).

### BgGame_Lib

**Purpose:** Game/play substrate — multi-mode scaffolding shared by scored quiz, user-vs-user, user-vs-bot, and bot-vs-bot tournament.
**Branch:** main
**Solution:** `BgGame_Lib\BgGame_Lib.slnx`
**Depends on:** BgDataTypes_Lib, BgMoveGen

Key facts:

* Substrate types in flat `BgGame_Lib` namespace: `MatchState` / `GameState` (separated rather than fused — match-level vs game-level state have distinct lifecycles), `MatchSnapshot` / `GameSnapshot` (immutable views for transcript / replay), `GameResult` + `GameResultKind`, `CubeAction` enum, skeletal `Referee` (end-of-game detection, cube-response application), `Transcript` plus a 3-subtype `TranscriptEntry` hierarchy, `IPlayAgent`, `ICubeAgent` (two-method offer / response shape), `IProblemSetSource` (re-iterable `IAsyncEnumerable`), `SubmittedPlay`, `QuizScore`.
* Razor-free — the "human via clicks" `IPlayAgent` implementation lives in `BgQuiz_Blazor`, not here, keeping the substrate Blazor-free for non-UI consumers (future bot-vs-bot loops, replay analytics, etc.).
* `GameState.ApplyPlay(play, die1, die2)` is the public unified turn-transition primitive — validates legality (delegates to `MoveGenerator.ApplyPlay`), applies the play to the board (which flips board perspective internally via `BoardState.ApplyPlay`), and re-expresses match labels and cube ownership under the new on-roll perspective. **No public swap/flip surface anywhere on the substrate.** Match-label re-expression is internal to MatchState (reachable from GameState in-assembly); cube-owner flip is inline in GameState.ApplyPlay's body. Skipping `GameState.ApplyPlay` (e.g., calling `Board.ApplyPlay` directly) leaves match labels and cube ownership unflipped — half-flipped substrate.
* `Referee` retains end-of-game detection (`IsGameOver`) and cube-response application (`ApplyCubeResponse`) only. Play application moved to `GameState.ApplyPlay` per the encapsulation principle that perspective transitions are atomic operations on the data, not external orchestration.
* Consumers: `BgQuiz_Blazor` (pending — Phase 1, queue item 1).

### XgFilter_Razor

**Purpose:** Razor Class Library for shared filter UI — `FilterPanel.razor` and supporting types. Razor-side counterpart to `XgFilter_Lib`, paralleling `BgDiag_Razor`'s relationship with `BackgammonDiagram_Lib`.
**Branch:** main
**Solution:** `XgFilter_Razor\XgFilter_Razor.slnx`
**Depends on:** XgFilter_Lib, BgDataTypes_Lib

Key facts:

* Hosts `Components/FilterPanel.razor` (filter-building UI) and `Shared/FilterConfig.cs` (JSON-serialisable filter DTO). Pure relocation from `ExtractFromXgToCsv.Client`; preserves existing behaviour including two known encapsulation leaks tracked on the umbrella Deferred list.
* `Microsoft.AspNetCore.Components.Web` pinned at 10.0.7 (newer than BgDiag_Razor's 10.0.5; intentional — kept on the latest).
* Three bUnit smoke tests for FilterPanel; the `ToLabel()` extension methods FilterPanel uses are owned by `XgFilter_Lib.Enums.EnumLabel` (with their own tests there).
* `IJSRuntime` localStorage coupling — FilterPanel persists filter state via JS interop. Consumers must provide an `IJSRuntime` registration; Blazor Server / WASM defaults handle this, non-Blazor consumers would need adapters.
* `Shared/FilterConfig.cs` placement is provisional — JSON DTO, not Razor-specific. Future-cleanup candidate to move to `XgFilter_Lib` (or `BgDataTypes_Lib`); tracked on Deferred.
* Consumers: `ExtractFromXgToCsv.Client` (current — references this lib for `FilterPanel` + `FilterConfig`; the host's server csproj picks up the dep transitively through `Client → XgFilter_Razor`, which is "hidden" rather than ideal but resolves cleanly when `FilterConfig` moves out per the Deferred entry below) and `BgQuiz_Blazor` (pending — Phase 1).

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
| BgDiag_Razor | 🔧 In progress — `BackgammonPlayEntry` shipped; cube-entry sibling deferred |
| BgRLEngine | 🔧 In progress |
| BgQuiz_Blazor | 🔧 In progress — Phase 1 (scored quiz) shipped; filter-narrowing bug deferred |
| BgGame_Lib | 🔧 In progress — substrate types in; awaiting BgQuiz_Blazor consumer (Phase 1) |
| XgFilter_Razor | 🔧 In progress — in use by ExtractFromXgToCsv; awaiting BgQuiz_Blazor consumer (Phase 1) |

### Next up

**BgQuiz_Blazor multi-phase build-out** — Phase 0 architectural
review complete (read-only subproject session, 2026-04-27).
Encapsulation arc closed in Session E. **Phase 1 (scored quiz
mode) shipped.** Substrate location decided: new `BgGame_Lib`
subproject (not BgQuiz_Blazor-internal), so the four eventual
modes (scored quiz, user-vs-user, user-vs-bot, bot-vs-bot
tournament) share scaffolding from day one.

Concrete sessions, rank-ordered:

1. **Decision identification scheme.** No stable way today to
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
   Pre-Phase-2+.

**Phase 2+** (answer tracking with weighted re-recurrence on
wrong answers, the three two-agent modes) builds on item 1 and
gets queued after item 1 lands.

### Deferred

* CSV download button for Azure/browser mode
* PPTX download for Azure/browser mode (SkiaSharp native isn't available under Blazor WASM)
* ColumnSelector wired into UI
* ExtractFromXgToCsv 0-rows bug diagnosis (regression after XGID perspective fix)
* BgDiag_Razor: verify Blazor component layout under new 16:9 aspect default; adapt or pass `AspectPreset.Natural` if needed
* BgDataTypes_Lib: reinstate 2 cube-error adapter tests (`UsesDoubleError`, `FallsBackToTakeError`) dropped from XgFilter_Lib in `d8fac0d` when the filter-test suite was consolidated onto `DecisionFilterAsserts`. The fallback logic being tested lives in `BgDataTypes_Lib.BgDecisionData.FilterError`, not in the filter, so the tests belong with the type.
* XgFilter_Lib: `FilteredDecisionIterator` exception-swallowing. Catch block discards exceptions silently; either handle something real or delete the catch. Correctness concern masquerading as style.
* XgFilter_Razor: two encapsulation leaks in `Components/FilterPanel.razor` — PositionType checkbox label renders `@pt` (bare identifier) instead of `@pt.ToLabel()` at line 116; local `DecisionTypeLabel` switch at lines 288-294 should use `value.ToLabel()`. Same pattern Session 4 fixed for PlayType. Preserved as-is during the extraction from `ExtractFromXgToCsv.Client` (pure relocation kept the diff reviewable as such); small single-session cleanup whenever the subproject next opens.
* ConvertXgToJson_Lib: dance-sentinel notation glitch. XG's no-move sentinel for a "dance" (closed-out roll) reaches the formatter and renders as `"1/1"`. Latent pre-existing bug surfaced during the BgMoveGen migration. Documented in subproject INSTRUCTIONS.md Pitfalls. Likely fix sites: `XgMoveTranslator` (filter sentinel before constructing `Play`) or upstream in `BuildMoveDiagramRequest`. Single-session ConvertXgToJson_Lib follow-up.
* BgQuiz_Blazor: clear stale BgDiag_Razor-fingering pitfall at `INSTRUCTIONS.md:137-140`. The click-handling bug it describes was fixed upstream in `BackgammonDiagram_Lib.DiagramRenderer.GetHitRegions` (coordinate-system alignment with `RenderSvg`); the pitfall text should reflect that the fix shipped in the lib, not the Razor wrapper. Single-session BgQuiz_Blazor INSTRUCTIONS.md edit.
* BgMoveGen `MoveEntryState`: rejects hit on second move — concrete bug surfaced during Phase 1's browser-verification walk. Repro: from a position where `B/20 20/16*` is legal (bar-entry on 20 then a hit on 16), `MoveEntryState.TryAddClick` rejects the second-move click despite the play being in `MoveGenerator.GeneratePlays`'s legal-set output. Specific to the click-semantics state machine, not the legal-play enumerator. Concrete repro recorded for the next BgMoveGen session; correctness fix.
* BgMoveGen `MoveEntryState`: broader click-semantics review (separate from the rejects-hit bug above, though related). The α two-click model (source-then-destination, intermediate `ClickOutcome.SourceSelected`) was a first-pass commit; Phase 1 has been click-tested in earnest now, so the precondition is met. Evaluate whether refinements (e.g., destination-only with inferred source for unambiguous cases) better fit observed UX.
* BgQuiz_Blazor: filter UI not narrowing the decision stream — every decision is presented regardless of filter state. Phase 1 shipped with this limitation; the FilterPanel's `OnFiltersChanged` event surfaces a `DecisionFilterSet` to the controller but the in-quiz consumption path doesn't honour it. Subproject-internal correctness fix; logged in BgQuiz_Blazor's INSTRUCTIONS as a next step but tracked here for umbrella visibility since it's user-facing.
* XgFilter_Lib: directory iteration enumerates `*.xg` only; the parser side (`XgDecisionIterator.IterateXgDirectory`) enumerates both `*.xg` and `*.xgp` via a private `EnumerateXgFormatFiles` helper. Filter-side iteration silently drops `.xgp` (XG position-file) inputs that unfiltered parser iteration would surface. Real concern for Phase 1 quiz problem-set production — `.xgp` is a likely problem-set source. Small fix; folds naturally into the next XgFilter_Lib touch (or the same session that addresses the existing exception-swallowing entry above).
* XgFilter_Razor: `Shared/FilterConfig.cs` is a JSON-serialisable filter DTO, not a Razor-specific type. Currently lives in the Razor library because that's where its only consumer (FilterPanel) was when extraction landed; moving it then would have stretched scope. Better long-term home: `XgFilter_Lib` (where the filter classes it mirrors live) or `BgDataTypes_Lib` (per the shared-data-layer charter). Mildly more urgent now: `ExtractFromXgToCsv`'s server csproj picks up `XgFilter_Razor` transitively through `Client → XgFilter_Razor` to reach `FilterConfig` — a hidden dependency that dissolves once `FilterConfig` moves to a non-Razor home. Future cleanup; not blocking.
* ExtractFromXgToCsv: rename `Client/Shared/FilterConfig.cs` → `Client/Shared/ProcessRequest.cs`. After the FilterPanel/FilterConfig extraction landed, the file holds only the `ProcessRequest` class (host-app-specific, wraps `OutputFormat` etc.). One-line rename + reference updates; folds naturally into the next ExtractFromXgToCsv touch.
* BgDataTypes_Lib: port four unique cases from BgMoveGen.Tests' deleted `BoardStateBridgeTests` into `BgDataTypes_Lib.Tests.BoardStateTests` — Bg960 Mop round-trip, mid-game hand-built position, bar-as-highest, all-zero pseudoboard. Tests followed BoardState to its new home; coverage gap until porting lands.
* Heads-up for downstream JSON consumers: post-EquityLoss-flip, `BgDecisionData` JSON output emits `"EquityLoss": 0` for best plays (was `"EquityLoss": null`). No code change needed under the new `double` (non-nullable) shape; new producers and tooling absorb cleanly. Pre-flip JSON archives — if any exist with `"EquityLoss": null` for best — won't deserialise under the new convention without a custom converter or migration. Concrete instance observed: gitignored fixture files at `BackgammonDiagram_Lib/TestData/DiagramRequest/*.json`. Resolution there is to regenerate fixtures from a post-arc ConvertXgToJson_Lib run; for any other archive, deal with it when it surfaces.
* BgDiag_Razor: cube-entry sibling component to `BackgammonPlayEntry`. The current play-entry component rejects `Decision.IsCube == true` with `NotImplementedException`. A future cube-entry component owns that path — collects double / take / pass / beaver / raccoon clicks (via dice-area / cube-area or a separate decision panel), exposes `OnCubeActionCompleted(CubeAction)`. Needed when Phase 1 extends to cube decisions or whenever a multi-mode flow spans both checker and cube agents.
* BgDiag_Razor: optional `LegalNextClicks` hover-hint overlay on `BackgammonPlayEntry`. The original component design included a `ShowLegalHints` parameter that would render a translucent highlight over each point in `state.LegalNextClicks`; the implementing session minimised the public surface and dropped the parameter. Easy to re-add when consumer demand surfaces. Adds the parameter, an overlay layer in the markup, and tests for the hint visibility toggle.

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
BgDataTypes_Lib (atomic — no subproject deps; shared data + primitive ops)
├── ConvertXgToJson_Lib (parsing)
│   └── XgFilter_Lib (filtering)
│       ├── XgFilter_Razor (Razor wrapper for filter UI)
│       │   └── ExtractFromXgToCsv.Client (app)
│       └── ExtractFromXgToCsv (app — also consumes XgFilter_Lib directly)
├── BackgammonDiagram_Lib (rendering)
│   └── BgDiag_Razor (Razor wrapper for diagram + click-driven play entry)
│       └── BgQuiz_Blazor (app)
├── BgMoveGen (move-generation algorithms over BgDataTypes_Lib's Move/Play/BoardState)
│   └── BgRLEngine (standalone, Python; via NativeAOT)
├── BgGame_Lib (game/play substrate)
├── BgPositionRouter (planned — position routing)
└── BgInference (planned — ONNX inference)

XgAnalytics (standalone)
```

`BgDataTypes_Lib` is the atomic foundation — by design, never gains a subproject dependency. Hosts Move/Play/BoardState (with full instance API: factories, bridges, apply/undo, ApplyPlay, derived PipCount/IsRace) plus the BgDecisionData / DecisionRow / PlayCandidate type set. All other in-tree libraries depend on it directly or transitively.

Cross-edges not shown in the tree:

- ExtractFromXgToCsv also consumes BackgammonDiagram_Lib server-side for PPTX output.
- BackgammonDiagram_Lib's test project references ConvertXgToJson_Lib for fixture-driven visual tests.
- ConvertXgToJson_Lib consumes BgMoveGen for move-notation formatting (`MoveNotationFormatter.Format(Play)`).
- BgGame_Lib consumes BgMoveGen for `MoveGenerator` (algorithms) — substrate types come from BgDataTypes_Lib.
- BgDiag_Razor consumes BgMoveGen for `MoveEntryState` (used inside `BackgammonPlayEntry`); also explicit BgDataTypes_Lib reference for `Move`/`Play`/`BoardState`.
- ExtractFromXgToCsv's server project picks up XgFilter_Razor transitively through `Client → XgFilter_Razor` (it actually needs `FilterConfig` for HTTP API contract). "Hidden" rather than explicit; resolves cleanly when `FilterConfig` moves out of XgFilter_Razor per the Deferred entry.

## Pre-session verification

```powershell
cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git submodule foreach 'echo "$name $(git rev-parse --short HEAD) $(git log --oneline -1 origin/main)"'
```

Compare each line against `git log --oneline -1` for the umbrella. Any mismatch means stale pointers — resolve before starting work.