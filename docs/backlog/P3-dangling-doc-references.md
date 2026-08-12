---
by: owner (run finding — rediscovered independently by two reviews)
---

# Two documents cite things that do not exist

**What's wrong:** Two references point at targets that are not there — one link
anchor that cannot resolve, and two citations of a guard no document describes.
Both pre-date this run and were correctly left alone by items whose scope
excluded them, but each has now been rediscovered independently by more than one
review, which is the signal to fix rather than re-find.

**Evidence:**

- `docs/team-charter.md:14` — `[Project profile](#project-profile)` does not
  match the heading it targets, `## Project profile — read it first`, whose slug
  is `project-profile--read-it-first`. The link silently goes nowhere.
- `agents/reviewer.md:96` and `agents/merge-clerk.md:140` both instruct the
  reader to install only when the lockfile changed, "the same guard the QA watch
  loop uses" — but `agents/qa.md` never describes any such guard. The citation
  points at a description that does not exist, so a reader who follows it to
  learn the mechanism finds nothing.

**Consequence:** Low individual impact, but this is a docs-as-spec repo whose
architecture doc makes reference integrity a stated property. A spec that cites
absent things teaches readers to skim past its citations, which is the habit
that lets a load-bearing broken reference through later. Two reviews spending
budget rediscovering the same defects is itself waste.

**Acceptance criteria:**

- The anchor at `docs/team-charter.md:14` resolves — either by correcting the
  fragment to the heading's real slug or by adjusting the heading; say which and
  why.
- The install-guard citation resolves: either `agents/qa.md` gains the brief
  description of the lockfile-hash-and-marker guard that the other two files
  claim it holds, or the two citations stop referring to it and state the guard
  directly. Prefer describing it once in `agents/qa.md` — that is what the
  citations assume.
- **Preserve a deliberate asymmetry while fixing this:** `agents/reviewer.md`'s
  install guard correctly has no docs-only clause (a docs-only review now
  creates no worktree, so it never reaches installation), while
  `agents/merge-clerk.md`'s correctly keeps one. Do **not** "harmonise" the two
  — that would reintroduce a defect fixed in `7f2a9de`.
- Sweep the changed files for other unresolvable relative links and `#anchor`
  fragments while you are in them, and fix any found; report the sweep's scope.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification, including actually resolving each touched link.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, `agents/qa.md`,
possibly `agents/reviewer.md` and `agents/merge-clerk.md`,
`docs/backlog/P3-dangling-doc-references.md` (delete).

**Suggested approach:** Mechanical rot, batched into one item deliberately —
dozens of one-line items help nobody. Shares `docs/team-charter.md` with the
other charter items.
