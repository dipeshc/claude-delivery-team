---
by: qa (consistency-sweep)
---

# The charter's payload table defines a QA → Manager `CONSISTENCY` message nothing emits and nothing reads

**What's wrong:** `docs/team-charter.md`'s canonical payload table declares a
QA → Manager `CONSISTENCY` message with payload `{items_filed[], scope}`, but no
agent file ever emits it and no agent file ever consumes it — it is residue of an
earlier message set.

**Evidence:**

Table side:

- `docs/team-charter.md:454` — "| QA → Manager | `REGRESSION` / `CONSISTENCY` /
  `QA-GREEN` / `QA-BLOCKED` | `{item_file, first_bad_sha, test}` /
  `{items_filed[], scope}` / `{tip, cycles}` / `{reason}` |"

Emitting side — `agents/qa.md` defines exactly three exits, and the consistency
cadence is not one of them:

- `agents/qa.md:171` — "End your turn with `REGRESSION {item_file, first_bad_sha,
  test}`"
- `agents/qa.md:91` — "if you can't, report `QA-BLOCKED`"
- `agents/qa.md:199-200` — the sole exit of the consistency cadence, including the
  Manager-triggered `RUN-CONSISTENCY` case at `agents/qa.md:176-177`: "End your
  turn with the terse `QA-GREEN {tip, cycles}` (plus anything filed) — the Manager
  resumes you for the next cycle."

Consuming side — `agents/manager.md:304-306` names only "a QA `REGRESSION` is P0
and jumps the queue". A repo-wide grep for `CONSISTENCY` returns three hits:
`agents/qa.md:177` and `agents/manager.md:306` (both `RUN-CONSISTENCY`, a
different Manager→QA message) and the charter row itself.

**Canonical side:** `agents/qa.md`. The charter sets the tie-break itself at
`docs/team-charter.md:7-9` — "When this charter and an agent file disagree, the
more specific agent file wins for that role" — and this is not one of the floor
rules that clause excepts (writers, verification). QA's own file is unambiguous
that the consistency pass exits with `QA-GREEN {tip, cycles}` "(plus anything
filed)", which subsumes what `{items_filed[], scope}` was for. This is the
"sections, fields, or files nothing consumes" residue class named in
`skills/consistency-check/SKILL.md:83-87`.

**Consequence:** A field nothing produces and nothing reads sits in the document
that calls itself the canonical schema authority. An agent reading the table
concludes it must emit `CONSISTENCY` after a consistency pass, ends its turn with
a payload the Manager has no handler for, and the pass's result is dropped.
Low likelihood, but it is a false instruction in the one place agents are told to
trust.

**Acceptance criteria:**

- The `CONSISTENCY` token and its `{items_filed[], scope}` payload are removed
  from `docs/team-charter.md:454`, leaving
  `REGRESSION` / `QA-GREEN` / `QA-BLOCKED` and their three payloads aligned in
  order.
- No new emitter is invented: `agents/qa.md` is not changed to start sending
  `CONSISTENCY`, since the QA-GREEN "(plus anything filed)" path already covers
  reporting a consistency pass.
- Verification is a read: a repo-wide grep for `CONSISTENCY` returns only
  `RUN-CONSISTENCY` occurrences. (No test suite exists —
  `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md` (line 454 only).

**Suggested approach:** A one-line edit — delete the token and its
slash-separated payload, keeping the remaining three tokens and payloads in
matching order. If it lands alongside
`P2-add-missing-rows-to-the-canonical-payload-table`, do both table edits in that
item's commit rather than touching the same row twice.
