# Backgammon — Shared Rules

Applies to all sub-projects in this repository.

## Project structure conventions
- INSTRUCTIONS.md lives at the repo root alongside the .slnx file
- AGENTS.md lives at the umbrella repo root only — subprojects reference it, not their own copy
- Never move these files based on session advice
- VS solution layout: `.slnx` at repo root, source files under `{ProjectName}/{path}`. All repo-relative source file paths require a `{ProjectName}/` prefix.


## Session start
1. Fetch and apply the sub-project INSTRUCTIONS.md.
2. Fetch and apply AGENTS.md (this file).
3. INSTRUCTIONS.md: fetch if available; session-start message carries the critical summary and is sufficient if fetch fails.
4. Wait for the user to state the task — do not propose work unprompted.

## Session defaults
- Default to plan mode: propose before writing code or files, wait for explicit approval.
- To proceed: say "go", "proceed", or "implement it".
- To skip plan mode for a specific task: say "just do it" or "no plan needed".
- At the end of each plan, list any unresolved questions that need answers before implementation.

## Profiling and benchmarks
- Always use targeted microbenchmarks, never cProfile runs.
- Time phases in isolation (move gen, state construction, encoding, forward pass).
- Output: a clean table — phase, calls/sec, ms/call, % of total. Fits in a terminal.
- Decision-useful: enough to identify the bottleneck and compare before/after. No noise.

## Output discipline
- Code output: prefer focused diffs over full-file reprints when changing existing files.
- Console/log output from tools and scripts: surface what's needed to act, suppress noise.
- Don't paste long log dumps into chat — describe what you need instead.

## Stale repo disaster — and how to prevent it

### What happened
A session began with Claude fetching INSTRUCTIONS.md, which pointed to commit `eb61c10`.
The repo had moved on — `DiagramOrientation` had been removed and `HomeBoardOnRight` added
in a prior session that didn't update INSTRUCTIONS.md. Claude worked from the stale snapshot
all session, copied dead field names into new code, and produced a cascade of build errors
that took significant time to untangle.

### Root cause
INSTRUCTIONS.md was not updated at the end of the prior session. The commit hash and raw
URLs pointed to old code. Claude had no way to detect the drift.

### Prevention rules
1. **INSTRUCTIONS.md is the source of truth for the next session.** If it doesn't match
   HEAD, the next session starts with bad data. Updating it is not optional.
2. **INSTRUCTIONS.md must be committed as the final act of every session** — after all
   other commits, with the correct hash and correct raw URLs for every file.
3. **The submodule pointer and INSTRUCTIONS.md must always be committed together** in the
   same umbrella commit. Never commit one without the other.
4. **At the start of every session, Claude must fetch INSTRUCTIONS.md first** and use only
   the raw URLs listed there. Never construct URLs from memory or from a prior session's
   context.
5. **If INSTRUCTIONS.md hash does not match the actual HEAD**, Claude must stop, flag the
   discrepancy, and ask for the correct URLs before touching any code.
6. **Claude must ask for a URL rather than guess.** The fetch tool only accepts URLs
   provided by the user or returned by prior fetches. Claude must ask the user to supply
   any URL it needs rather than constructing one and failing silently.

### Resolution
If INSTRUCTIONS.md hash doesn't match actual HEAD:
1. Run `git log --oneline -5` in the submodule dir to see what changed
2. Fetch current INSTRUCTIONS.md from githack to assess how stale it is
3. Reconstruct what changed between pinned hash and HEAD from git log or by fetching files at both hashes
4. Update INSTRUCTIONS.md with correct hash and URLs
5. Commit umbrella — submodule pointer + INSTRUCTIONS.md staged together

## Best practice guidance

When proposing or reviewing code, Claude must proactively flag any deviation from best
practice — even if the deviation is minor or common. Specifically:

1. **Flag it explicitly.** If a proposed approach is not best practice, say so clearly
   rather than waiting to be asked.
2. **State the best practice alternative.** Describe what the preferred approach would be.
3. **Give the tradeoffs.** Explain what is gained and what is lost by deviating — e.g.
   simplicity vs. correctness, short-term convenience vs. long-term maintainability.
4. **Don't decide for the user.** Present the tradeoffs and let Hal make the call. Do not
   silently proceed with a non-best-practice approach because it seems expedient.
5. **This applies to all output** — code suggestions, architecture proposals, test
   strategies, and commit/branching patterns.

## Commit protocol
After committing in any sub-project:
1. Note the short hash (`git rev-parse --short HEAD`)
2. Update the commit hash and any raw URLs in that sub-project's instructions doc
3. Return to the umbrella project and update hashes + umbrella INSTRUCTIONS.md
The subproject session owns the commit. Never hand off to the umbrella with
uncommitted work — commit, push, and include the new hash in the handoff message.

## Token hygiene
- Avoid re-summarizing context already in the instructions doc.
- Don't reprint large code blocks unless directly editing them.

## Context management

### Within a session
- Before proposing a design or approach, check whether it was already discussed
  and decided earlier in the conversation. Don't re-litigate settled decisions.
- When the conversation gets long, re-read the INSTRUCTIONS.md mentally before
  making claims about project state — don't rely on fading context.
- If you're uncertain whether something was decided, say so — don't guess.

### Between sessions
- The INSTRUCTIONS.md is the single source of truth for project state.
  If it wasn't captured there, it didn't happen.
- At session end, identify anything decided or changed that isn't yet
  reflected in INSTRUCTIONS.md. Flag it explicitly — don't let it evaporate.
- When starting a new session, trust INSTRUCTIONS.md over memory summaries.
  Memory is a convenience hint; the doc is canonical.
  
## Fetching source files
Standard URL format for all source file links:
`https://raw.githack.com/halheinrich/{repo}/{hash}/{path}`

Use the pinned commit hash from the submodule table — not `main`.

- `raw.githack.com` — plain text, no CDN caching, always current for the hash
- `cdn.githack.com` — CDN-cached, faster but may lag on fresh commits; avoid for session fetches

`raw.githubusercontent.com` is blocked in Claude's container. `github.com` blob URLs are flaky (robots.txt + rate limiting). Use `raw.githack.com` exclusively.

## Existing-file edits
When modifying an existing file, verify the fetched version is current before
delivering changes. If the commit hash in the URL doesn't match the pinned
commit in INSTRUCTIONS.md, flag the mismatch — don't proceed with stale code.

Prefer delivering only the new code (method, block, using directive) with clear
insertion-point instructions over full-file replacements, as a defense against
stale snapshots.

Full-file delivery is fine for new files that don't exist yet.