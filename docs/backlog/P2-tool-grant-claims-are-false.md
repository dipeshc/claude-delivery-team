---
by: owner (shakedown finding F4)
---

# Reviewer/Merge-Clerk specs claim tool grants the runtime does not honour

**What's wrong:** Two documents assert that Reviewers and the Merge-Clerk hold
no `Write`/`Edit` tools. Under the plugin loader they are registered **with**
`Write, Edit` appended, so the claim is false and the spot-check built on it
reports a leak on every healthy run.

**Evidence:** `agents/reviewer.md:201` states "you carry no Write/Edit tools —
if you ever feel the need to, that is a signal you are out of role", and
`skills/team/SKILL.md:83-87` tells the root to spot-check that "Reviewers and
the Merge-Clerk carry none at all; a widened grant is a scope leak". The
installed frontmatter is correct — `agents/reviewer.md` and
`agents/merge-clerk.md` both declare `tools: Bash, Read, Grep, Glob` — but the
session's agent registry lists both with `Write, Edit` added. The widening is
harness-side, not a repo defect; the defect is that our spec asserts otherwise.

**Consequence:** A spec that is verifiably false teaches readers to discount it,
and the root's spot-check fires a false "scope leak" on every run — noise that
buries a real one. Containment currently rests on the prose prohibition alone,
which no document says out loud.

**Acceptance criteria:**

- `agents/reviewer.md` states the boundary as a **behavioural prohibition**
  ("you never write — no Write/Edit, no edits anywhere") rather than a claim
  about which tools are granted, and notes that a tool being present is not
  permission to use it.
- `agents/merge-clerk.md`'s Boundaries section carries the same framing.
- `skills/team/SKILL.md`'s tool-grant spot-check is rewritten to check the
  agent files' declared `tools:` frontmatter (which the repo controls) and to
  say explicitly that the runtime may add grants beyond what is declared, so an
  added grant is not by itself a finding.
- `docs/architecture.md` — if its invariants imply tool-level containment,
  reword to behavioural containment. Verify while editing; change nothing if it
  does not.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/reviewer.md`, `agents/merge-clerk.md`,
`skills/team/SKILL.md`, possibly `docs/architecture.md`,
`docs/backlog/P2-tool-grant-claims-are-false.md` (delete).

**Suggested approach:** Sharpening only — the intent (these roles never write)
is unchanged; only the false mechanism claim goes. Related to
`P3-docs-only-review-worktree`, which also edits `agents/reviewer.md`; group
them if dispatched together.
