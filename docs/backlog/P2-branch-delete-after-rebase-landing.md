---
by: owner (run finding — found by exercising the rule minutes after it landed)
---

# No sanctioned way to delete a branch landed via the rebase path

**What's wrong:** The cleanup rule makes `git branch -d` the default and
reserves `-D` for the rescue-then-force path — but `-d` refuses for **every**
branch landed via cherry-pick, which is the normal landing whenever the base
moved. There is currently no sanctioned way to delete a normally-landed branch.

**Evidence:** `docs/team-charter.md` and `agents/manager.md` (the cleanup
ownership rule landed in `c86b1f1`) prescribe the safe forms. Immediately after
that commit landed, the Manager ran `git branch -d item/spec-cleanup-and-rerere`
and got `error: the branch 'item/spec-cleanup-and-rerere' is not fully merged`,
then correctly stopped rather than forcing. The refusal is correct by git's
rules and wrong in fact: the Merge-Clerk's cherry-pick creates a **new commit
object**, so the branch tip is not an *ancestor* of the default branch even
though its content is fully applied. `git cherry main <branch>` reports `-`
(applied upstream) and `git range-diff` reports `=` (patch-identical).

**Consequence:** Every rebase-path landing leaves an undeletable `item/*`
branch. That breaks a load-bearing signal: the charter uses
`git branch --list 'item/*'` to enumerate in-flight work, so stale branches make
a Manager see phantom work in flight, and a future agent may "recover" a branch
whose work shipped hours ago. The alternative — agents reaching for `-D`
whenever `-d` refuses — trains exactly the reflex the safe-form rule exists to
prevent.

**Acceptance criteria:**

- The spec states the correct precondition for deleting a landed branch:
  **ancestry is the wrong test under a cherry-pick landing style**. The
  substitute already exists — `git cherry <default-branch> <branch>` emitting
  only `-` lines proves every commit is applied upstream.
- With that proof obtained, the forced form is **sanctioned and not a policy
  violation**; without it, the refusal stands and the agent stops and reports.
  The rule makes the ordering explicit: prove, then delete — never force first.
- The rescue-before-force rule is untouched and still applies to a worker
  believed dead; this clause covers only a branch whose content is provably
  landed.
- Wherever the owning agent file states the cleanup commands, it states this
  precondition too, so the two cannot drift.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification. The reviewer should confirm the `git cherry` test
  actually behaves as claimed rather than accepting the reasoning.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, `agents/manager.md`,
`docs/backlog/P2-branch-delete-after-rebase-landing.md` (delete).

**Suggested approach:** The owner's ruling, so this is not a decision to
re-litigate: `git cherry` proving upstream application is the sanctioned
precondition for the forced delete. Implement that; do not invent a third
option. Shares `docs/team-charter.md` and `agents/manager.md` with
`P2-resign-audit-recipe-unsafe`.
