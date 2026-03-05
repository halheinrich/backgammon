\# Backgammon Tools — Project Vision



\## Mission



Build an open, engine-agnostic ecosystem of backgammon tools in C# / .NET that serves three audiences simultaneously:



\- \*\*Personal\*\* — a toolkit for analysing and improving one's own game

\- \*\*Community\*\* — a platform tournament and club players can use to study their matches and compete

\- \*\*Developers\*\* — well-structured, well-tested open-source libraries that others can build on



All projects live on GitHub and are designed to be forked, extended, and composed.



---



\## Guiding Principles



\- \*\*Engine-agnostic by design.\*\* XG (eXtreme Gammon) is the first and primary data source, but abstractions should allow other engines and formats to be added without breaking consumers. The RL engine is designed around a variant-agnostic game interface so that standard backgammon, Nackgammon, Hypergammon, Longgammon, and future variants are all first-class citizens.

\- \*\*Library-first.\*\* Every project should expose a clean public API usable as a dependency, not just a CLI or UI. UIs and pipelines are built on top of libraries, not the other way around.

\- \*\*Test-driven.\*\* Binary format parsing, XGID encoding, decision iteration, and CSV/JSON output all have unit tests. New projects inherit this culture.

\- \*\*C# / .NET throughout.\*\* Consistency across the ecosystem lowers the barrier for contributors and makes shared utilities (models, helpers, test infrastructure) straightforward to reuse. The RL engine stack is TBD but must interop cleanly with the C# ecosystem.

\- \*\*Composable over monolithic.\*\* Small, focused repos with clear responsibilities, wired together by consumers rather than bundled into one large project.



---



\## Projects



\### `ConvertXgToJson\_Lib`

\*\*Status:\*\* Active / stable  

\*\*Role:\*\* Core data layer — the foundation everything else depends on.



Parses `.xg` and `.xgp` files produced by eXtreme Gammon into strongly-typed C# models, serialises them to JSON, and exposes a `XgDecisionIterator` that yields `DecisionRow` records (one per analysed checker play or cube decision) suitable for downstream analysis.



Key capabilities:

\- Full binary parsing of XG's multi-stream ZLib format (`temp.xg`, `temp.xgi`, `temp.xgr`, `temp.xgc`)

\- XGID encoding for any position/game state

\- Rollout context resolution (depth label, games rolled, ply level)

\- CSV export of decisions with match score, player, error, equity, analysis depth



---



\### Backgammon Quiz App

\*\*Status:\*\* Planned  

\*\*Platform:\*\* Blazor on Azure  

\*\*Role:\*\* Player-facing study tool built on top of `ConvertXgToJson\_Lib`.



Presents positions drawn from the user's own `.xg` / `.xgp` files and challenges them to find the best move or cube action. Covers:

\- Checker play decisions

\- Cube decisions (double / take / drop)

\- Opening rolls

\- Position type recognition (blitz, priming, back games, races, holding games, etc.)



Positions are sourced directly from analysed XG files, so the quiz is grounded in real games and real errors rather than curated puzzles. Difficulty and category filtering are driven by the `DecisionRow` metadata already produced by `ConvertXgToJson\_Lib`.



---



\### Backgammon Tournament App

\*\*Status:\*\* Planned  

\*\*Role:\*\* Arena for computer engines to compete under controlled conditions.



Supports:

\- \*\*Match play\*\* — fixed-length matches between engines

\- \*\*Knockout / bracket tournaments\*\* — single or double elimination

\- \*\*Money sessions\*\* — unlimited play with running score



Engines participate through a common interface, making it straightforward to pit the RL engine against XG, GNU Backgammon, or any other conforming implementation. Results are persisted for statistical analysis.



---



\### Backgammon RL Engine

\*\*Status:\*\* Research / early planning  

\*\*Role:\*\* A self-play reinforcement learning engine for backgammon and variants.



Goals:

\- Learn entirely from self-play (no supervised pre-training on existing engine data)

\- Support a variant-agnostic game interface so the same architecture covers standard backgammon, Nackgammon, Hypergammon, Longgammon, and future variants with minimal changes

\- Compete in the Tournament App against established engines as a benchmark

\- Serve as a research platform for RL techniques applied to combinatorial board games with dice



Tech stack is TBD. Key constraint: must interop with the C# / .NET ecosystem well enough to be driven by the Tournament App and potentially serve positions to the Quiz App.



---



\## Data Sources



| Source | Status |

|--------|--------|

| XG `.xg` match files | Supported |

| XG `.xgp` position files | Supported |

| Other engines / formats | Future |



---



\## Repository Conventions



\- All repos under the same GitHub account/org

\- Each repo has its own solution (`.slnx`) and test project

\- Shared test helpers live in the test project of the library that owns them; downstream projects reference the library, not its test project

\- Target: current .NET LTS or latest stable

\- XML doc comments on all public API surface

