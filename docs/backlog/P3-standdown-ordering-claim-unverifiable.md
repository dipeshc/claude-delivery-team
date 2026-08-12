---
by: owner (review finding — raised by reviewer-2, ruled non-gating, filed instead)
---

# The stand-down rule asserts a delivery ordering the harness does not guarantee

**What's wrong:** The relaunch guard states that a resurrected Manager "reads
the stand-down before it can act". That is a claim about message *delivery
ordering*, which nothing verifies — and it is false in exactly the scenario the
rule was written to fix.

**Evidence:** `skills/team/SKILL.md:62` reads: *"If it really is dead the
message is inert; if it wakes, it reads the stand-down before it can act."* The
incident that motivated the rule ran the other way: a message queued against a
presumed-dead Manager hours earlier fired on session wake, and the Manager acted
on **that** message — dispatching a Merge-Clerk — in the same minute a
replacement was launched. A stand-down sent at relaunch time would have been
*behind* the already-queued message in that ordering, so the Manager would have
acted first and read the stand-down afterwards.

**Consequence:** Bounded but real. A reader who trusts the sentence stops
worrying about the duplicate-dispatch window, which is the one thing the rule
cannot actually close. The residual exposure is brief — `TaskStop` is unilateral
and needs no cooperation from the resurrected agent, so the design degrades
safely and the end state is correct — but the spec should not claim a guarantee
it does not have. This is a docs-as-spec repo: a sentence asserting unverifiable
harness behaviour is exactly the kind of claim a later reader builds on.

**Acceptance criteria:**

- The sentence no longer asserts delivery ordering. It states what is actually
  true: the stand-down makes the relaunch safe **if** it is read before the
  resurrected Manager acts, and that ordering is not guaranteed.
- The rule names the backstop that does not depend on ordering — `TaskStop` is
  unilateral, so root can stop a duplicate regardless of what either Manager has
  read — and keeps the existing "stop the one with less context" guidance.
- The stand-down itself is **kept**, not removed: it is cheap, it is correct
  when it wins the race, and removing it would leave only the backstop.
- No new anchors or cross-references are introduced (the same discipline the
  developer of this rule applied deliberately, to avoid reintroducing the
  dangling-reference class just fixed).
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `skills/team/SKILL.md`,
`docs/backlog/P3-standdown-ordering-claim-unverifiable.md` (delete).

**Suggested approach:** A hedge, not a redesign — do not reopen the relaunch
guard's design, which is sound. **Sequencing: this touches
`skills/team/SKILL.md`, which `item/owner-consent-escalation` also edits. Do not
dispatch it until that branch has landed.**
