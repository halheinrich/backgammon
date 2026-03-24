# Backgammon — Shared Rules

Applies to all sub-projects in this repository.

## Session start
1. Fetch and apply the sub-project INSTRUCTIONS.md.
2. Fetch and apply AGENTS.md (this file).
3. Emit a brief context summary: current state, active deferred items, rules in effect.
4. Wait for the user to state the task — do not propose work unprompted.

## Session defaults
- Default to plan mode: propose before writing code or files, wait for explicit approval.
- To proceed: say "go", "proceed", or "implement it".
- To skip plan mode for a specific task: say "just do it" or "no plan needed".

## Profiling and benchmarks
- Always use targeted microbenchmarks, never cProfile runs.
- Time phases in isolation (move gen, state construction, encoding, forward pass).
- Output: a clean table — phase, calls/sec, ms/call, % of total. Fits in a terminal.
- Decision-useful: enough to identify the bottleneck and compare before/after. No noise.

## Output discipline
- Code output: prefer focused diffs over full-file reprints when changing existing files.
- Console/log output from tools and scripts: surface what's needed to act, suppress noise.
- Don't paste long log dumps into chat — describe what you need instead.

## Commit protocol
After committing in any sub-project:
1. Note the short hash (`git rev-parse HEAD`)
2. Update the commit hash and any raw URLs in that sub-project's instructions doc
3. Return to the umbrella project and update hashes + umbrella INSTRUCTIONS.md

## Token hygiene
- Avoid re-summarizing context already in the instructions doc.
- Don't reprint large code blocks unless directly editing them.

## Fetching source files
GitHub raw URLs are blocked in Claude's container. Standard pattern:
1. Ask Claude: "Give me the URLs I need to fetch"
2. Claude lists the raw GitHub URLs
3. Paste those URLs back into the chat
4. Claude calls web_fetch on each URL

If a raw URL returns 404, use the blob URL as fallback:
- Raw: `https://raw.githubusercontent.com/halheinrich/REPO/main/FILE`
- Blob: `https://github.com/halheinrich/REPO/blob/main/FILE`

Blob URLs work but return HTML with navigation chrome — raw is preferred when available.