---
by: owner
---

# Measure what a flat agent-teams topology would cost this framework

**What's wrong:** The framework's eventual migration to peer messaging is
blocked on an unmeasured assumption. Agent teams do not support nested teams,
so the current root → Manager → workers hierarchy would have to flatten — and
nobody has established what that costs. Deciding at general availability
without the measurement means deciding blind, under time pressure.

**Evidence:** `skills/team/SKILL.md` makes root a thin spawner-and-watchdog and
`agents/manager.md` makes the Manager a spawner of Developers, Reviewers, the
Merge-Clerk and the QA — two levels of spawning. `agents/manager.md`'s WIP
model also assumes INACTIVE children cost nothing, which holds because they are
suspended subagents; teammates are independent sessions, so that assumption is
untested under a flat topology. The relay duty this would delete is real: one
run lost twelve minutes to a landed commit sitting unreported in root.

**Consequence:** Either the framework migrates on assumption and discovers the
economics afterwards, or it postpones indefinitely because the unknown never
shrinks. Both are avoidable for the cost of one throwaway experiment.

**Acceptance criteria:**

- A **throwaway repository** — never this one — runs a minimal delivery cycle
  (one item: implement, review, land) under a flat topology, and the findings
  are written up as a decision input. The experiment is the deliverable; no
  framework file changes under this item.
- The write-up answers, with observation rather than reasoning: who leads when
  the hierarchy flattens and what happens to root's watchdog role; whether an
  idle teammate is actually free or holds a session's worth of cost; whether a
  child's payload reaches its spawner without a relay hop; what recovery looks
  like when a teammate dies, given that reconstruction from git is the current
  guarantee; and whether the merge serialization survives without the singleton
  being enforced by spawn topology.
- It states plainly what it could **not** determine, and what would settle it.
- The write-up lives outside this repo (owner's notes or the scratchpad) or, if
  kept here, as a decision record — not as prose in the charter or an agent.
  Nothing in the framework may come to depend on agent teams.
- Blocked while agent teams remain experimental: if the feature is still behind
  a flag and unavailable, move this item to the backlog's `blocked/`
  subdirectory with that reason rather than approximating the experiment.
- This item file is deleted by the merging commit.

**Exact files to change:** none in this repo beyond deleting
`docs/backlog/P3-measure-the-agent-teams-flatten-cost.md`.

**Suggested approach:** Cheapest experiment that answers the questions; resist
building a parallel framework. Pairs with
`P3-name-the-relay-sunset-trigger` — that item names the trigger, this one
measures whether tripping it is affordable.

---

⛔ **Blocked on environment (2026-08-14).** Claude Code agent teams are still behind
the experimental flag `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`, which is unset in
this environment (verified: absent from the process environment and from
settings). The flat-topology experiment cannot be run, and this item's own
acceptance criteria direct that it move here with that reason rather than be
approximated. Unblocks when teams are available to run a real delivery cycle
under a flat topology.
