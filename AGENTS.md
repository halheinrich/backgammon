# Backgammon — Shared Rules

Applies to all sub-projects: BgRLEngine, BgMoveGen, and any future additions.

## Session defaults
- Default to plan mode: propose before writing code or files, wait for explicit approval.
- To proceed: say "go", "proceed", or "implement it".
- To skip plan mode for a specific task: say "just do it" or "no plan needed".

## Output discipline
- Profiling and benchmarks: write targeted microbenchmarks that time phases in isolation.
  Output should be decision-useful — enough to identify the bottleneck and compare before/after.
  Not a cProfile dump; not artificially compressed.
- Code output: prefer focused diffs over full-file reprints when changing existing files.
- Console/log output from tools and scripts: surface what's needed to act, suppress noise.

## Commit protocol
After committing in any sub-project:
1. Note the short hash (`git rev-parse HEAD`)
2. Update the commit hash and any raw URLs in that sub-project's instructions doc
3. Return to the umbrella project and update hashes + umbrella INSTRUCTIONS.md

## Token hygiene
- Avoid re-summarizing context already in the instructions doc.
- Don't reprint large code blocks unless directly editing them.
- Long cProfile / log dumps: don't paste into chat — describe what you need instead,
  or use the microbenchmark approach.
