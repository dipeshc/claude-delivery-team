---
by: owner (shakedown finding F5)
---

# Bless worktree-less review for a docs-only diff

**What's wrong:** The Reviewer's clean-worktree discipline is written as
unconditional, but for a docs-only diff a worktree buys nothing — so the
shakedown Reviewer skipped it and reviewed via `git show`/`git diff`. The
judgment was right; the spec says otherwise, which makes correct behaviour look
like a deviation.

**Evidence:** `agents/reviewer.md:62` opens "Clean-worktree discipline — every
activation, before anything else", with no exception. The same file already
carves docs-only diffs out of verification at `agents/reviewer.md:146`
("Docs-only diff … review-only. No test runs, no typecheck — prose cannot fail
them; your reading IS the verification"), so the exception exists for the
expensive half of the job but not the cheap one.

**Consequence:** Either the Reviewer wastes a worktree create/reset/install
cycle on a diff it will only read, or it does the sensible thing and diverges
from its own spec. Both are bad; the second erodes the spec's authority.

**Acceptance criteria:**

- `agents/reviewer.md`'s clean-worktree section states that a **docs-only diff**
  (nothing outside the docs root — the same test the verification section
  already uses) may be reviewed without a worktree, read directly from the
  branch (e.g. `git show <branch>:<path>`, `git diff <base>...<branch>`).
- Any diff touching a non-docs path keeps the existing unconditional
  clean-worktree discipline, and the verdict says which path was taken.
- The two docs-only carve-outs use identical wording so they cannot drift apart.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/reviewer.md`,
`docs/backlog/P3-docs-only-review-worktree.md` (delete).

**Suggested approach:** Sharpening an already-decided thing (docs-only diffs are
cheaper to review) — not a new policy. Shares `agents/reviewer.md` with
`P2-tool-grant-claims-are-false`; group them if dispatched together.
