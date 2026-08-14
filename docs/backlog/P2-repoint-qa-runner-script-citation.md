---
by: qa (consistency-sweep)
---

# `agents/qa.md` cites the wrong profile section for the watch-loop runner script

**What's wrong:** `agents/qa.md` tells the QA that the runner script's path is
"the QA entry in the profile's **Sanctioned direct-write paths**", but the profile
section that actually declares that path is **QA watch loop → Runner script** —
a section `agents/qa.md` never names, including in the list of profile sections it
tells QA to read.

**Evidence:**

Mis-citation:

- `agents/qa.md:65-67` — "The mechanical loop lives in a **shell script the
  project supplies**, not in you. Its path is the QA entry in the profile's
  **Sanctioned direct-write paths**; the suite it runs is the profile's **Quality
  gate → Full**."

What that section actually is:

- `docs/project-profile.template.md:108-113` — "## Sanctioned direct-write paths
  / Paths the root/orchestrating instance may commit to directly, outside the
  normal review pipeline (everything else lands via the pipeline). / - e.g.
  `docs/backlog/`, `scripts/qa/`, `.claude/`" — a write-permission grant listing
  *directories*, not the script.

The owning section:

- `docs/project-profile.template.md:66-72` — "## QA watch loop / - **Runner
  script:** the project's own watch-and-test script (e.g.
  `scripts/qa/watch-and-test.sh`) — a shell loop that pulls each new
  default-branch tip, runs the **full** gate, and wakes the QA agent only on a new
  failure or a green heartbeat."

The correct citation already exists elsewhere:

- `agents/manager.md:366` — "QA watch loop section names the loop's runner
  script) — a plain read, no wait."

Compounding omission — the section list QA is told to read:

- `agents/qa.md:31-37` — "Read `<repo>/.claude/project-profile.md` before anything
  else. It is the only place this project's specifics live: the **Quality gate**
  (scoped vs full), the **Backlog** location and conventions, the **Repo-wide
  invariant guards**, **Cross-surface parity**, the **Sanctioned direct-write
  paths** (which name your QA loop's own path), the **Worktree layout**, and the
  **Verification environment**." — **QA watch loop**, the one profile section
  written for this agent, is absent. (`agents/qa.md` does reference the loop
  indirectly — "riding the project's QA watch loop" at line 5, "If the profile
  names no QA runner" at line 99 — but never by section name, so a QA following
  the read list literally never opens it.)

**Canonical side:** the template's **QA watch loop** section. It is the owning
section — it declares the runner script and the known-failures file — and
`agents/manager.md:366` already cites it correctly. `agents/qa.md:65-67` is an
incidental restatement aimed at the wrong section: Sanctioned direct-write paths
grants a *write scope*; it does not declare the script's path.

**Consequence:** A QA agent following its own file looks for the runner script in
a section that lists directories and permissions, not scripts, and in a repo whose
"Sanctioned direct-write paths" is `n/a` (`docs/project-profile.template.md:114`)
concludes no loop exists when the profile in fact declares one. The section list
at `agents/qa.md:31-37` reinforces the miss by never sending QA to the section
that has the answer.

**Not part of this defect** (checked and dismissed): `agents/qa.md:80-82` — "a
**known-failures file** kept beside the script" — does not contradict
`docs/project-profile.template.md:73-75`. The template at line 71-72 explicitly
assigns the loop's *design* to the QA agent definition, and this line is a design
default inside a numbered list describing how the loop works, not a competing
declaration of where the profile records it. It could gain a pointer for
consistency; it is not a mis-citation.

**Acceptance criteria:**

- `agents/qa.md:65-67` cites the profile's **QA watch loop → Runner script** for
  the script's path.
- **QA watch loop** is added to the profile-section list at `agents/qa.md:31-37`.
- **Sanctioned direct-write paths** remains cited where it is genuinely the
  write-permission authority — `agents/qa.md:34` (with its parenthetical corrected
  if it now over-claims) and `agents/qa.md:100`.
- Verification is a read: every profile section `agents/qa.md` names resolves to a
  real heading in `docs/project-profile.template.md`, and every fact `agents/qa.md`
  attributes to a section is actually stated in that section. (No test suite
  exists — `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/qa.md` (lines 31-37 and 65-67).

**Suggested approach:** Two small edits, mirroring the wording
`agents/manager.md:366` already uses. While in the section list, check the
parenthetical "(which name your QA loop's own path)" — after the repoint, what
Sanctioned direct-write paths names is where QA may *write* (its logs, status
file, and bootstrapped script), not where the runner script is declared; phrase it
so the two roles stay distinct.
