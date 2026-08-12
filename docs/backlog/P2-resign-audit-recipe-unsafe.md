---
by: owner (run finding — reproduced by the Merge-Clerk and by root)
---

# The charter's re-sign audit recipe fabricates unsigned commits

**What's wrong:** The charter prescribes a command for deriving the unsigned-SHA
list that produces false positives in any repo where `log.showSignature=true`.
The spec's own mechanical rule is unsafe.

**Evidence:** `docs/team-charter.md` § Signing fallback prescribes
`git -C <repo> log --format='%H %G?' <range>`, taking the `N` SHAs as the
unsigned set. With `log.showSignature=true` (set in this repo — confirm with
`git config --get log.showSignature`), git interleaves human-readable
`Good "git" signature for …` banner lines into the format stream. A Merge-Clerk
applying the natural inverse filter (`grep -v '^G$'`) counted every banner as a
violation and reported **13 unsigned commits in a repo with zero**. Re-running
with `--no-show-signature` returns `13 G`.

**Consequence:** A fabricated re-sign list sends the owner to "fix" signatures
that are already good, and — worse in the other direction — a reader who learns
the audit is noisy stops trusting it, which is exactly when a real unsigned
commit slips through. The charter calls this list "derived mechanically, never
hand-accumulated", so it is relied on precisely because it is supposed to be
trustworthy.

**Acceptance criteria:**

- Every prescribed signature-audit command in the spec passes
  `--no-show-signature` (or an equivalent that provably suppresses banner
  interleaving), so the output is machine-parseable regardless of the repo's
  `log.showSignature` setting.
- The charter states *why* the flag is mandatory in one clause — a reader who
  drops it must understand what breaks.
- Grep the whole repo for other `%G?` / `--format` audit recipes (agent files
  as well as the charter) and fix every instance; the requirement is that no
  surviving recipe is unsafe, not that one is patched.
- No behaviour change to the signing policy itself: the fallback, the reporting
  obligation, and the owner-owned re-sign remain exactly as they are.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification. The reviewer should run both command forms in this repo
  and confirm they now agree.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, plus any agent file carrying
its own copy of the recipe, `docs/backlog/P2-resign-audit-recipe-unsafe.md`
(delete).

**Suggested approach:** A sharpening of an existing rule, not a policy change.
Shares `docs/team-charter.md` with `P2-branch-delete-after-rebase-landing` and
`P3-dangling-doc-references`; group as the Manager sees fit.
