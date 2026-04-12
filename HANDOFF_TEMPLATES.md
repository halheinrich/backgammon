# Handoff Templates

## How to use

Copy the relevant template, fill in the blanks, and paste into the new session as
two separate messages. Message 1 first, wait for Claude to ask you to run the
commands, run them, then paste the output into Message 2 and send.

The dependency `cd` blocks in Message 1 come from the subproject's INSTRUCTIONS.md
Depends on / Dependency files section. Add or remove blocks as needed.

---

## Template — subproject with dependencies

### Message 1

```
I need you to give me the output of these PowerShell commands. Please paste
the results — I'll wait. I will then send a SECOND MESSAGE with instructions
on what to fetch and what to do. Do not fetch or search for anything until
you receive that second message.

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
Wait for the second message.
```

### Message 2

Paste the PowerShell output into the indicated spot, then send.

```
Here are the results:

{paste PowerShell output here}

Now fetch these files using the hashes from the output above:

1. AGENTS.md — use the hash from the first git log command:
   https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md

2. INSTRUCTIONS.md — use the hash from "git log --oneline -1 -- INSTRUCTIONS.md":
   https://raw.githubusercontent.com/halheinrich/__REPO__/{hash}/INSTRUCTIONS.md

3. Dependency files — INSTRUCTIONS.md lists file paths under "Dependency files".
   For each dependency, use the hash from the git rev-parse output above to
   construct URLs:
   https://raw.githack.com/halheinrich/{repo}/{hash}/{path}

Do NOT fetch anything using `main` — CDN caching returns stale content.

After reading AGENTS.md and INSTRUCTIONS.md, follow the session start protocol
in AGENTS.md. Then wait for me to state the task.

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
the results — I'll wait. I will then send a SECOND MESSAGE with instructions
on what to fetch and what to do. Do not fetch or search for anything until
you receive that second message.

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon"
git log --oneline -1 -- AGENTS.md

cd "D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon\__SUBPROJECT__"
git rev-parse --short HEAD
git log --oneline -1 origin/main
git log --oneline -1 -- INSTRUCTIONS.md

Do not search the web. Do not fetch any files. Do not use any tools.
Wait for the second message.
```

### Message 2

```
Here are the results:

{paste PowerShell output here}

Now fetch these files using the hashes from the output above:

1. AGENTS.md — use the hash from the first git log command:
   https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md

2. INSTRUCTIONS.md — use the hash from "git log --oneline -1 -- INSTRUCTIONS.md":
   https://raw.githubusercontent.com/halheinrich/__REPO__/{hash}/INSTRUCTIONS.md

Do NOT fetch anything using `main` — CDN caching returns stale content.

After reading AGENTS.md and INSTRUCTIONS.md, follow the session start protocol
in AGENTS.md. Then wait for me to state the task.

**Tasks:**
1. __task__
```

---

## Filled example — ExtractFromXgToCsv

### Message 1

```
I need you to give me the output of these PowerShell commands. Please paste
the results — I'll wait. I will then send a SECOND MESSAGE with instructions
on what to fetch and what to do. Do not fetch or search for anything until
you receive that second message.

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
Wait for the second message.
```

### Message 2

```
Here are the results:

{paste PowerShell output here}

Now fetch these files using the hashes from the output above:

1. AGENTS.md — use the hash from the first git log command:
   https://raw.githubusercontent.com/halheinrich/backgammon/{hash}/AGENTS.md

2. INSTRUCTIONS.md — use the hash from "git log --oneline -1 -- INSTRUCTIONS.md":
   https://raw.githubusercontent.com/halheinrich/ExtractFromXgToCsv/{hash}/INSTRUCTIONS.md

3. Dependency files — INSTRUCTIONS.md lists file paths under "Dependency files".
   For each dependency, use the hash from the git rev-parse output above to
   construct URLs:
   https://raw.githack.com/halheinrich/{repo}/{hash}/{path}

Do NOT fetch anything using `main` — CDN caching returns stale content.

After reading AGENTS.md and INSTRUCTIONS.md, follow the session start protocol
in AGENTS.md. Then wait for me to state the task.

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