---
by: owner
---

# Root-held payloads have stalled two runs; the relay duty needs a checklist hook

**What's wrong:** Child completions that bubble to root while the Manager is
between turns depend on root forwarding them promptly. Root has now held a
payload twice across runs — 12 minutes once, 11 minutes once — each time
stalling a Manager that was waiting on exactly that message.

**Evidence:** `skills/team/SKILL.md`, the relay-duty section (which already
calls the relay "a latency optimization, not a correctness gate" — true, and
both stalls were pure latency with no lost work). Both incidents involved a
`CLOSED` payload arriving in root's conversation at a busy moment and being
reported to the owner but not forwarded to the Manager. The existing `DELETE
WHEN` tag on this duty names the peer-messaging successor as the sunset; this
item is mitigation until that fires, not a replacement for it.

**Consequence:** Minutes of idle pipeline per occurrence, and a Manager whose
cleanup or next dispatch waits on a message root already holds. Twice is a
pattern: the failure mode is root treating "reported to the owner" as
"delivered to the Manager".

**Acceptance criteria:**

- The relay-duty section gains the concrete discipline that failed: **relay
  before narrating** — when a team-agent payload arrives in root's
  conversation, the SendMessage to the Manager happens before root reports the
  event to the owner, so the forward cannot be displaced by the summary.
- The heartbeat prompt in the same skill instructs each beat to check for
  unforwarded payloads from the interval since the last beat — a sweep that
  bounds any miss at one heartbeat interval.
- The rule keeps the existing framing (latency optimization; the Manager's git
  reconciliation remains the correctness backstop) and keeps the `DELETE WHEN`
  sunset intact — this narrows the gap, the successor removes it.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `skills/team/SKILL.md`,
`docs/backlog/P3-root-relay-stall-mitigation.md` (delete).

**Suggested approach:** A sharpening of an existing duty with an ordering rule
and a sweep; do not add machinery beyond the two clauses.
