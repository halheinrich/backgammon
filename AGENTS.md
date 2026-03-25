# Backgammon — Shared Rules

Applies to all sub-projects in this repository.

## Project structure conventions
- INSTRUCTIONS.md lives at the repo root alongside the .slnx file
- AGENTS.md lives at the umbrella repo root only — subprojects reference it, not their own copy
- Never move these files based on session advice

## Session start
1. Fetch and apply the sub-project INSTRUCTIONS.md.
2. Fetch and apply AGENTS.md (this file).
3. INSTRUCTIONS.md: fetch if available; session-start message carries the critical summary and is sufficient if fetch fails.
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
1. Note the short hash (`git rev-parse --short HEAD`)
2. Update the commit hash and any raw URLs in that sub-project's instructions doc
3. Return to the umbrella project and update hashes + umbrella INSTRUCTIONS.md

## Token hygiene
- Avoid re-summarizing context already in the instructions doc.
- Don't reprint large code blocks unless directly editing them.

## Fetching source files
Standard URL format for all source file links:
`https://raw.githack.com/halheinrich/{repo}/{hash}/{path}`

Use the pinned commit hash from the submodule table — not `main`.

- `raw.githack.com` — plain text, no CDN caching, always current for the hash
- `cdn.githack.com` — CDN-cached, faster but may lag on fresh commits; avoid for session fetches

`raw.githubusercontent.com` is blocked in Claude's container. `github.com` blob URLs are flaky (robots.txt + rate limiting). Use `raw.githack.com` exclusively.
