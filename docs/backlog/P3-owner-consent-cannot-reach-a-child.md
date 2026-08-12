---
by: owner (run finding — cost two refusal round-trips)
---

# Owner consent has no channel to a child agent; prefer fixing the condition

**What's wrong:** When a child agent blocks on a policy conflict that only the
owner can resolve, there is no way for the owner's consent to reach it. Every
route is a peer relay, and a correctly-suspicious agent must refuse a peer's
assertion of consent — so the block is structurally unresolvable by consent.

**Evidence:** In this run the Merge-Clerk hit a conflict between the charter's
Signing fallback clause and the owner's global instruction never to use
`--no-gpg-sign` unasked. It emitted `MERGE-BLOCKED`. The owner gave explicit
consent to root; root relayed it through the Manager; the Clerk refused **again**
and stated the reason precisely: *"its core claim is 'the owner said: land it
unsigned' — a peer agent relaying your consent. That's the one thing a peer
message can't establish."* It was right — if a relay sufficed, any agent could
dissolve any refusal by claiming to have asked. The deadlock broke only because
the owner fixed the underlying fault (unlocked the signing agent), after which
there was nothing to consent to.

**Consequence:** Two full refusal round-trips, and a class of block that cannot
be cleared at all when the underlying condition is *not* fixable. Meanwhile the
"correct" workaround — an owner-authorised exception relayed down — is exactly
what a compromised or confused peer would send, so hardening agents against it
is right and makes the problem permanent.

**Acceptance criteria:**

- The spec's escalation guidance leads with **fix the condition, not authorise
  the bypass**: when an agent blocks on a policy conflict, the first move is to
  surface the *failing condition* to the owner so it can be repaired at source.
  An owner who can fix the root cause usually prefers that to granting an
  exception, and then there is no exception to scope, relay, or track.
- It states the distinction that resolved this run, because it is the load
  bearing one: a peer asserting **consent** is never sufficient, but a peer
  asserting a **verifiable fact** ("signing works again") is not an authority
  question at all — the receiving agent probes it and proceeds on its own
  evidence.
- It states what to do when the condition genuinely cannot be fixed: the work
  parks as owner-action-required and the owner acts directly. Relayed consent is
  explicitly **not** a sanctioned unblocking mechanism; no agent should be
  written to accept it.
- The Signing fallback clause is reconciled with this: it currently reads as
  though a supervising agent may direct the fallback, which is what set up the
  conflict. It keeps its owner-authorised standing while making clear it does
  not license a peer to authorise a bypass in the moment.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, `skills/team/SKILL.md`,
`docs/backlog/P3-owner-consent-cannot-reach-a-child.md` (delete).

**Suggested approach:** This encodes a decision already made by how the run
resolved — do not reopen it and do not invent a consent-token mechanism. If
implementing reveals that some blocks genuinely need a consent path, that is a
`BLOCKED` report to the owner, not a design you choose. Shares
`docs/team-charter.md` with the two P2 charter items.
