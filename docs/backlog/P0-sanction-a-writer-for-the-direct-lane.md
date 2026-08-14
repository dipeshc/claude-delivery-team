---
by: qa (consistency-sweep)
---

# The direct lane has no sanctioned writer — invariant 2 and the lane rule cannot both hold

**What's wrong:** The direct lane tells Root (or one mid-tier subagent) to
implement and land code on the default branch, while the charter's scoped-writers
floor rule and architecture invariant 2 name the Merge-Clerk as the *only* writer
of code and declare the housekeeping-writer list exhaustive — so the framework's
declared default lane is, on paper, off the rails.

**Evidence:**

Lane side — Root implements and lands:

- `skills/team/SKILL.md:149-159` — "**Direct lane** (the default for everything
  else — single-area, small expected diff, existing test coverage): skip the
  Manager entirely. Root (or one mid-tier subagent) does a quick grep/read pass
  itself — no dedicated research-spike agent, no backlog file, no worktree. If
  root cause is found: 1. implement on a short-lived branch; 2. run the profile's
  **scoped** gate … 3. one lightweight self-review pass; 4. ff-merge directly."
- `docs/team-charter.md:69-73` — "**Direct lane** (the default): small,
  single-area changes … Investigate directly, implement on a short-lived branch,
  run the profile's *scoped* gate (plus any invariant guard the change triggers),
  self-review once, land."
- `docs/architecture.md:55-57` — invariant 3, "**Risk-routed lanes.** Small,
  single-area changes take the direct lane; the full pipeline is reserved for
  work whose risk earns it."

Writer side — only the Merge-Clerk writes code:

- `docs/team-charter.md:231-232` — "1. **Merge-Clerk** — the only writer of
  *code*, via `merge --ff-only` of an approved branch. The serialization point
  for history." The numbered list runs 1–5 (Merge-Clerk, QA, Root, Researcher,
  Manager) and closes at `docs/team-charter.md:245-246` with "Anyone else writing
  to the main tree is off the rails."
- `docs/team-charter.md:235-236` — Root's entry is scoped to "backlog item files
  and the framework files under `.claude/`, explicit-path only" — not code.
- `docs/team-charter.md:8-9` — the section is declared a floor rule: "the
  invariants below (writers, verification) are floor rules no role overrides."
- `docs/architecture.md:51-54` — invariant 2, "**Single writer for code.** Only
  the Merge-Clerk writes code to a project's main working tree, via
  `merge --ff-only`, one commit per item, linear history. A short list of scoped
  housekeeping writers (charter, \"Scoped writers\") is the only exception, and
  none of them edit code."

The contradiction is also visible inside a single file, fourteen lines apart:

- `README.md:40-41` — "at intake, small single-area work is routed down a
  **direct lane**" (introduced as the default), against `README.md:55-56` —
  "Because only the Merge-Clerk lands code, history stays linear and every commit
  traces to a reviewed item." No carve-out sits between them.

Concrete instance in this repo: `.claude/project-profile.md:78-81` sanctions only
`docs/backlog/` and `.claude/`, so a direct-lane edit here to `agents/`, `skills/`
or `docs/` has no sanctioned writer at all.

**Canonical side:** Neither side wins on precedence — that is precisely the
defect. The lane rule is architecture invariant 3, a peer entry in the same
normative list as invariant 2, so `docs/architecture.md` contradicts *itself* and
the precedence rule at `docs/architecture.md:5-6` ("Where this document and an
agent or skill file disagree, this document is right") cannot break the tie. The
direct lane is additionally the declared default in the charter, the skill, and
the README, and QA's full-suite watch loop is named as its independent
verification (`skills/team/SKILL.md:161-163`). **Resolution: the writer rule must
gain an explicit direct-lane carve-out.** Deleting the lane by routing its
landings through the Merge-Clerk is not a viable alternative — it would
contradict invariant 3.

**Consequence:** The framework's most-used path is formally unsanctioned. Any
reviewer, QA pass, or consistency audit applying the floor rule literally must
flag every direct-lane landing as off the rails; any agent applying the lane rule
literally writes code the charter forbids. Agents resolving this ad hoc will
resolve it inconsistently, and a consuming project cannot tell which rule its
profile's "Sanctioned direct-write paths" is meant to satisfy.

**Acceptance criteria:**

- `docs/architecture.md` invariant 2 states the direct-lane exception explicitly,
  so invariants 2 and 3 can both be true as written.
- `docs/team-charter.md`'s "Scoped writers to the main working tree" list names
  the direct-lane writer as a numbered entry, stating **who** lands (Root or the
  single mid-tier subagent that did the work), and **under what proof** — the
  profile's scoped gate, plus any triggered repo-wide invariant guard, plus one
  self-review pass, per `skills/team/SKILL.md:152-159`.
- `README.md:55-56` no longer asserts an unqualified "only the Merge-Clerk lands
  code" fourteen lines after introducing the direct lane.
- `docs/project-profile.template.md`'s "Sanctioned direct-write paths" section
  makes clear whether direct-lane code paths belong in it, so a consuming project
  can fill it in unambiguously.
- Verification is a read: after the change, grepping `Merge-Clerk` and
  `direct lane` across `docs/`, `skills/`, `agents/` and `README.md` yields no
  statement that the Merge-Clerk is the sole lander without an adjacent
  direct-lane carve-out. (This repo has no test suite —
  `.claude/project-profile.md:24-26` makes reading against the spec the
  verification.)
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/architecture.md` (invariant 2, lines 51-54),
`docs/team-charter.md` ("Scoped writers to the main working tree", lines 227-246),
`README.md` (lines 55-56), and `docs/project-profile.template.md`
(lines 108-114) if the sanctioned-paths guidance needs the same qualification.

**Suggested approach:** Add the carve-out where the writer rule is *owned* rather
than qualifying each restatement — one numbered entry in the charter's
scoped-writers list plus one clause in invariant 2 — and let the lane passages
keep pointing at it. Widening the carve-out beyond the direct lane's own
preconditions (single-area, small diff, existing coverage) would hollow out
invariant 2, so keep the proof requirements in the entry itself. Whether a
mid-tier direct-lane subagent may land, or only Root, is the one judgment call
here; if it reads as a spec re-decision rather than a repair, raise it with the
owner rather than choosing.
