# Handoff Templates

## How to use

Copy the relevant template, fill in the blanks, and paste into the new session as
two separate messages. Message 1 first, wait for results, then Message 2.

The dependency `cd` blocks in Message 1 come from the subproject's INSTRUCTIONS.md
Depends on / Dependency files section. Add or remove blocks as needed.

---

## Template — subproject with dependencies

### Message 1

```
I need you to give me the output of these PowerShell commands. Please paste
the results — I'll wait.

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git log --oneline -1 -- AGENTS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\__SUBPROJECT__"
git rev-parse --short HEAD
git log --oneline -1 origin/main
git log --oneline -1 -- INSTRUCTIONS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\__DEP1__"
git rev-parse --short HEAD

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\__DEP2__"
git rev-parse --short HEAD

Do not search the web. Do not fetch any files. Do not use any tools.
Just wait for my response.
```

### Message 2

```
Use the AGENTS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md
Use the INSTRUCTIONS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/__REPO__/{hash}/INSTRUCTIONS.md
Use the dependency hashes to construct fetch URLs per the Dependency files
section in INSTRUCTIONS.md.

Do NOT fetch anything using `main` — CDN caching returns stale content.

**What changed upstream:**
- __describe changes__

**Tasks:**
1. __task__
```

---

## Template — subproject with no dependencies

### Message 1

```
I need you to give me the output of these PowerShell commands. Please paste
the results — I'll wait.

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git log --oneline -1 -- AGENTS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\__SUBPROJECT__"
git rev-parse --short HEAD
git log --oneline -1 origin/main
git log --oneline -1 -- INSTRUCTIONS.md

Do not search the web. Do not fetch any files. Do not use any tools.
Just wait for my response.
```

### Message 2

```
Use the AGENTS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md
Use the INSTRUCTIONS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/__REPO__/{hash}/INSTRUCTIONS.md

Do NOT fetch anything using `main` — CDN caching returns stale content.

**Tasks:**
1. __task__
```

---

## Filled example — ExtractFromXgToCsv

### Message 1

```
I need you to give me the output of these PowerShell commands. Please paste
the results — I'll wait.

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git log --oneline -1 -- AGENTS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\ExtractFromXgToCsv"
git rev-parse --short HEAD
git log --oneline -1 origin/main
git log --oneline -1 -- INSTRUCTIONS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\BgDataTypes_Lib"
git rev-parse --short HEAD

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\ConvertXgToJson_Lib"
git rev-parse --short HEAD

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\XgFilter_Lib"
git rev-parse --short HEAD

Do not search the web. Do not fetch any files. Do not use any tools.
Just wait for my response.
```

### Message 2

```
Use the AGENTS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md
Use the INSTRUCTIONS.md hash to fetch:
https://raw.githubusercontent.com/halheinrich/ExtractFromXgToCsv/{hash}/INSTRUCTIONS.md
Use the dependency hashes to construct fetch URLs per the Dependency files
section in INSTRUCTIONS.md.

Do NOT fetch anything using `main` — CDN caching returns stale content.

**What changed upstream:**
- XgFilter_Lib: all filters accept IDecisionFilterData; DecisionFilterSet.Apply removed;
  ErrorRangeFilter handles nullable; InnerBoard54321Classifier added
- BgDataTypes_Lib: DecisionRow lives here now (namespace BgDataTypes_Lib);
  Board is IReadOnlyList<int>; IDecisionFilterData interface
- ConvertXgToJson_Lib: DecisionRow removed; IterateDiagramRequests yields BgDecisionData

**Tasks:**
1. Adjust to upstream changes — update ProjectReferences, resolve DecisionRow namespace
2. Add diagram request support using IterateDiagramRequests
```

---

## Quick reference — which subprojects have which dependencies

| Subproject | Dependencies |
|---|---|
| BgDataTypes_Lib | none |
| ConvertXgToJson_Lib | BgDataTypes_Lib |
| XgFilter_Lib | BgDataTypes_Lib, ConvertXgToJson_Lib |
| ExtractFromXgToCsv | BgDataTypes_Lib, ConvertXgToJson_Lib, XgFilter_Lib |
| XgAnalytics | (check INSTRUCTIONS.md) |
| BackgammonDiagram_Lib | BgDataTypes_Lib |
| BgDiag_Razor | BackgammonDiagram_Lib |
| BgRLEngine | BgMoveGen |
| BgMoveGen | none |
| BgQuiz_Blazor | BgDiag_Razor |
