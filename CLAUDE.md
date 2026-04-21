# Backgammon — Umbrella Project Guide for Claude Code

Canonical conventions for Claude Code sessions working on this repo.
Read this at session start.

## What this is

Umbrella repo at `D:\Users\Hal\Documents\Visual Studio 2026\Projects\backgammon`.
C# / .NET 10, Visual Studio 2026, Windows.
Ten git submodules — see `INSTRUCTIONS.md` for the full list, status, and
dependency graph.
`VISION.md` (once cleaned up) for project mission and principles.

## Working directory

**Default: operate in the main checkout** at the repo root.
Do not work inside `.claude/worktrees/*`. Worktrees add submodule-init friction
and concept load without matching benefit for a single-developer, sequential
workflow. If a session starts in a worktree by accident, exit it first — don't
begin work.

## Shell

Claude Code's Bash tool uses Git Bash on Windows. Use Unix shell syntax for
tool calls: `cd`, `&&`, forward slashes, `/dev/null` (not `NUL`).

## Submodule boundary

Each subproject is its own git repo, with its own commits, its own branches,
its own INSTRUCTIONS.md.

**Rule: one session, one repo.**

- An umbrella session commits umbrella-level work only: submodule pointer
  bumps, umbrella `INSTRUCTIONS.md`, umbrella `CLAUDE.md`, `VISION.md`,
  `.gitignore`, `.gitattributes`, `TestData/` structure.
- A subproject session commits subproject-level work only: source code, tests,
  subproject `INSTRUCTIONS.md`.
- Never commit across the boundary. If umbrella work and subproject work are
  both needed, they go in two separate sessions and two separate commits.

An umbrella session may *read* subproject files to understand what changed or
to draft commit messages — reading across the boundary is fine, writing is not.

## Planning discipline

For non-trivial tasks, produce a plan before writing code and get explicit
user approval to proceed. For trivial tasks (one-line fixes, typos, obvious
renames) skip the plan.

What a plan covers is a matter of judgment, not a checklist. At minimum:

- **What** — what the change needs to accomplish
- **How** — the chosen approach, with alternatives and tradeoffs if real
  choices exist
- **Open questions** — anything needing user input before implementation starts

More involved changes may warrant more detail (test strategy, best-practice
alignment for each alternative, risk analysis). Less involved changes warrant
less. Err on the side of tight plans; bloat is worse than terseness.

To proceed past a plan: "go", "proceed", or "implement it". To skip plan mode
for a specific trivial task: "just do it" or "no plan needed".

## Pair-programming cadence

**Checkpoint discipline.** Non-trivial plans are broken into checkpoint-sized
steps. At natural boundaries (plan approved, first chunk done, build green,
about to commit), stop and let the user react.

**For multi-task sessions, execute tasks one at a time.** Produce the full
plan up front and get it approved. Then: implement Task 1 → commit → report
a diff summary → wait for explicit user approval before starting Task 2.
Repeat per task. Do not batch multiple tasks into one commit or offer to
"do them all" after the plan is approved. User digest capacity is the review
bottleneck; task-by-task commits let each change be read and approved before
the next lands on top.

**Uncertainty signaling.** When making a factual claim about the codebase, a
git hash, an API, a tool version, or a behavior not verified in the current
session, either verify it or flag it as uncertain. No confident-sounding
guesses. Honest "I don't know" is always preferred to a hedged assertion.

**Verification before claiming done.** A task isn't done because the edit
looks correct. It's done when the thing has been run — build succeeds, tests
pass, or for doc changes, the final file has been re-read. If verification
isn't possible, say so and explicitly downgrade the claim from "done" to
"drafted, unverified."

