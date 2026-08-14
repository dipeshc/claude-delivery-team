---
by: qa (consistency-sweep)
---

# The research-spike brief never hands the Researcher the charter it is bound by

**What's wrong:** The charter binds a "root-dispatched Researcher" to its rules,
but the only dispatch that creates one — the research-spike brief in
`skills/team/SKILL.md` — spawns a plain `general-purpose` agent and gives it no
path to the charter, so the Researcher never receives the explicit-path commit
floor rule, the capability gate's self-check, or the full item-shape contract.

**Evidence:**

Charter side — the Researcher is a charter-bound role:

- `docs/team-charter.md:3-5` — "This is the single source of the rules every team
  agent obeys — the Manager, the Developers, the Reviewers, the Merge-Clerk, the
  QA, the Engine-Supervisor, and a root-dispatched Researcher."
- `docs/team-charter.md:237` — "4. **Researcher** — backlog item files,
  explicit-path only", under a section that closes at
  `docs/team-charter.md:244-245` with "Every non-merge path is **explicit-path
  only** — `git add <path>`, never `git add -A`." Declared a floor rule at
  `docs/team-charter.md:8-9`.
- `docs/team-charter.md:85-88` — "Judgment roles (Manager, Reviewer, Merge-Clerk,
  QA, Researcher) require a frontier model … **State your model as your first
  action**; if you cannot confidently determine your identity or you are a small
  model, **STOP** and say so plainly" (also `docs/architecture.md:58-60`,
  invariant 4).
- `docs/team-charter.md:271-282` — the item-shape contract: "Every item file
  states: **What's wrong** … **Evidence** … **Consequence** … **Acceptance
  criteria** … **Exact files to change** … **Suggested approach**."

Dispatching side — `skills/team/SKILL.md:186-194` is the entire Researcher
dispatch: "spawn a `general-purpose` agent (background, frontier model) with the
question and this brief — *investigate for real …; adversarially check your own
conclusion before filing …; answer only the question asked …; file one item file
into the profile's backlog location per its conventions (frontmatter
`by: research`) with a one-sentence what's-wanted, exact acceptance criteria, and
`file:line` evidence … end with `FILED {items[], commit, confidence,
open_questions}`*". It names no charter path.

Three concrete gaps:

1. **Explicit-path commit — no path to the agent at all.** The brief says "file
   one item file" and never mentions committing, let alone `git add <path>`.
   Contrast the sibling instruction two paragraphs up, `skills/team/SKILL.md:181-183`:
   "write the item file yourself into the profile's backlog location … and commit
   it explicit-path".
2. **Capability gate — half conveyed.** The brief does say "(background, frontier
   model)", so the model *tier* is requested at spawn time. What is missing is the
   self-check half — "State your model as your first action … STOP" — which exists
   precisely because pinning is unreliable (`docs/team-charter.md:93-94`:
   "DELETE WHEN model frontmatter pinning is reliable — this gate exists only
   because a spawned agent cannot always be pinned to a model").
3. **Item shape — the brief's own list displaces the contract.** The brief
   enumerates three fields (what's-wanted, acceptance criteria, `file:line`
   evidence) and reads as the complete requirement; Consequence, Exact files to
   change, and Suggested approach are absent. In *this* repo a diligent Researcher
   could still chain "per its conventions" → `.claude/project-profile.md:39-40`
   ("the charter's \"Backlog conventions\" section is the convention") → the six
   fields. That chain is not guaranteed generically:
   `docs/project-profile.template.md:41-42` defines `n/a` as meaning "the
   convention is just \"one file per item.\"", which terminates the chain with no
   item shape at all.

The convention for passing the contract to a non-team agent already exists:
`workflows/consistency-sweep.js:153` — "Every item must satisfy the charter's
item-shape contract — a stranger must be able to action it cold."

**Canonical side:** `docs/team-charter.md`. It is the explicitly-normative owning
source for the roles it names, and it names the Researcher; a `general-purpose`
agent does not read it unless the dispatch says so, so the brief in
`skills/team/SKILL.md` is where the obligation is discharged.

**Consequence:** A Researcher spawned from this brief commits its item with
`git add -A`, sweeping unrelated working-tree state into a backlog commit on the
main tree — the exact failure the floor rule exists to prevent. It may also run on
a non-frontier model without ever announcing it, and files items missing three of
the six fields the charter says make an item actionable cold, pushing the
re-derivation cost onto the Developer who picks it up.

**Acceptance criteria:**

- The research-spike brief in `skills/team/SKILL.md` points the spawned agent at
  `${CLAUDE_PLUGIN_ROOT}/docs/team-charter.md` and tells it to read it before
  filing.
- The brief states the explicit-path commit requirement in-line
  (`git add <path>`, never `git add -A`), matching `skills/team/SKILL.md:181-183`.
- The brief carries the capability gate's self-check, not just the frontier-model
  spawn request: state your model first, STOP if small or unidentifiable.
- The brief requires the charter's full six-field item shape rather than its
  current three-field list, or defers to the charter contract by name so the
  shorter list cannot read as complete.
- Verification is a read: the requirements reachable by an agent that has only the
  brief and the profile are the same set as those in
  `docs/team-charter.md:227-246`, `:85-88`, and `:271-282`. (No test suite exists —
  `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `skills/team/SKILL.md` (the research-spike bullet,
lines 186-194).

**Suggested approach:** Lead with the charter pointer — one
`${CLAUDE_PLUGIN_ROOT}/docs/team-charter.md` reference discharges all three gaps
at once and avoids restating charter rules inside a skill (which is the residue
pattern the charter's "Spec prose is timeless" section warns against). Then keep
only the brief-specific additions the charter does not cover (`by: research`, the
`FILED` / `NO-ITEM` payloads, "answer only the question asked"). Do not duplicate
the six field names into the skill; name the contract and let the charter own it.
