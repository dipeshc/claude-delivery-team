---
by: qa (consistency-sweep)
---

# `docs/backlog/README.md` re-legislates the charter's `blocked/` rule and has already dropped a clause

**What's wrong:** `docs/backlog/README.md` restates the `blocked/` and `deferred/`
definitions that the charter's "Backlog conventions" section owns, and the copy
has already lost the load-bearing half of the `blocked/` rule.

**Evidence:**

Copy:

- `docs/backlog/README.md:8-10` — "- `blocked/` — items waiting on an owner
  decision or on environment (devices, credentials, live services).
  / - `deferred/` — items deliberately out of scope, never drifted into."

Owner:

- `docs/team-charter.md:254-256` — "- `blocked/` — blocked on an owner decision or
  on environment (devices, credentials, live services). **Waits regardless of
  priority.** / - `deferred/` — deliberately out of scope; **never** drifted
  into."

The README already knows where the owner is, and points at it two lines later:

- `docs/backlog/README.md:12-13` — "For the full item shape and lifecycle rules,
  see the team charter's [\"Backlog conventions\"](../team-charter.md#backlog-conventions)
  section."

**Canonical side:** `docs/team-charter.md`'s "Backlog conventions". It is the
explicitly-normative owning section, and `.claude/project-profile.md:39-40` defers
to it by name — "**Conventions doc:** n/a — the charter's \"Backlog conventions\"
section is the convention". This is the "same rule legislated in two places where
one should own it and the other point at it" residue class
(`skills/consistency-check/SKILL.md:85-86`).

**Consequence:** The drift is not hypothetical — it has already happened. The
README's `blocked/` gloss silently drops "Waits regardless of priority", which is
the operative half of the rule: a P0 in `blocked/` still waits. An agent that
reads the README (the nearest doc to the backlog it is working in) and not the
charter can conclude a blocked P0 should be picked up on priority grounds. Every
future edit to either definition reopens the same gap.

**Acceptance criteria:**

- The two restated bullets at `docs/backlog/README.md:8-10` are deleted.
- The pointer at `docs/backlog/README.md:12-13` is widened so it covers the
  subdirectory meanings as well as item shape and lifecycle, and the sentence
  introducing the bullets (`docs/backlog/README.md:5-6`, "Two subdirectories carry
  a fixed meaning:") is adjusted so the file still reads cleanly.
- `docs/team-charter.md:254-256` is unchanged — it is the canonical side.
- The anchor `../team-charter.md#backlog-conventions` still resolves after the
  edit.
- Verification is a read: `blocked/` and `deferred/` are defined in exactly one
  place in the repo. (No test suite exists — `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/backlog/README.md` (lines 5-13).

**Suggested approach:** Delete rather than sync. The README's own stated purpose —
"This file is documentation, not a work item" (`docs/backlog/README.md:3`) — plus
the location/priority-naming facts it states are enough; the pointer already
carries the rules. Keeping a corrected copy would just reset the drift clock.
