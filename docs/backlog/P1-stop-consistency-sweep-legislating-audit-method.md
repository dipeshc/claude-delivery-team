---
by: qa (consistency-sweep)
---

# `workflows/consistency-sweep.js` legislates audit method the skill is declared to own, and the copy has drifted

**What's wrong:** Both the architecture doc and the skill state that
`skills/consistency-check/SKILL.md` owns the audit method and the workflow owns
only the fan-out, but the workflow's `references` prompt carries four checks the
skill's owning list does not, and its audit prompt restates the skill's
canonical-side tiebreaker list verbatim.

**Evidence:**

Ownership side:

- `docs/architecture.md:41` — "`workflows/consistency-sweep.js` | Runs that audit
  as a whole-spec sweep … **Owns the fan-out only; the method stays in the
  skill.**"
- `docs/architecture.md:40` — the skill "Owns the method — the applicability
  gate, the finding classes, **how a contradiction is resolved on paper**, and the
  filing rules."
- `skills/consistency-check/SKILL.md:21` — "This file owns the method."
- `docs/architecture.md:5-6` — "Where this document and an agent or skill file
  disagree, this document is right."

Drift, instance 1 — the `references` class:

- `workflows/consistency-sweep.js:74` — "Reference integrity: every relative link
  resolves, every #anchor has a generating heading (GitHub slug rules — a heading
  containing \" — \" yields a double hyphen), every ${CLAUDE_PLUGIN_ROOT} path
  exists, every citation of a section title matches a real heading verbatim, and
  every cited behaviour is actually described where it is cited."
- `skills/consistency-check/SKILL.md:76-77` — "**Reference integrity** — every
  relative link resolves; every `#anchor` has a generating heading; every
  decision-record citation points at a real record."

Four checks exist only in the workflow: the GitHub slug rule for `" — "` headings,
`${CLAUDE_PLUGIN_ROOT}` path existence, verbatim section-title matching, and
cited-behaviour-actually-described.

Drift, instance 2 — the resolution rule, copied not pointed at:

- `workflows/consistency-sweep.js:116` — "which side is canonical and why
  (glossary over usage, newer decision over older, explicitly-normative over
  passing mention, the owning section over an incidental restatement, an owner
  ruling over anything predating it)"
- `skills/consistency-check/SKILL.md:93-95` — the same five tiebreakers, in the
  same order, in the section `docs/architecture.md:40` assigns to the skill as
  "how a contradiction is resolved on paper".

This copy has *not* drifted yet, which is why it belongs in the same fix — it is
the next divergence waiting to happen, and reducing it to a pointer costs nothing
because every audit agent already reads the skill
(`workflows/consistency-sweep.js:108`).

**Not part of this defect** (checked and dismissed, so a future audit does not
re-raise them): the `contradictions` class divergence between
`workflows/consistency-sweep.js:73` and `skills/consistency-check/SKILL.md:71-73`
is not drift — both state the identical rule ("the same fact asserted two ways")
and then give an *open, parenthetical example set*; the skill's examples are
generic-project (status codes, route shapes, config field names) as befits a
project-agnostic skill, the workflow's are this-repo-flavoured. Likewise the
comment at `workflows/consistency-sweep.js:70-71` ("The audit classes are the
skill's phase-1 list") is still true at class level and claims no wording
equivalence.

**Canonical side:** `skills/consistency-check/SKILL.md` owns the method — the
architecture doc says so explicitly and is authoritative over any skill file, and
the skill's own line 21 agrees. The workflow is the incidental restatement.

**Consequence:** QA's incremental cadence calls the skill directly and cannot
launch a workflow (`agents/qa.md:176-178`, `docs/architecture.md:41`), so the four
workflow-only checks never run on the incremental path — the routine audit is
silently weaker than the on-demand sweep, and nothing in either file reveals the
gap. No run produces a *wrong* answer today (the extra checks are additive), but
the two copies will keep diverging as long as method lives in two places.

**Acceptance criteria:**

- The four reference checks currently unique to `workflows/consistency-sweep.js:74`
  appear in `skills/consistency-check/SKILL.md`'s phase-1 Reference-integrity item,
  so the incremental path runs them too.
- The `${CLAUDE_PLUGIN_ROOT}` check is **generalised** when folded in — that
  variable exists only in plugin repos, while the skill governs arbitrary consuming
  projects. Phrase it so it holds anywhere (e.g. "every path reference in a doc
  resolves on disk"), or invariant 6 (`docs/architecture.md:66-70`, project-agnostic
  core) is violated by the fix itself.
- The `references` `ask` string in `workflows/consistency-sweep.js` is reduced to a
  class label plus a pointer to the skill, carrying no check the skill does not
  state.
- The tiebreaker list at `workflows/consistency-sweep.js:116` is replaced by a
  pointer to the skill's resolution section rather than a second copy.
- The delivery-team-specific examples in `workflows/consistency-sweep.js:73` are
  **not** moved into the skill — doing so would put this repo's vocabulary into a
  project-agnostic skill.
- Verification is a read: after the change, every audit rule a sweep agent receives
  is either in `skills/consistency-check/SKILL.md` or is fan-out mechanics. (No test
  suite exists — `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `skills/consistency-check/SKILL.md` (phase-1 list,
lines 66-89) and `workflows/consistency-sweep.js` (the `CLASSES` `ask` strings,
lines 72-78, and the audit prompt, line 116).

**Suggested approach:** Move the checks up into the skill first, generalising the
plugin-specific one, then hollow out the workflow strings to class labels — the
workflow prompt already tells each agent to read the skill, so a label plus "your
class, and only this class" is sufficient. Resist the temptation to also unify the
example sets: differing illustrations of one identical rule are correct here,
because the skill must stay project-agnostic and the workflow ships inside this
repo.
