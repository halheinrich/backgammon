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
about to commit), stop and let the user react. Checkpoints are relaxed — fine
to complete multiple related edits before pausing, but never finish an entire
multi-step task without an intermediate nod.

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
5. **All output.** Applies to code suggestions, architecture proposals, test
   strategies, and commit/branching patterns.

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
