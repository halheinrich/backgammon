# Backgammon Tools — Project Vision

## Mission

Build an open, engine-agnostic ecosystem of backgammon tools in C# / .NET that
serves three audiences simultaneously:

- **Personal** — a toolkit for analysing and improving one's own game
- **Community** — a platform tournament and club players can use to study
  their matches and compete
- **Developers** — well-structured, well-tested open-source libraries that
  others can build on

All projects live on GitHub and are designed to be forked, extended, and
composed.

## Guiding Principles

- **Engine-agnostic by design.** XG (eXtreme Gammon) is the first and primary
  data source, but abstractions should allow other engines and formats to be
  added without breaking consumers. The RL engine is designed around a
  variant-agnostic game interface so that standard backgammon, Nackgammon,
  Hypergammon, Longgammon, and future variants are all first-class citizens.
- **Library-first.** Every project exposes a clean public API usable as a
  dependency, not just a CLI or UI. UIs and pipelines are built on top of
  libraries, not the other way around.
- **Test-driven.** Binary format parsing, XGID encoding, decision iteration,
  rendering, move generation, and CSV/JSON output all have unit tests. New
  projects inherit this culture.
- **C# / .NET throughout.** Consistency across the ecosystem lowers the
  barrier for contributors and makes shared utilities (models, helpers, test
  infrastructure) straightforward to reuse. The RL engine stack is Python /
  PyTorch for training; ONNX export provides interop with the C# runtime.
- **Composable over monolithic.** Small, focused repos with clear
  responsibilities, wired together by consumers rather than bundled into one
  large project.

## Projects

Current subprojects, grouped by role:

### Foundation

- **BgDataTypes_Lib** — Shared type layer. `PositionData`, `DecisionData`,
  `DescriptiveData`, `BgDecisionData`, `DecisionRow`. The foundation everything
  else depends on.
- **BgMoveGen** — C# move generation library for backgammon and all variants.

### Parsing & Filtering

- **ConvertXgToJson_Lib** — Reads `.xg` and `.xgp` files; produces
  `DecisionRow` and `BgDecisionData` records. Binary parsing of XG's
  multi-stream ZLib format and XGID encoding.
- **XgFilter_Lib** — Filtering and classification for decision records. Works
  on any `IDecisionFilterData`.

### Rendering

- **BackgammonDiagram_Lib** — Pure rendering library. Produces backgammon
  diagrams as SVG, PNG, PDF, or PowerPoint from a `DiagramRequest`.
- **BgDiag_Razor** — Thin Razor Class Library wrapper exposing the diagram
  component for Blazor consumers.

### Applications

- **ExtractFromXgToCsv** — Blazor web app. Extracts decisions from `.xg` /
  `.xgp` files, applies filters, exports CSV.
- **XgAnalytics** — Ad-hoc analysis tools against XG files (player match
  count, match score distributions, non-standard starts).
- **BgQuiz_Blazor** — Player-facing study tool. Presents positions drawn from
  the user's own files and challenges them to find the best move or cube
  action.

### Research

- **BgRLEngine** — Self-play reinforcement learning engine for backgammon
  and variants. Python / PyTorch; ONNX export for C# runtime interop.

### Planned

- **Tournament App** — Arena for computer engines to compete under controlled
  conditions (match play, knockout tournaments, money sessions). Engines
  participate through a common interface. Not yet broken out as its own
  submodule.

For status, dependency graph, and cross-cutting facts, see `INSTRUCTIONS.md`.
That file is the canonical state-of-the-world document; this file is the
canonical why-and-how document.

## Data Sources

| Source | Status |
|--------|--------|
| XG `.xg` match files | Supported |
| XG `.xgp` position files | Supported |
| Other engines / formats | Future |

## Repository Conventions

- All repos under the same GitHub account (`halheinrich`)
- Each repo has its own solution (`.slnx`) and test project
- Shared test helpers live in the test project of the library that owns them;
  downstream projects reference the library, not its test project
- Shared test data lives in `backgammon/TestData/` (umbrella), referenced by
  every subproject's test project
- Target: current .NET LTS or latest stable
- XML doc comments on all public API surface
