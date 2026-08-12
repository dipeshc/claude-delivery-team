# Architecture — the delivery-team plugin

This is the **specification and entry doc for this repository itself**. The
plugin it describes tells other projects to treat their docs as the spec; this
repo holds itself to the same rule. Where this document and an agent or skill
file disagree, this document is right and the agent/skill file is a bug.

## What this repo is

A [Claude Code plugin](https://code.claude.com/docs/en/plugins.md) implementing
an autonomous, docs-driven software delivery team. The repo is prose-only:
its "code" is the agent definitions under `agents/`, the skills under
`skills/`, and the plugin manifest under `.claude-plugin/`. Its spec is the
documents under `docs/`.

## Document roles

| Document | Role |
|---|---|
| `docs/architecture.md` | This file — the design spec and entry doc for the repo. |
| `docs/team-charter.md` | The shared behavioural spec every team agent implements. Agent files reference it rather than restating it; a role-specific twist lives in the role's own file. |
| `docs/project-profile.template.md` | A shipped artifact for consuming projects. Its section contents describe *consumers*, not this repo — the consistency check audits its structure (references, terminology), never treats its example values as claims about this repo. |
| `docs/backlog/` | This repo's own backlog: one file per item, priority in the filename, deleted by the merging commit. |
| `README.md` | The public front door. It summarizes this spec and must never contradict it. |

## Components

| File | Role |
|---|---|
| `agents/manager.md` | Singleton orchestrator: triages the backlog, dispatches Developers, routes reviews and merges, supervises liveness and scope. Never does the work itself. |
| `agents/developer.md` | Implements exactly one backlog item (or one assigned related group) per dispatch, in its own worktree, as one commit. Never merges. |
| `agents/reviewer.md` | Judges correctness and standards, re-verifies with the reverse-dependency slice. Emits verdicts only; carries no write tools. One instance by default. |
| `agents/merge-clerk.md` | Singleton, the only writer of code to the default branch: lands approved branches ff-only, rebasing trivially-behind branches itself via cherry-pick + range-diff. |
| `agents/qa.md` | Singleton guardian: rides the project's watch loop running the full gate on every default-branch tip, files regressions, runs the consistency cadence. Never fixes anything. |
| `agents/engine-supervisor.md` | Opt-in, small-model supervisor of an external engine. The **only** file in the repo that describes external-engine mechanics. |
| `agents/exploration.md` | Root-owned specialist for feature-spike waves, outside the pipeline. |
| `agents/ui-inspector.md` | Root-owned specialist for visual QA of a project's rendered UI. |
| `skills/team/SKILL.md` | How the root instance runs a delivery: lane routing, spawning the Manager, status heartbeat, watchdog, recovery. |
| `skills/team/exploration-and-promotion.md` | How exploration runs and how a kept spike is promoted through the pipeline. |
| `skills/consistency-check/SKILL.md` | The two-phase spec audit: spec-vs-spec, then spec-vs-code. Files findings; fixes nothing. |

## Invariants

These bind every component; a component contradicting one is defective.

1. **Docs drive design.** For a project that declares docs-as-spec, the docs
   are the source of truth: a doc-vs-code mismatch is a code bug, spec edits
   ship in the same commit as the behaviour they describe, and agents sharpen
   docs but never re-decide them (decisions are the owner's).
2. **Single writer for code.** Only the Merge-Clerk writes code to a project's
   main working tree, via `merge --ff-only`, one commit per item, linear
   history. A short list of scoped housekeeping writers (charter,
   "Scoped writers") is the only exception, and none of them edit code.
3. **Risk-routed lanes.** Small, single-area changes take the direct lane; the
   full pipeline is reserved for work whose risk earns it. Verification effort
   scales with risk, not habit.
4. **Capability gate.** Judgment roles (Manager, Reviewer, Merge-Clerk, QA,
   Researcher) require a frontier model and refuse to run otherwise. The
   Developer and the Engine-Supervisor are exempt.
5. **External engines are opt-in and contained.** Engine mechanics live only in
   `agents/engine-supervisor.md`; every other file may at most *point* there.
   Engines exist for a run only when the consuming project's profile declares
   one; engine output is always an untrusted contributor diff that passes the
   same review and gates as any other change.
6. **Project-agnostic core.** The charter, agents, and skills are identical for
   every project and contain nothing project-specific — no references to any
   particular consuming project, product, or domain. Everything
   project-specific lives in the consuming repo's `.claude/project-profile.md`,
   whose shape is defined by the template.
7. **State lives in git.** Backlog items are files; work-in-flight is branches
   and worktrees; an item is done when the merging commit deletes its file. Any
   agent's death is recoverable from git alone.
8. **Verification is honest.** Agents report the exact commands they ran and
   their outcomes; a verification not run is reported as not run. Sunset-tagged
   workarounds (`DELETE WHEN …`) mark rules that exist only to route around a
   current harness limitation, so they are deleted when the cause is gone.

## Changing this repo

This repo consumes its own process: `.claude/project-profile.md` declares
docs-as-spec with `docs/` as the docs root, and work is filed into
`docs/backlog/`. A change that alters behaviour specified here — a role's
responsibilities, an invariant, a message flow — updates this document (and the
charter where the rule is shared) in the same commit. The `consistency-check`
skill is the audit: phase 1 checks these documents against each other, phase 2
checks the agent and skill definitions against them.
