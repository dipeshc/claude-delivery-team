---
by: qa (consistency-sweep)
---

# `agents/ui-inspector.md` writes backlog files to the main tree as an unsanctioned sixth writer

**What's wrong:** The charter's scoped-writers floor rule enumerates exactly five
roles permitted to write to a project's main working tree, but
`agents/ui-inspector.md` instructs a sixth role to create backlog item files
there — with no worktree discipline, no explicit-path commit rule, and in fact no
commit instruction at all.

**Evidence:**

Charter side (owning, normative, declared a floor rule):

- `docs/team-charter.md:227-229` — "The main working tree is written by a small
  set of **scoped** paths. This is not \"single writer\" — it is
  single-writer-for-code plus a few scoped housekeeping writers:" followed by the
  numbered list 1 Merge-Clerk, 2 QA, 3 Root, 4 Researcher, 5 Manager
  (`docs/team-charter.md:231-241`), closing at `docs/team-charter.md:245-246`:
  "Every non-merge path is **explicit-path only** — `git add <path>`, never
  `git add -A`. Anyone else writing to the main tree is off the rails."
- `docs/team-charter.md:8-9` — "the invariants below (writers, verification) are
  floor rules no role overrides", so the agent-file-wins tie-break at
  `docs/team-charter.md:7-8` does not apply.
- `docs/architecture.md:52-54` — invariant 2: "A short list of scoped housekeeping
  writers (charter, \"Scoped writers\") is the only exception."

Implementing side:

- `agents/ui-inspector.md:221-224` — "For each confirmed finding, create an item
  file at the profile's **Backlog** location following its naming convention
  (priority in the filename — `P2` for a real visual defect users will hit, `P3`
  for polish; whatever frontmatter the project's backlog conventions require, plus
  `by: ui-inspector`)."
- `agents/ui-inspector.md` contains no workspace-discipline section anywhere.
  Contrast the sibling root-dispatched agent, `agents/exploration.md:125-130`:
  "**Workspace discipline.** The main working tree keeps the default branch
  checked out at all times … only the Merge-Clerk writes code there. Never
  `git checkout` your exploration branch in the main tree. Resolve the main tree
  first, then create your own worktree per the profile's **Worktree layout** and
  do all work there". With no such instruction, the Backlog location the
  ui-inspector writes *is* the main working tree.
- No commit instruction of any kind accompanies the create. Contrast
  `skills/team/SKILL.md:180-183`, which tells Root to "commit it explicit-path".

**Canonical side:** `docs/team-charter.md`'s "Scoped writers to the main working
tree". It is the explicitly-normative owning section, it names itself a floor rule
no role-specific agent file overrides, and architecture invariant 2 restates it as
the only exception. Root-dispatch does not confer the grant on its own: the
Researcher is also root-dispatched and still needed its own numbered entry at
`docs/team-charter.md:237`. So the ui-inspector's write is unsanctioned as
written, and the fix belongs in `agents/ui-inspector.md` — *adding* a sixth writer
to the charter would be a spec re-decision and therefore an owner call
(`.claude/project-profile.md:45-50`).

**Consequence:** Two distinct failures. (1) An unsanctioned writer mutates the
main working tree, which every other role is told to treat as the Merge-Clerk's
serialized surface — a dirty tree here can collide with a fast-forward merge in
flight. (2) Because the agent is never told to commit, the item file is created
but left uncommitted, and the Manager detects new backlog work by comparing
`git log --no-show-signature -1 --format=%H -- <backlog-path>` against its
last-processed SHA (`agents/manager.md:80-82`). An uncommitted file produces no
new SHA, so the finding is *never dispatched* while the main tree is left dirty —
the UI defect is silently lost.

**Acceptance criteria:**

- `agents/ui-inspector.md` no longer instructs an unsanctioned write to the main
  working tree: it either files into a path the charter sanctions, or returns its
  findings as its turn-ending payload for the root/QA to file.
- Whichever route is taken, the commit question is settled explicitly — if the
  ui-inspector commits, the instruction says `git add <path>` and never
  `git add -A` (`docs/team-charter.md:244-245`); if it does not commit, the file
  says who does, so the Manager's backlog-SHA poll can see it.
- `agents/ui-inspector.md` gains a workspace-discipline statement equivalent to
  `agents/exploration.md:125-130`, or an explicit statement that it writes nothing
  to disk.
- Verification is a read: after the change, the roles named in
  `docs/team-charter.md:231-241` and the set of agent files that write to a
  project's main tree are the same set. (No test suite exists —
  `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/ui-inspector.md` (section "5. Report", around
lines 219-235).

**Suggested approach:** Prefer the return-findings route — it needs no new charter
entry, keeps the floor rule intact, and matches how a read-only inspection agent
should behave. If filing directly is genuinely wanted, that is a charter change
(a sixth numbered scoped writer, with its explicit-path commit rule) and therefore
an owner decision, not something to settle inside the agent file. Do not solve it
by adding a worktree to the ui-inspector: a backlog item written into a worktree
never reaches the Manager's poll either.
