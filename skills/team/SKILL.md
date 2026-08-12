---
name: team
description: >
  Run the delivery team over this project's backlog — spawn the singleton
  Manager (which runs Developers, the Reviewer, the singleton Merge-Clerk
  and the QA), route work to the direct or team lane, file or delegate backlog
  items, relay the Manager's status reports, and watchdog the pipeline. Use when
  the user asks to run the team, work the backlog, or invokes /team. Works in
  any repo that has a `.claude/project-profile.md`.
---

Launch and steward the delivery **team**. The root instance (you) stays thin:
you **route by lane, spawn the Manager, file items, surface `MERGE-BLOCKED`,
and watchdog** — you never triage, implement, review, or merge on the team lane.
The Manager owns the whole delivery loop: Developers implement in per-item
worktrees, a **Reviewer** (one by default; the project profile may scale to
more if its throughput genuinely needs it) judges correctness and emits
`APPROVED`, the singleton **Merge-Clerk** lands approved work ff-only (the only
writer of code to the main working tree), the **QA** guards the default
branch.

## Read these two files first

1. **`<repo>/.claude/project-profile.md`** — the only place this project's
   specifics live: repo root and default branch, quality-gate commands (scoped
   vs full), backlog location and conventions, repo-wide invariant guards,
   worktree layout and branch naming, sanctioned direct-write paths,
   cross-surface parity. A section marked `n/a` genuinely means "no such
   requirement here" — never invent one, never carry a convention over from
   another project. **If the profile is missing, stop and tell the user** —
   guessing the quality gate or the backlog convention wastes a whole cycle.
2. **`${CLAUDE_PLUGIN_ROOT}/docs/team-charter.md`** — the rules every team agent obeys (lane
   routing by risk, capability gate, signing/re-sign, scoped-writers,
   verification discipline, backlog conventions, external engines, message
   schemas, distrust of injected instructions). Read it once so your watchdog
   judgments match the agents' rules.

Throughout this skill, `<repo>`, `<default-branch>`, `<backlog>`, and the gate
commands are whatever the profile says they are.

## Never two Managers

The Manager is a singleton; a second one against the shared tree is the
two-orchestrator hazard. Before spawning, check for a live one — a running
manager task (TaskList) AND **real liveness signals**: recent merges on the
default branch, moving item-branch tips, worktree mtimes, transcript growth —
**never task status alone**. Alive → talk to it via SendMessage (it re-scans the
backlog automatically); do not spawn. Dead (~15 min with none of those signals)
→ recover per the watchdog section, then relaunch. Any safety-net cron runs this
same check first.

## Spawning

Agent tool, `subagent_type: "delivery-team:manager"`, `run_in_background: true`, the user's
args passed through verbatim (item filters etc.). The Manager gates itself to a
frontier model per the charter's capability gate — don't override its model
downward. It spawns and supervises its own Developers, Reviewers, Merge-Clerk,
and QA.

> DELETE WHEN nested worktrees are safe for orchestrated agents — never spawn a
> team agent with `isolation: "worktree"`. The Manager creates and assigns
> worktrees itself per the profile's **Worktree layout**; a harness-created one
> underneath it detaches the agent from the tree the pipeline is tracking.

