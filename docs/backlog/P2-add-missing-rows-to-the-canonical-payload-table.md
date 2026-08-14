---
by: qa (consistency-sweep)
---

# The charter's payload table declares itself exhaustive but omits the Manager → QA messages

**What's wrong:** `docs/team-charter.md`'s "Message schemas — the canonical
payload table" states that every message conforms to one of its shapes, but the
Manager → QA messages `RUN-CONSISTENCY` and `RUN-CYCLE` — emitted by
`agents/manager.md` and gating behaviour in `agents/qa.md` — have no row.

**Evidence:**

Table side (owning, declares itself exhaustive):

- `docs/team-charter.md:432` — "## Message schemas — the canonical payload table"
- `docs/team-charter.md:434-435` — "Children end their turn to talk to their
  spawner (their final text is the payload); they are resumed with SendMessage.
  **Every message conforms to one of these shapes.**"
- Rows run `docs/team-charter.md:440-455`. There is no `Manager → QA` row at all;
  the only Manager→child row is `docs/team-charter.md:453` —
  "| Manager → Developer | `FEEDBACK` / `REBASE` / `SHUTDOWN` | …".

Agent side (both ends, and the agent files defer to the table):

- `agents/manager.md:301` — "everything routes through you (schemas: the
  charter)", then `agents/manager.md:304-306` — "You are the QA's clock —
  `RUN-CYCLE` after each MERGED and on your status cadence, resource grants /
  `RUN-CONSISTENCY` as needed".
- `agents/qa.md:176-177` — "If a consistency pass is due — every ~5 heartbeats, or
  whenever the Manager sent `RUN-CONSISTENCY` — invoke the **`consistency-check`**
  skill".

`RUN-CONSISTENCY` is the load-bearing half: it is named at both ends and gates
distinct QA behaviour. `RUN-CYCLE` is one-sided — it occurs exactly once
repo-wide, at `agents/manager.md:305`, and `agents/qa.md` never names it,
describing the same event as an unnamed bare resume: "The **Manager resumes you**
for the next cycle when the default branch moves again" (`agents/qa.md:111-114`,
and again at `agents/qa.md:139-140` and `agents/qa.md:200`).

**Canonical side:** `docs/team-charter.md`'s payload table. It is the
explicitly-normative owning section, it states the universal claim itself, and
the agent files point *at* it rather than the reverse
(`agents/manager.md:301`, `agents/reviewer.md:38`). The existing exemption is
narrow and does not cover these: the standing review ruling in
`.claude/agent-memory/delivery-team-reviewer/project_charter-payload-table-scope.md:19-21`
says "do not raise \"new message not in the canonical payload table\" … for a
Manager→Root payload. **Do raise it for any child↔Manager message.**"
`RUN-CYCLE`/`RUN-CONSISTENCY` are Manager→QA.

**Two adjacent candidates checked and deliberately excluded** — do not re-file
them as separate items:

- Manager → QA/Reviewers/Merge-Clerk `SHUTDOWN` (`agents/manager.md:356-357`) is
  **not** a missing shape. `docs/team-charter.md:453` already declares the shape
  `SHUTDOWN` with payload `{merged_sha | reason}`, and the `reason` variant is
  exactly the drained spin-down case. At most the From→To cell is narrower than
  practice.
- Reviewer → Manager `HANDOFF {pending: []}` (`agents/reviewer.md:203-204`) is a
  **judgment call, not a settled defect**. `HANDOFF` appears nowhere in the
  charter for any role — repo-wide it exists only at `agents/reviewer.md:204`,
  `agents/manager.md:374` (`HANDOFF: relaunch manager`, Manager→Root) and
  `skills/team/SKILL.md:295`. Read one way it is a child↔Manager message the
  ruling above says to raise; read the other way it is the context-degradation
  self-replacement signal, uniformly excluded from the table, and excluding the
  Manager's variant while tabling the Reviewer's would be inconsistent. See the
  open question below.

**Consequence:** The table cannot be trusted as the schema authority it claims to
be. An agent (or a reviewer checking a new message against it) that treats the
table as complete will reject or fail to recognise `RUN-CONSISTENCY`, and QA's
consistency cadence — the trigger for this very audit — has no documented wire
shape. The `RUN-CYCLE` naming mismatch means the Manager and the QA describe the
same event with two different vocabularies, so neither file alone tells a fresh
agent whether a token is sent.

**Acceptance criteria:**

- `docs/team-charter.md`'s payload table gains a `Manager → QA` row covering
  `RUN-CONSISTENCY` with its payload shape.
- The `RUN-CYCLE` question is reconciled across both ends: either it is named in
  `agents/qa.md`'s activation text alongside a table row, or the label is dropped
  from `agents/manager.md:305` and described as a plain resume. It is not left
  named in one file and unnamed in the other.
- The fix lands in `docs/team-charter.md` (plus `agents/qa.md` if the reconcile
  goes that way), not by weakening the table's "Every message conforms to one of
  these shapes" claim.
- Verification is a read: after the change, every message token appearing in
  `agents/*.md` on a child↔Manager edge resolves to a row in the table, or is
  covered by a documented exemption. (No test suite exists —
  `.claude/project-profile.md:24-26`.)
- This item file is deleted by the merging commit.

**Open question for the owner (answer before or while implementing):** should
`HANDOFF` be tabled at all? Either (a) add a `Reviewer → Manager | HANDOFF |
{pending[]}` row and keep the Manager→Root `HANDOFF: relaunch manager` out under
the Manager→Root precedent — which also disambiguates two same-token, different
shape messages — or (b) state in the table's preamble that context-degradation
handoff signals are uniformly out of scope. Both are defensible; leaving one
tabled and one not is the outcome to avoid. If this reads as a re-decision rather
than a repair, it is the owner's call and belongs in `blocked/`.

**Exact files to change:** `docs/team-charter.md` (the table, lines 440-455, and
its preamble at lines 434-435 if the `HANDOFF` scope is stated there);
`agents/qa.md` (activation text at lines 111-114 / 139-140 / 200) and/or
`agents/manager.md:305` for the `RUN-CYCLE` reconcile.

**Suggested approach:** Add the `Manager → QA` row first — it is uncontested — and
resolve `RUN-CYCLE` by preferring the cheaper direction: if the QA's next cycle is
in practice a bare resume, drop the label from `agents/manager.md` rather than
inventing a token the receiver never reads. Take the `HANDOFF` decision explicitly
rather than by omission.
