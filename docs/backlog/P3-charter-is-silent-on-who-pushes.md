---
by: owner
---

# The charter governs who writes the tree but not who pushes

**What's wrong:** The charter's "Scoped writers to the main working tree"
section specifies who may write, by what mechanism, and under what proof — but
says nothing about who may push the default branch to a remote. The one
operation that makes every landing externally visible is unassigned.

**Evidence:** `docs/team-charter.md`, the scoped-writers section (rewritten
end-to-end when the direct-lane entry landed, still without a push clause). The
gap is rediscovered by working agents rather than by audits: a Merge-Clerk
flagged unexplained `origin/main` movement as a possible foreign writer in two
separate runs, and a Manager did the same in a third — each time the movement
was root's sanctioned post-landing push, and each time the agent had no rule to
check it against.

**Consequence:** Every careful agent that reconciles against the remote must
either re-derive "root pushes after landings" from a relay conversation or
report a false anomaly. Three reports across three runs is recurring noise, and
noise trains readers to skim the report that finally matters — the same decay
this repo's known-failures discipline exists to prevent elsewhere.

**Acceptance criteria:**

- The scoped-writers section (or an adjacent clause it points to) states who
  pushes the default branch and when. The current practice to encode, which the
  owner has operated all along: **root pushes after landings, on the owner's
  standing instruction; no team agent pushes.** Encode the practice — do not
  redesign it.
- The rule tells a reader what remote movement means: `origin/<default-branch>`
  advancing to a commit already on the local default branch is root's push and
  is accounted for; any other remote movement remains reportable.
- Agent files that reason about the remote (Merge-Clerk, Manager) need at most
  a pointer — the charter owns the rule; do not restate it per role.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, possibly one-line pointers
in `agents/merge-clerk.md` and `agents/manager.md`,
`docs/backlog/P3-charter-is-silent-on-who-pushes.md` (delete).

**Suggested approach:** A sharpening — the practice is settled and
owner-instructed; only its absence from the spec is the defect.
