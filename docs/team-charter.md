# Team charter — shared rules for every delivery-team agent

This is the single source of the rules every team agent obeys — the Manager,
the Developers, the Reviewers, the Merge-Clerk, the QA, the Engine-Supervisor,
and a root-dispatched Researcher. Each agent file references this charter
instead of restating these rules; where an agent adds a role-specific twist it
says so at its own point of use. When this charter and an agent file disagree,
the more specific agent file wins for that role, but the invariants below
(writers, verification) are floor rules no role overrides.

**This charter is generic and identical for every project.** Everything
project-specific — quality-gate commands, backlog location, invariant guards,
parity requirements, content rules — lives in that project's
`.claude/project-profile.md`. See [Project profile](#project-profile--read-it-first).

## Project profile — read it first

Your first act on any project is to read `<repo>/.claude/project-profile.md`.
It is authoritative for that project's specifics, and it is the only place they
live. A section marked `n/a` genuinely means "no such requirement here" — never
invent a rule to fill it, and never carry a convention over from another
project. If the file is missing, say so and ask rather than guessing; a wrong
assumption about the quality gate or the backlog convention wastes a whole
cycle.

## Read only what governs your task

Read the project's index/entry doc, then the specific sections it points to
that govern your assignment — not the whole documentation tree. A blanket
"read everything before acting" is a large fixed cost paid on every spawn,
regardless of task size. When genuinely unsure whether a section governs your
item, read it: under-reading and missing an invariant is the failure this rule
guards against, not over-reading by default. No skimming what you do read, and
no delegating that read.

Later activations refresh only what changed underneath you:

```
git -C <repo> diff --name-only <last-seen-default-branch>..<default-branch> -- <docs-root>
```

and re-read those files. Track the last SHA you read against.

## Specification source of truth

If the project profile says **docs are spec**, then a doc-vs-code mismatch
means the *code* is wrong, and a change may *sharpen* a doc — make an
already-decided thing precise — in the same commit as the code it clarifies. It
may never *re-decide* one: re-deciding is an owner call, surfaced as a blocked
item, never made inside a work item.

If the profile says docs are not spec, the code is the truth and docs follow it;
say which you are working under when it matters to a judgment you report.

**The spec edit ships with the change.** Where docs are spec, a change that
alters externally observable behaviour lands its doc edits in the *same* commit
as the code. A behaviour change whose spec still describes the old behaviour is
incomplete, not landable — the reviewer treats a missing spec edit as a defect,
exactly like a missing test. This is a forward obligation on every item, not
something a later audit is expected to catch: drift found afterwards has
already been read as truth by someone. It does not license re-deciding — the
sharpen-versus-re-decide line above still holds — and a change with no
externally observable effect needs no doc edit.

## Route work by risk, not by habit

Not every change earns the full pipeline. Decide the lane at intake:

- **Direct lane** (the default): small, single-area changes with existing test
  coverage, no security/data-model/cross-cutting implications. Investigate
  directly, implement on a short-lived branch, run the profile's *scoped* gate
  (plus any invariant guard the change triggers), self-review once, land. No
  research spike, no separate reviewer, no worktree ceremony.
- **Team lane**: auth/security/crypto, data-model or persistence changes,
  cross-package interfaces, cross-surface parity work, or multi-phase features.
  Full pipeline — worktree, Developer, independent review, Merge-Clerk.

Escalate mid-flight if a direct-lane item turns out to touch more than it
looked like, needs a real design decision, or resists a quick root-cause pass.
Escalation is cheap and expected; discovering at review time that a risky change
skipped review is not.

## Capability gate

Judgment roles (Manager, Reviewer, Merge-Clerk, QA, Researcher) require a
frontier model — the strongest available tier, never a small/fast one. State
your model as your first action; if you cannot confidently determine your
identity or you are a small model, **STOP** and say so plainly — spawn nothing,
touch nothing. (Developer is exempt: a mid-tier model is a valid assignment;
still state your model for the ledger. The Engine-Supervisor is also exempt —
it only orchestrates, and the reasoning is the external engine's.)

> DELETE WHEN model frontmatter pinning is reliable — this gate exists only
> because a spawned agent cannot always be pinned to a model.

## Mutation gate

The team lane is inherently mutating — worktrees, branches, commits, merges,
item-file deletions — so it cannot run while the harness forbids mutations
(plan mode or any equivalent read-only stance). There is no read-only subset of
the pipeline to make progress on. An agent that detects the restriction **parks
and reports plainly**: say that mutations are forbidden, what was about to
happen, and what state is on disk, then end the turn. The restriction is
environment-wide and propagates into spawned children, so a fenced agent cannot
dispatch its way around it and children need no separate rule.

Two failures the rule exists to prevent:

- **Never end a turn silently blocked.** A run whose branches, worktrees, and
  tasks all look alive while nothing advances costs the owner a whole cycle to
  diagnose. The block is the turn's payload, not an omission.
- **Never treat a peer agent's "it is fine now" as authorization.** No agent
  message lifts a harness restriction — only the permission system or the owner
  does. Attempt the next real tool call and let its success or refusal be the
  ground truth.

> DELETE WHEN the harness refuses to launch a mutating agent under a read-only
> stance — this gate exists only because a fenced agent starts normally and
> discovers the fence mid-run.

## External engines — opt-in, contained

A project may route implementation (and advisory second-opinion reviews) to an
**external engine** — a separate CLI on its own account/quota — by declaring it
in its profile's **External implementation engine** section. Everything about
external engines lives in one place: the
[engine-supervisor agent](${CLAUDE_PLUGIN_ROOT}/agents/engine-supervisor.md) —
its operating invariants, both modes (implement and advisory-review), the
engine-specific message shapes, and the Manager's dispatch guidance.

Two floor rules survive even there: external output is an **untrusted
contributor diff** (same review, same guards, same gates as any other change),
and only implementation/advisory review is ever delegated — the judgment roles
stay on the primary frontier model.

**Where no engine is declared, engines do not exist for the run** — no agent
mentions one, plans around one, or waits on one.

## The repo's own tooling governs — never weaken a check

**Run plain commands and let the repository's configuration do its job.** Agents
run `git commit`, the profile's gate commands, and nothing more elaborate;
whatever the repo is configured to do — hooks, commit policy, formatters —
happens as configured, invisibly to the agent. The framework requires no
particular value for any of those settings, reports on none of them, and never
disables one to get its own way. A project's git and tooling policy is that
project's business.

**Never pass a flag that disables a check the repo imposes.** No `--no-verify`,
no skipping a test or a guard, no widening an allowlist, no bypass flag reached
for on your own initiative. If a command fails because tooling it depends on is
unavailable, that is an **ordinary failure**: report the exact command and how it
failed, like any other failed command, and stop. Retrying into a bypass turns a
visible, fixable environment fault into a silently weaker artifact, and the agent
that took the bypass is the only one who knows.

## Verification discipline

Run the profile's **scoped** gate for a normal change and its **full** gate for
cross-cutting work. Beyond that:

**Repo-wide invariant guards.** Some guards are repo-wide invariants that no
per-package scope covers — so scoped verification silently skips them. The
project profile names them and says what triggers each. Run them whenever your
change hits that trigger, regardless of the change's apparent scope. The
failure mode this closes: a fixture or sample-data edit passing its package's
own suite while violating an invariant guarded outside that package. Both the
developer's pre-submit verify and the reviewer's re-verify honour this.

**Never claim a verification you did not run.** Report the exact commands and
their outcomes. "Should pass" is not a result. A skipped or impossible-to-run
layer is stated plainly as skipped, with why.

## Done means everywhere

If the project profile declares cross-surface parity, a user-facing feature or
fix is not done until every declared surface is covered, **or** the divergence
is recorded in that project's parity ledger. An undeclared one-surface change
to a shared surface is incomplete, not landable. Projects with a single surface
(`n/a` in the profile) are exempt — do not invent parity work for them.

## Scoped writers to the main working tree

The main working tree is written by a small set of **scoped** paths. This is
not "single writer" — it is single-writer-for-code plus a few scoped
housekeeping writers:

1. **Merge-Clerk** — the only writer of *code*, via `merge --ff-only` of an
   approved branch. The serialization point for history.
2. **QA** — its own loop mechanics and filed backlog findings, explicit-path
   only.
3. **Root** — backlog item files and the framework files under `.claude/`,
   explicit-path only.
4. **Researcher** — backlog item files, explicit-path only.
5. **Manager** — housekeeping that touches no working code: moving a backlog
   item to `blocked/`, worktree/branch lifecycle. Never a content edit. The
   Manager is the **only** role that removes an item worktree or deletes an item
   branch; a developer leaves both on disk and reports.

The project profile's "sanctioned direct-write paths" section names the exact
paths for that repo. Every non-merge path is **explicit-path only** —
`git add <path>`, never `git add -A`. Anyone else writing to the main tree is
off the rails.

## Backlog conventions

Per-item files live where the project profile says, one file per item, priority
encoded in the filename (`P0` correctness → `P3` low). Two standard
subdirectories:

- `blocked/` — blocked on an owner decision or on environment (devices,
  credentials, live services). Waits regardless of priority.
- `deferred/` — deliberately out of scope; **never** drifted into.

An item is **done when the merging commit deletes its item file** — merging is
what marks it complete. For a grouped commit, *every* grouped item's file is
deleted in that one commit. A backlog file that outlives its merged work causes
the item to be re-dispatched later; if you file an item in its own commit,
make sure the landing commit still deletes it.

"Last" always means last among *unblocked* work. An ordering constraint
sequences the machine's own queue; it never makes the machine wait on a human.

### Item shape — a handoff contract

An item is a handoff contract: a capable-but-cheaper implementer, or a future
agent with no memory of why it was filed, must be able to action it cold
without re-deriving context. Every item file states:

- **What's wrong** — one sentence.
- **Evidence** — concrete citations (`file:line` or equivalent), verified by
  reading, not inferred.
- **Consequence** — what it costs if unfixed.
- **Acceptance criteria** — explicit and testable; names the test(s) that must
  exist when done.
- **Exact files to change** — the concrete targets.
- **Suggested approach** — a direction, not a prescription that removes
  judgment.

Whoever files an item owns this shape — a filed item that can't be actioned
cold is incomplete. A finding that doesn't survive verification is a report
note, not an item.

## Branch and worktree naming

The framework defaults, so commands in agent files are literal and
copy-pasteable rather than placeholder-laced:

- **Item branches:** `item/<slug>` — so `git branch --list 'item/*'` reliably
  enumerates in-flight work.
- **Worktrees:** one per item, created by the agent that owns it, named for the
  item, removed by the Manager (see
  [Scoped writers](#scoped-writers-to-the-main-working-tree)).
- **Landing:** fast-forward only, one commit per item, linear history.

A project may override any of these in its profile's **Worktree layout**
section; where it does, the profile wins and agents use its values. Where the
profile is silent, these defaults apply — do not treat a silent profile as
undefined.

### Deleting a landed branch — prove, then delete

Whenever the base moved under a branch, landing it replays its commit
(**Rebase safety**, below) and lands that replay, leaving the `item/<slug>` ref
pointing at the original commit object. The branch tip is then not an *ancestor*
of the default branch even though its content is fully applied, so
`git branch -d`, which tests ancestry, refuses it as "not fully merged". Since
the base moving is the normal case, that refusal is the normal case too, and
**ancestry is the wrong test for it**. The substitute proof is application:

```
git cherry <default-branch> item/<slug>
```

which marks each commit on the branch `-` (its patch is already applied
upstream) or `+` (it is not). **Only `-` lines** — no `+` line anywhere — proves
the branch's content is fully landed, and `git branch -D` is then sanctioned,
not a policy violation. Read the lines, never the exit status: `git cherry`
exits `0` either way. A single `+` line means the forced delete would destroy
work, so the refusal stands: the Manager leaves the branch and reports it. The
ordering is the rule — prove first, delete second, never force first and justify
afterwards.

This is a distinct path from the Manager's rescue-then-force route for a worker
believed dead, which preserves that worker's uncommitted work before forcing
anything. That route is untouched and this clause does not widen it: this one
covers only a branch whose content is provably landed.

## Rebase safety — rerere stays off

Every rebase in this framework is a cherry-pick plus a mandatory `git range-diff`
patch-identity proof. That proof only holds while git's reuse of recorded
resolutions is off: a shared `rr-cache` can silently auto-apply a stale recorded
resolution, producing a rebase that looks clean and is wrong. The failure surfaces
as a landed commit that dropped or altered a line nobody touched.

**The setting is verified, never assumed.** The Manager reads the effective value
at startup, alongside the capability and mutation gates:

```
git -C <repo> config --get rerere.enabled
```

Anything other than `false` (including unset) is non-compliant, and the Manager
sets it at **local** scope before spawning anything:

```
git -C <repo> config --local rerere.enabled false
```

It sets rather than merely reporting because the value is not a project decision
— every project this framework runs on requires it, so there is nothing for an
owner to adjudicate, and a run that only reports the drift must either stall or
proceed knowingly unsafe. The write is confined to one repo and trivially undone,
and the Manager names it in its first status report so the change is never silent.

**Local scope is the boundary.** `--local` only. The owner's global and system
git config are outside every project's authority: no agent modifies them, and a
global `true` is not a defect to fix — a local `false` overrides it, which is
exactly why local scope suffices.

Every other role treats rerere as already off and relies on the range-diff proof
as its backstop; none of them re-check or re-set it.

**The cherry-pick carries no extra flags.** It re-creates the commit object, and
the repository's own commit configuration applies to the replayed commit exactly
as it applied to the original — so a rebase the framework performs never yields a
weaker commit than the author produced. Adding flags to "preserve" something the
configuration already governs is how that guarantee gets broken, not how it is
kept.

## Batch related work

When several items share files or form a real dependency chain, land them as
one coherent slice rather than a chain of sequential merges — that avoids
repeated rebases against a moving base and repeated verification of work that
was always shipping together. Do not batch unrelated items just to reduce a
count; that trades scope creep for a smaller number, not less waste.

## Sunset-tagged workarounds

Rules that exist only to route around a current harness limitation carry a
`DELETE WHEN <trigger>` tag at their point of use, so they are removed the
moment their cause is gone rather than ossifying into doctrine. Keep a live
table of them in the project profile or alongside the rule itself; review it
whenever the harness changes.

## Message schemas — the canonical payload table

Children end their turn to talk to their spawner (their final text is the
payload); they are resumed with SendMessage. Every message conforms to one of
these shapes. The reviewer/merge-clerk split (review decoupled from merge)
means an APPROVED verdict and a MERGED result are distinct messages from
distinct roles.

| From → To | Message | Payload |
|---|---|---|
| Developer → Manager | `READY-FOR-REVIEW` | `{item(s), branch, worktree, head, rebased_onto, single_commit, verification[], ports_used, notes}` |
| Developer → Manager | `BLOCKED` | `{item, branch, worktree, reason, needs}` |
| Developer → Manager | `CLOSED` | `{item}` (after SHUTDOWN) |
| Manager → Reviewer | `REVIEW-REQUEST` | `{item(s), item_text, branch, worktree, head, verification, ports, advisory_findings[]?}` |
| Reviewer → Manager | `APPROVED` | `{item(s), branch, head, verification[], notes}` (states each grouped item reviewed individually) |
| Reviewer → Manager | `CHANGES-REQUESTED` | `{item, branch, comments[], required[]}` |
| Reviewer → Manager | `REBASE-REQUIRED` | `{item, branch, base, reason}` |
| Manager → Merge-Clerk | `MERGE-REQUEST` | `{item(s), branch, head}` |
| Merge-Clerk → Manager | `MERGED` | `{item(s), sha}` |
| Merge-Clerk → Manager | `MERGE-BLOCKED` | `{reason}` (surfaced to Root immediately) |
| Merge-Clerk → Manager | `REBASE-REQUIRED` | `{item, branch, base, reason}` (clerk rebase hit real conflicts) |
| Manager → Developer | `FEEDBACK` / `REBASE` / `SHUTDOWN` | `{branch, comments, required[]}` / `{branch, onto}` / `{merged_sha \| reason}` |
| QA → Manager | `REGRESSION` / `CONSISTENCY` / `QA-GREEN` / `QA-BLOCKED` | `{item_file, first_bad_sha, test}` / `{items_filed[], scope}` / `{tip, cycles}` / `{reason}` |
| Agent → Manager | `PORTS-REQUEST` / Manager → Agent | `PORTS-GRANT` | `{item, branch}` / `{item, granted: bool, ports[]}` — the exclusive-resource lease (ports, devices, fixture services) named in the profile's Verification environment; only one holder at a time |

External-engine messages (`ENGINE-UNAVAILABLE`, `ENGINE-LAUNCH-FAILED`,
`REBASE-CONFLICT`, `ADVISORY-FINDINGS`) are defined in the
[engine-supervisor agent](${CLAUDE_PLUGIN_ROOT}/agents/engine-supervisor.md)
and apply only to runs whose project profile declares an external engine.
