# Team charter — shared rules for every delivery-team agent

This is the single source of the rules every team agent obeys — the Manager,
the Developers, the Reviewers, the Merge-Clerk, the QA, the Engine-Supervisor,
and a root-dispatched Researcher. Each agent file references this charter
instead of restating these rules; where an agent adds a role-specific twist it
says so at its own point of use. When this charter and an agent file disagree,
the more specific agent file wins for that role, but the invariants below
(writers, signing) are floor rules no role overrides.

**This charter is generic and identical for every project.** Everything
project-specific — quality-gate commands, backlog location, invariant guards,
parity requirements, content rules — lives in that project's
`.claude/project-profile.md`. See [Project profile](#project-profile).

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

## Signing fallback

**Owner-authorised, and it overrides the global "never use `--no-gpg-sign`"
rule.** That global rule exists to stop an agent quietly disabling a check to
make a problem go away. This is the opposite: a *declared*, *reported*, and
*reversible* fallback for an unreachable signing agent, with the SHA tracked
for a later bulk re-sign that is content-verified. A signing failure never
blocks delivery. Do not treat a `--no-gpg-sign` fallback taken under this
clause as a policy violation, and do not report it as one.

The rule still stands for everything it was written for: never `--no-verify`,
never skip a test or a guard, never widen an allowlist to make a check pass.

Sign commits via the configured signing key. If the signing agent is
unreachable, retry **once**, then commit unsigned (`--no-gpg-sign`) and report
`signed: false`. Signing never blocks work. Every unsigned SHA goes on the
re-sign list; re-signing is owner-owned and happens later — it never gates
delivery.

**The re-sign list is derived mechanically, never hand-accumulated.** Over any
range:

```
git -C <repo> log --format='%H %G?' <range>
```

the SHAs whose `%G?` is `N` are the unsigned set. Report that set; do not keep
a parallel hand-maintained tally that can drift.

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
   item to `blocked/`, worktree/branch lifecycle. Never a content edit.

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
  item.
- **Landing:** fast-forward only, one commit per item, linear history.

A project may override any of these in its profile's **Worktree layout**
section; where it does, the profile wins and agents use its values. Where the
profile is silent, these defaults apply — do not treat a silent profile as
undefined.

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
| Merge-Clerk → Manager | `MERGED` | `{item(s), sha, signed}` |
| Merge-Clerk → Manager | `MERGE-BLOCKED` | `{reason}` (surfaced to Root immediately) |
| Merge-Clerk → Manager | `REBASE-REQUIRED` | `{item, branch, base, reason}` (clerk rebase hit real conflicts) |
| Manager → Developer | `FEEDBACK` / `REBASE` / `SHUTDOWN` | `{branch, comments, required[]}` / `{branch, onto}` / `{merged_sha \| reason}` |
| QA → Manager | `REGRESSION` / `CONSISTENCY` / `QA-GREEN` / `QA-BLOCKED` | `{item_file, first_bad_sha, test}` / `{items_filed[], scope}` / `{tip, cycles}` / `{reason}` |
| Agent → Manager | `PORTS-REQUEST` / Manager → Agent | `PORTS-GRANT` | `{item, branch}` / `{item, granted: bool, ports[]}` — the exclusive-resource lease (ports, devices, fixture services) named in the profile's Verification environment; only one holder at a time |

External-engine messages (`ENGINE-UNAVAILABLE`, `ENGINE-LAUNCH-FAILED`,
`REBASE-CONFLICT`, `ADVISORY-FINDINGS`) are defined in the
[engine-supervisor agent](${CLAUDE_PLUGIN_ROOT}/agents/engine-supervisor.md)
and apply only to runs whose project profile declares an external engine.
