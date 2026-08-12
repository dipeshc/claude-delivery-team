---
by: owner (run finding — observed live; root stopped the duplicate)
---

# A queued message can resurrect a dead Manager, defeating the singleton check

**What's wrong:** The "never two Managers" check tests whether a Manager is
*currently* alive. It cannot see a message already queued against a dead one,
which resurrects it the moment the session resumes — so a correct liveness
check can authorise a relaunch that produces two Managers.

**Evidence:** `skills/team/SKILL.md` § "Never two Managers" gates relaunch on a
running task plus real liveness signals (recent merges, moving branch tips,
worktree mtimes, transcript growth). In this run every one of those was
correctly negative — no live task, no worktree file activity for six hours, no
branch or default-branch movement — so root relaunched. The previous Manager
then woke in the same minute, executing a `SendMessage` root had queued against
it hours earlier, and both Managers proceeded to dispatch a Merge-Clerk at the
same approved branch. Root detected it and stopped the newer one; no damage
resulted, but only because the collision was noticed within about a minute.

**Consequence:** Two orchestrators against one working tree — the exact hazard
the singleton rule exists to prevent. Two clerks cherry-picking the same branch
race for the landing, and the loser's state is left behind for someone to
misread. The failure is silent at the moment of relaunch: every signal the spec
tells root to check says "dead".

**Acceptance criteria:**

- The spec states that liveness signals are necessary but **not sufficient**:
  before relaunching, root must also account for messages it has queued against
  the presumed-dead Manager, because delivery can resurrect it later.
- It gives root a concrete discipline — the minimum is that root tracks whether
  it has an undelivered message outstanding and treats that as "may return",
  and that a relaunch is announced to any resurrected instance so one of the two
  stands down rather than both proceeding.
- It states the recovery action when two are detected: stop the one with less
  context (normally the newly launched one), verify no double-landing occurred
  (`git log` on the default branch, stray `CHERRY_PICK_HEAD`, duplicate item
  branches), and report.
- The existing liveness list is kept — this adds a condition, it does not
  replace one.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `skills/team/SKILL.md`, and
`docs/team-charter.md` only if the rule genuinely belongs in the shared charter
rather than the root-facing skill — judge that while implementing and say which
you chose. `docs/backlog/P2-relaunch-can-double-manager.md` (delete).

**Suggested approach:** Sharpening the existing recovery procedure. Note this is
a root-instance rule: it constrains who *relaunches*, not the Manager itself.
File-disjoint from the two `docs/team-charter.md` items unless you conclude the
charter needs the rule too.
