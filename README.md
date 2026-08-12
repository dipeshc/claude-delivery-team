# claude-delivery-team

An autonomous, **docs-driven** software delivery team built on
[Claude Code](https://claude.com/claude-code) subagents. A **Manager** agent
works a prioritized backlog and dispatches **Developers**, a **Reviewer**, a
singleton **Merge-Clerk**, and a **QA** guardian — implementing in isolated git
worktrees, cross-reviewing each other's work, enforcing quality gates, and
landing signed, fast-forward-only commits with minimal human intervention. It
has produced thousands of commits end-to-end, under human direction but not
human keystrokes.

## Docs drive design

The framework's central bet is that **documentation is the specification** and
everything else follows from it:

- **Docs are the source of truth.** Where a project declares docs-as-spec, a
  doc-vs-code mismatch means the *code* is wrong. Agents may *sharpen* a doc
  (make an already-decided thing precise) but never *re-decide* one — decisions
  belong to the owner.
- **The spec edit ships with the change.** A change that alters observable
  behaviour lands its doc amendment in the same commit; a missing spec edit is
  a review defect, exactly like a missing test.
- **The backlog is files in the repo.** One file per item, priority in the
  filename, done when the merging commit deletes it — work state lives in git,
  not in anyone's memory.
- **The spec is audited.** The `consistency-check` skill runs two phases —
  spec-vs-spec (does the documentation agree with itself?) and spec-vs-code
  (does the code obey it?) — on demand and on QA's cadence, filing every
  confirmed delta as a backlog item.

## How it works

```mermaid
flowchart TD
    U[You: file backlog items] --> R[Root / team skill]
    R -->|team lane| M[Manager<br/>singleton, frontier model]
    R -->|direct lane| D0[Small change: implement directly,<br/>scoped gate, ff-merge]
    M --> DEV1[Developer<br/>worktree: item/a]
    M --> DEV2[Developer<br/>worktree: item/b]
    DEV1 -->|READY-FOR-REVIEW| REV[Reviewer<br/>one by default, project-scalable]
    DEV2 -->|READY-FOR-REVIEW| REV
    REV -->|APPROVED| MC[Merge-Clerk<br/>singleton, ff-only]
    REV -->|CHANGES-REQUESTED| DEV1
    MC -->|MERGED| MAIN[(default branch<br/>linear history)]
    QA[QA watch loop<br/>full gate on every tip] --> MAIN
    QA -->|REGRESSION → backlog item| M
```

The lifecycle of a work item:

1. **Backlog** — one file per item in the repo's backlog directory, priority in
   the filename (`P0`–`P3`). Filed by you, by a research spike, or by QA.
2. **Lane routing at intake** — low-risk, single-area changes take the *direct
   lane* (no pipeline ceremony); auth/security, data-model, cross-cutting, or
   multi-phase work takes the *team lane*.
3. **Implement** — a Developer gets its own git worktree on `item/<slug>`,
   produces the change as one commit, and rebases onto the default branch.
4. **Review** — an independent Reviewer judges correctness *and* re-verifies by
   running the quality gate itself. One Reviewer by default; a project may scale
   to more in its project profile if its throughput genuinely needs it.
   Reviewers carry no write access at all.
5. **Land** — the Merge-Clerk, the only writer of code to the main tree,
   fast-forward-merges the approved branch. The merging commit deletes the
   item's backlog file — merging is what marks work done.
6. **Guard** — QA pulls every new default-branch tip into its own worktree and
   runs the full quality gate, filing regressions back into the backlog.

## Design decisions that matter

- **Single-writer merge point.** Only the Merge-Clerk writes code to the main
  working tree, ff-only, one commit per item. History stays linear and every
  commit traces to a reviewed item.
- **Risk-routed lanes.** Verification effort scales with actual risk, not habit.
  A typo fix doesn't pay for a five-agent pipeline; a persistence change does.
- **Capability gating.** Judgment roles (Manager, Reviewer, Merge-Clerk, QA)
  self-check that they're running on a frontier model and refuse to proceed
  otherwise. Implementation can run on mid-tier models.
- **External engines are contained.** A project may opt in to routing
  implementation through an external engine (a separate CLI on its own quota).
  Everything engine-related lives in one dedicated `engine-supervisor` agent —
  its output is treated as an untrusted contributor diff and reviewed like any
  other change, and runs without an engine configured never see, mention, or
  pay tokens for any of it.
- **Verification discipline.** "Should pass" is not a result. Agents report the
  exact commands they ran and their outcomes; repo-wide invariant guards run
  whenever triggered, even when a scoped gate wouldn't catch them.
- **Crash-safe by construction.** State lives in git (branches, worktrees,
  backlog files), not in agent memory. A dead Manager is relaunched and
  reconstructs everything from `git worktree list` and the backlog.
- **Project-agnostic core.** The charter and agents are identical for every
  project. Everything project-specific — quality-gate commands, backlog
  location, invariant guards, parity rules — lives in that repo's
  `.claude/project-profile.md`.

## Repository layout

This repo is a [Claude Code plugin](https://code.claude.com/docs/en/plugins.md):

| Path | Purpose |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | Single-plugin marketplace manifest, so the repo installs directly from GitHub |
| `agents/` | Agent definitions: `manager`, `developer`, `reviewer`, `merge-clerk`, `qa`, the opt-in `engine-supervisor`, plus `exploration` and `ui-inspector` specialists |
| `skills/team/` | The team skill — how the root instance spawns, stewards, and watchdogs a run |
| `skills/consistency-check/` | Spec-vs-spec and spec-vs-code audit skill — run on request (owner or root) or by QA on cadence |
| `docs/architecture.md` | The spec for this repo itself — what each component is and the invariants that bind them |
| `docs/team-charter.md` | Shared rules every agent obeys: lanes, writers, signing, verification, message schemas |
| `docs/project-profile.template.md` | Template for the per-repo `.claude/project-profile.md` |
| `docs/backlog/` | This repo's own backlog (it dogfoods its own process) |

This repo practices what it preaches: `docs/` is its specification,
`.claude/project-profile.md` declares docs-as-spec, and changes to the agents
and skills are measured against the docs — not the other way round.

## Install

In Claude Code (the repo is its own single-plugin marketplace):

```
/plugin marketplace add dipeshc/claude-delivery-team
/plugin install delivery-team@claude-delivery-team
```

Then, per project:

1. Copy `docs/project-profile.template.md` to `<repo>/.claude/project-profile.md`
   and fill it in (quality-gate commands, backlog location, layout — and, if
   your docs are the spec, say so in its Specification source of truth section).
2. Create the backlog directory and file an item or two.
3. Run `/delivery-team:team`.

The Manager takes it from there and a status heartbeat keeps you posted; you
stay in the loop for owner decisions, blocked items, and anything marked
`MERGE-BLOCKED`.

## Requirements

- Claude Code with subagent support (the `Agent` tool and `.claude/agents`).
- Judgment roles want a frontier model (e.g. Opus-class or above); developers
  can run on mid-tier models.
- A git repo. The framework leans hard on worktrees, ff-only merges, and
  commit signing (with a declared, tracked fallback when the signing agent is
  unreachable).

## License

MIT
