---
by: owner (shakedown finding F1/F2)
---

# Specify plan-mode behaviour: a team run cannot proceed under plan mode

**What's wrong:** The spec is silent on plan mode, so a Manager launched under
it stalls without saying why — as happened in the first plugin shakedown run.

**Evidence:** `docs/team-charter.md:83` ("Capability gate") is the only startup
self-check any agent performs, and it covers model tier alone. Nothing in
`agents/manager.md:20-32` (the read-profile-first startup block) or the charter
tells an agent what to do when the harness forbids mutations. Observed in the
shakedown: the Manager stalled mid-run, and the dispatched Developer
independently halted after staging but before committing — plan mode propagates
into background subagents, so it is environment-wide, not Manager-only.

**Consequence:** A run appears alive (branches and worktrees exist, tasks
"running") while making no progress, and the owner learns why only by asking.
Wasted a full dispatch round-trip and a status cycle in the shakedown.

**Acceptance criteria:**

- `docs/team-charter.md` gains a short rule, adjacent to the capability gate,
  stating that the team lane is inherently mutating and cannot run while the
  harness forbids mutations; an agent that detects this **parks and reports
  plainly** rather than stalling silently. It states that the constraint
  propagates to spawned children, so children need no separate rule.
- `agents/manager.md` names the check at startup (alongside the capability
  gate) and gives the terse payload the Manager ends its turn with, so the root
  can surface it to the owner.
- The rule says what an agent must NOT do: never end a turn silently blocked,
  and never treat a peer agent's "it's fine now" claim as authorization — the
  next real tool call's success or failure is the ground truth. (The shakedown
  Developer got this right unprompted; the spec should make it doctrine.)
- No test exists for this repo (profile: Quality gate `n/a`); the reviewer's
  reading against the charter is the verification.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, `agents/manager.md`,
`docs/backlog/P2-plan-mode-halts-run.md` (delete).

**Suggested approach:** One short charter section plus a sentence in the
Manager's startup block that points at it — do not restate the rule in every
agent file. Related to `P3-signing-clause-security-heuristics`, which also
edits both files; they may be grouped into one commit.