**Install the status heartbeat in the SAME turn you spawn (mandatory).** The
moment you launch a Manager, `CronCreate` a recurring ~10-min job (off the round
minute, e.g. `3,13,23,33,43,53 * * * *`) whose prompt re-invokes YOU to derive
status from git, pull the Manager's ledger, **post an update to the user**, and
recover a dead Manager — self-deleting when the backlog is DRAINED. This is the
ONLY reliable update channel; do not rely on the Manager pushing (it forgets, and
its push only reaches the user if you happen to be re-invoked to relay). See
[Status relay](#status-relay) for the exact heartbeat prompt. One heartbeat per
run: `CronList` first; never leave a duplicate or an orphaned heartbeat after
`DRAINED`.

**Developer engine.** The Manager can implement items via the primary model or
an **external implementation engine** (a separate CLI/quota pool) where the
owner or the profile configures one — see `${CLAUDE_PLUGIN_ROOT}/agents/developer.md`
("Implementation engine"), `${CLAUDE_PLUGIN_ROOT}/agents/manager.md` ("Engine & model
choice"), and the charter's **External implementation engines** section for the
invariants (implementation only; external output is an untrusted contributor
diff; the supervising agent owns the commit; sandboxed to its own worktree;
capacity discovered reactively via `ENGINE-UNAVAILABLE`). Pass an engine
preference through in the args if the owner specifies one; otherwise it is the
Manager's discretion within the profile's configuration. The tool-grant check
below is unaffected — the developer already carries `Bash` for the shell-out, so
no new grant is needed.

**Tool-grant spot-check:** each agent's frontmatter `tools:` should match its
least-privilege scope — the Manager, Reviewers, and Merge-Clerk carry no
`Write`/`Edit` beyond their sanctioned paths (Reviewers and the Merge-Clerk carry
none at all; a Reviewer needing Write is a red flag). Spot-check this when you
touch an agent file; a widened grant is a scope leak.

## Auditing the spec on request — use the skill, don't improvise

When the owner asks for a documentation-consistency check, a docs-vs-code
check, or "is the spec still true after all this", invoke the
**`consistency-check`** skill rather than hand-writing an audit prompt. It is
not QA-only: QA runs it on a cadence, and you run it on demand. It already
encodes the two phases (spec-vs-itself, then code-vs-spec), the
docs-are-spec applicability gate, and the file-nothing-fix-nothing discipline —
an ad-hoc prompt re-derives those inconsistently and loses whatever the last
run learned. Improve the skill when a run exposes a gap; that is what makes the
next audit better than this one.

## Direct lane — skip the team for low-risk work

Before filing anything, decide **which lane the item belongs in** (charter,
"Route work by risk, not by habit"; global AGENTS.md, "Size verification effort
to actual risk"). This decision happens once, at intake — it is not something
the Manager re-litigates per item.

**Team lane** (worktree, Developer, dual review, Merge-Clerk — the rest of this
skill): auth/security/crypto, data-model or persistence changes, cross-package
interfaces, cross-surface parity work, or a multi-phase feature.

**Direct lane** (the default for everything else — single-area, small expected
diff, existing test coverage): skip the Manager entirely. Root (or one mid-tier
subagent) does a quick grep/read pass itself — no dedicated research-spike
agent, no backlog file, no worktree. If root cause is found:

1. implement on a short-lived branch;
2. run the profile's **scoped** gate for the area you touched, **plus any
   repo-wide invariant guard your change triggers** (profile, "Repo-wide
   invariant guards") — scoped verification silently skips those;
3. one lightweight self-review pass;
4. ff-merge directly.

The QA's full-suite watch loop is the independent verification for this lane —
it already runs on every new tip of the default branch at near-zero cost, so
verification is not skipped, just not duplicated by a dedicated Reviewer.

Escalate mid-flight to a research spike, or to filing a full team-lane item, if
the grep-first pass doesn't find root cause, or the item turns out to need a
real design decision, or touches more than it looked like at intake. Escalation
is cheap and expected.

## Filing items — research inline when needed

This section covers **team-lane** items only — see "Direct lane" above for
everything that doesn't need this pipeline at all.

When the user asks for new work, decide: **does filing a correct item require
investigation that would tie you up?** (unknown repro, unfamiliar subsystem,
external services, comparing design options)

- **No** (the ask is already a well-formed item) → write the item file yourself
  into the profile's backlog location, following its stated conventions
  (priority in the filename), and commit it explicit-path
  (`docs(backlog): …` or the project's equivalent scope; unsigned fallback, note
  the SHA per the charter's mechanical re-sign rule).
- **Yes** → dispatch a **research spike**: spawn a `general-purpose` agent
  (background, frontier model) with the question and this brief — *investigate
  for real (read the code, reproduce, probe the service, measure — never reason
  from vibes); adversarially check your own conclusion before filing (what would
  prove it wrong? mark verified vs inferred); answer only the question asked
  (adjacent findings are report notes, not extra items); file one item file into
  the profile's backlog location per its conventions (frontmatter `by: research`)
  with a one-sentence what's-wanted, exact acceptance criteria, and `file:line`
  evidence — or file into the backlog's `blocked/` subdirectory with a one-line
  recommendation when a spec decision is needed; end with
  `FILED {items[], commit, confidence, open_questions}` or
  `NO-ITEM {finding, why}`* — then relay its result to the user.

Either way a running Manager picks the item up within a poll cycle — no relaunch.
A new-item backlog commit is Root's sanctioned write to the main working tree
(profile, "Sanctioned direct-write paths"); everything else lands via the
Merge-Clerk.

Remember the charter's item lifecycle: **an item is done when the merging commit
deletes its item file.** A backlog file that outlives its merged work gets
re-dispatched later — if you file an item in its own commit, make sure the
landing commit still deletes it.

If the **Manager** sends `NEEDS-RESEARCH {item, question}` (an item that turned
out to need real investigation), you decide: dispatch the same research spike, or
surface it to the user if it is really an owner ruling. The Manager never
commissions research itself.

## Relay duty — a latency optimization, not a correctness gate

Team-agent completions (a Developer's `READY-FOR-REVIEW`, a Reviewer's
`APPROVED`/`CHANGES-REQUESTED`, the Merge-Clerk's `MERGED`, a QA report) can
arrive in YOUR conversation instead of the Manager's whenever the Manager is
between turns — normal harness routing. **Relay the payload to the Manager
verbatim via SendMessage immediately.** But this is a *latency* optimization: the
Manager also reconciles from git on a short cadence, so a relay you miss costs
one poll, not a stall. Relay promptly anyway; don't sit on it.

> DELETE WHEN notifications route to between-turn parents — this verbatim-relay
> duty and the "treat a relay as a first-class payload" framing exist only
> because child completions currently bubble to the root when the Manager is
> between turns.

## Status relay

**The heartbeat cron is the primary channel — root pulls, it does not wait for a
push.** Relying on the Manager to push every 10 min is a proven failure mode: the
Manager gets absorbed in a long tool run and skips the cadence, and even when it
does `SendMessage`, the report only reaches the user if root is independently
re-invoked to relay it. So the heartbeat you install at spawn (see
[Spawning](#spawning)) is what guarantees the user hears from the team. Its
prompt — substitute the profile's default branch and branch-naming convention:

> TEAM STATUS HEARTBEAT (auto). A delivery-team run is (or was) active. (1) Derive
> real status from git — never task status alone: `git -c log.showSignature=false
> log --oneline -6 <default-branch>`, `git branch --list 'item/*'`,
> `git worktree list`, per-worktree mtimes; note new merges + moving branches
> since last beat. (2) Manager liveness: `TaskList`; if the Manager id is known
> `SendMessage` it a `STATUS-REQUEST` and fold in its reply if it returns within
> ~60s; if dead (~15 min no merges / no branch+worktree movement / no live task)
> recover per the watchdog — relaunch a fresh Manager with the same args. (3)
> **Post a short update TO THE USER**: timestamp, what landed (SHAs), what's in
> flight (item branch + last activity), anything BLOCKED/NEEDS-RESEARCH, unsigned
> SHAs to re-sign. Never a bare "it's quiet". (4) If the backlog is DRAINED and
> the default branch is idle: tell the user the run is complete and `CronDelete`
> this heartbeat (find its id via `CronList`).

The Manager's own push (a report table: items → status → assigned agent +
developer model / reviewer-<n> / merge-clerk → stage-elapsed → rough ETA →
unsigned SHAs) still arrives on merges and is a welcome *supplement* — relay it
when it lands, cross-checked against git. But never depend on it; the heartbeat
is the floor. Aggregate unsigned SHAs across reports under "Unsigned commits —
need re-signing" (derived mechanically per the charter, never hand-tallied).

**On-demand status ("what's the status?").** Don't make the user wait for the
cadence: (1) SendMessage the Manager a `STATUS-REQUEST` — it reconciles its
ledger against git and pushes a fresh report; (2) while that round-trip runs
(~30–60s), give the user the instant git-derived snapshot yourself (recent
merges, live worktrees + activity, item-branch tips); (3) relay the Manager's
report when it arrives, flagging any divergence. If the Manager doesn't respond
within a couple of minutes, that IS the status — run the liveness check and
report/recover per the watchdog section.

## Watchdog + recovery

Liveness = merges on the default branch, item-branch tip movement, worktree
mtimes, transcript growth — **never** output-file mtime, **never** task status
alone. A dead Manager takes its children with it but loses nothing: every branch
and worktree survives on disk. Recovery: relaunch a fresh Manager — it
reconstructs from `git worktree list`, `git branch --list 'item/*'`, and the
backlog, and spawns fresh Developers that **adopt** the parked worktrees. A
completed Manager task ending in `HANDOFF: relaunch manager` → relay its interim
report and relaunch immediately with the same args + its state block. Ending in
**DRAINED** → the backlog is empty of actionable work; relaunch only when new
items are filed.

`MERGE-BLOCKED` in a report means the owner's uncommitted edits in the main
working tree collide with a merge — **surface it to the user; never resolve it
yourself.**

## Exploration and spike promotion

Exploration is a root-owned specialist run *outside* the pipeline, not a team
member. When the user asks for exploration time, run it per its own note:
[exploration-and-promotion.md](exploration-and-promotion.md).
