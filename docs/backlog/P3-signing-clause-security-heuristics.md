---
by: owner (shakedown finding F3)
---

# Dispatches should reference the signing fallback by pointer, not restate it

**What's wrong:** When a dispatch restates the charter's signing-fallback
mechanics inline, the literal `--no-gpg-sign` instruction reads to an external
security layer as an agent pre-authorizing itself to skip a check, and the run
gets flagged.

**Evidence:** `docs/team-charter.md:114-121` ("Signing fallback") is
owner-authorised and even anticipates being mistaken for a violation ("do not
treat a `--no-gpg-sign` fallback taken under this clause as a policy
violation") — but that reassurance is addressed to *agents*, and an external
security monitor never reads the charter. In the shakedown the Manager's
dispatch relayed the clause verbatim and the run was flagged with a
"[Security Weaken]" warning, despite every landed commit being signed (`%G?`=G)
and the fallback never firing.

**Consequence:** False-positive security warnings on ordinary runs. They cost
owner attention and, repeated, devalue the warning that finally matters.

**Acceptance criteria:**

- `agents/manager.md`'s dispatch-prompt contents specify that signing is passed
  **by pointer** ("sign per the charter's Signing fallback section"), never by
  restating the `--no-gpg-sign` mechanics.
- `docs/team-charter.md`'s Signing fallback section states that the clause is
  cited by reference in dispatches and messages, and that the fallback is
  reported after the fact (`signed: false` + the SHA) rather than
  pre-authorized in a prompt.
- The substance is unchanged: the fallback remains owner-authorised, signing
  still never blocks delivery, and the mechanical re-sign list still governs.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/manager.md`, `docs/team-charter.md`,
`docs/backlog/P3-signing-clause-security-heuristics.md` (delete).

**Suggested approach:** A sharpening, not a re-decision — the policy stands, only
how it is transmitted changes. Shares both files with
`P2-plan-mode-halts-run`; group them into one commit if dispatched together.