**Scope discipline.** Do what the user asked. Related cleanups noticed in
passing get *mentioned* at the end of the turn ("I noticed X is stale; want me
to fix that separately?") — never silently folded into the current change.
A task-scoped diff is reviewable; a task-plus-cleanups diff isn't.

## Best-practice flagging

When proposing or reviewing code, proactively flag any deviation from best
practice — even minor or common. Specifically:

1. **Flag explicitly.** If a proposed approach is not best practice, say so
   clearly rather than waiting to be asked.
2. **State the alternative.** Describe what the preferred approach would be.
3. **Give the tradeoffs.** Explain what is gained and what is lost by
   deviating.
4. **Don't decide.** Present the tradeoffs; let the user make the call. Do not
   silently proceed with a non-best-practice approach because it seems
   expedient.
5. **Present options with a recommendation.** When offering the user a choice
   between alternatives, identify which one is best practice and why. Neutral
   enumeration is a failure mode — it hides the recommendation behind
   diplomacy and pushes design decisions onto the user who asked for guidance.
   "Don't decide" means accept the user's final call; it does not mean
   withhold the recommendation.
6. **All output.** Applies to code suggestions, architecture proposals, test
   strategies, and commit/branching patterns.

## API breakage bias

The ecosystem has no external consumers yet. All callers live in this repo.
That makes breakage cheap — update callers as part of the same coordinated
change.

Bias toward fixing the API when it's wrong. Do not preserve an awkward
signature, a misleading name, or a legacy helper just because existing code
depends on it. "Existing caller" is not a reason to keep bad API; it's a
thing to update.

When a break crosses subprojects, the pattern is: break in the producer's
session, adapt each consumer in its own session, close with one coordinated
umbrella pointer-bump that moves all affected submodules at once.

Re-evaluate when external consumers appear. Until then, aesthetic and
correctness win over source compatibility.

## Best-practice bias

When writing new code, default toward robust over minimal. YAGNI applies
to speculative *features* — not to correctness, completeness, or defensive
handling of the contracts you're writing.

Practical tilts:

- **Edge-case inputs.** Empty / single-element / boundary inputs: define
  the contract and test it, even if no current caller exercises it. Silent
  wrong-answer bugs live here.
- **Extensible shapes.** Filter / classifier / helper classes with only one
  implementation today: build them as extensible abstractions parallel to
  sibling patterns already in the codebase, not one-off specialised classes.
- **Stubs with imminent callers.** A stub without callers is a YAGNI
  candidate. A stub whose caller lands next session is not — don't remove
  it.

Cost asymmetry: in this ecosystem (no external consumers, all callers
in-tree), too-robust is a small one-time up-front investment; too-YAGNI
surfaces later as phantom bugs, missed edge cases, or cross-subproject
cleanup cycles.

Complementary to "API breakage bias": that one governs *removing* awkward
existing API; this one governs *building* new code. Both lean quality
over expedience.

## Commit protocol

1. Prefer new commits over amending.
2. Stage files explicitly (`git add <path>`), not `git add -A` or `git add .`
   — avoids accidentally committing secrets or artifacts.
3. Never skip hooks (`--no-verify`) or bypass signing unless the user
   explicitly requests it.
4. Never force-push to `main` / `master`.
5. Never run destructive operations (`git reset --hard`, `git push --force`,
   `git clean -f`, `git branch -D`) without explicit user confirmation.
6. Commit messages: substantive body describing *what* changed and *why*, not
   just *that* it changed. The umbrella session reading a subproject's
   `git log` needs those messages to carry the story.
7. End every commit message with:

   ```
   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

8. Always pause for explicit user confirmation before pushing to origin.

## Subproject ↔ umbrella workflow

**Subproject → umbrella:** after a subproject commit, the umbrella session
picks up changes via `git log` in the submodule directory. No handoff file.
Substantive commit messages in the subproject are what carry the information
across.

**Umbrella → next task:** pending work and priorities live in the umbrella
`INSTRUCTIONS.md` under "Next up" / "Pending" / "Deferred" sections. Updated
as part of the same commit that makes any relevant decision. Not a transient
handoff document.

**To bump a submodule pointer from the umbrella:**

1. `git submodule update --remote <name>` — fetches the submodule's
   `origin/main` and moves its working tree.
2. `cd <subproject> && git log <OLD>..<NEW> --oneline` — review what changed.
3. Decide whether umbrella `INSTRUCTIONS.md` needs an edit (rare; umbrella
   records cross-cutting facts only).
4. Stage submodule pointer + any `INSTRUCTIONS.md` changes together.
5. Commit with a message naming the submodule and the gist of the change.
6. Push (after user confirmation).

## Subproject INSTRUCTIONS.md standard

Each subproject's `INSTRUCTIONS.md` is the deep reference for working in
that subproject. It complements the umbrella docs without duplicating them:

- `VISION.md` owns mission, principles, repo conventions.
- Umbrella `INSTRUCTIONS.md` owns cross-cutting status, dependency graph,
  and per-subproject one-paragraph summaries.
- Umbrella `CLAUDE.md` (this file) owns session conventions.
- Subproject `INSTRUCTIONS.md` owns everything else specific to working
  in that subproject.

**Required sections, in this order:**

1. **Header breadcrumb** — links to `../CLAUDE.md`, `../INSTRUCTIONS.md`,
   `../VISION.md`. One line each, blockquote.
2. **Stack** — one-line summary.
3. **Solution** — absolute `.slnx` path. Non-.NET subprojects substitute
   an equivalent (e.g., `pyproject.toml` for Python).
4. **Repo** — GitHub URL and branch.
5. **Depends on** — bullets; each dependency names what it provides this
   subproject. Write "Standalone" if none.
6. **Directory tree** — actual layout, kept current.
7. **Architecture** — the substance. Types, invariants, design decisions,
   hot-path rules, internal patterns. Detail too long for the umbrella's
   per-subproject summary lives here.
8. **Public API** — concrete signatures and contracts consumers depend on.
9. **Pitfalls** — things easy to get wrong when modifying this subproject.
10. **Subproject-internal next steps** — work bounded by this subproject.
    Cross-cutting items go in umbrella `INSTRUCTIONS.md` "Next up", not here.

**Forbidden content** (drift-prone, duplicative, or Claude.ai-web artifact):

- Commit hashes anywhere — git tracks them
- Raw `githubusercontent.com` URLs to source files — Claude Code reads
  from disk
- "GitHub fetch workaround" sections — irrelevant under Claude Code
- "Session handoff" with hash-bumping instructions — drift trap
- Test counts (e.g. "61 tests pass") — drift trap
- "Current commit" lines — drift trap
- Mission, principles, target framework, repo conventions — owned by
  `VISION.md`
- Per-subproject Purpose / Status / dependency graph — owned by umbrella
  `INSTRUCTIONS.md`

When slimming a subproject doc to this standard, default to deletion of
any forbidden content rather than updating it. The point of the standard
is to make stale facts impossible to introduce.

## Existing-file edits

When modifying an existing file, read the current version from disk first.
Don't edit based on assumptions about what the file contains. Prefer focused
edits over full-file rewrites.

## TestData convention

Shared directory at `backgammon/TestData`, referenced by every subproject's
test project.

- File contents are **not** tracked in the repo (`.gitignore` handles this).
- Directory structure **is** tracked — every structural subdirectory has a
  `.gitkeep`.
- When a new structural subdirectory is introduced, add a `.gitkeep` to
  preserve it through clones.

## Output discipline

- Prefer focused diffs over full-file reprints when changing existing files.
- Don't paste long log dumps into chat — describe what you need instead.
- Reading a file is not the same as dumping it verbatim. Summarize what's
  relevant.

## Known gotchas

**Git index.lock transience.** Claude Code (hosted inside `Claude.exe` on
Windows) runs `git diff --numstat`, `--name-status`, and `--no-ext-diff`
against the repo in the background to populate its own change-view UI. Those
commands take the `.git/index.lock` briefly and race with deliberate writes
from the Bash tool. Wrap git write operations in a short retry loop:

```bash
for i in 1 2 3 4 5 6 7 8 9 10; do
  if git <command> 2>/dev/null; then break; fi
  sleep 0.3
done
```

Usually succeeds on attempt 1 or 2.

**Windows cwd locking.** If the Claude Code session's working directory is
inside a directory being deleted (e.g. removing a worktree from inside it),
Windows won't let the deletion proceed. Exit the directory or restart the
session.

**Submodules in worktrees are uninitialized.** A fresh git worktree has all
submodules uninitialized — another reason to avoid worktrees as the default
workflow.

## What's intentionally not in this file

- **Hooks / `.claude/settings.json`.** None configured. Will add if concrete
  pain points emerge.
- **Per-subproject `CLAUDE.md` files.** Umbrella CLAUDE.md is the single
  source; subproject `INSTRUCTIONS.md` files include a breadcrumb pointing
  back here.
