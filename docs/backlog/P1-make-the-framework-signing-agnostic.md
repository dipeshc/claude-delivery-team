---
by: owner (owner ruling — re-decision, not a sharpening)
---

# Make the framework agnostic to commit signing

**Owner ruling, not a judgment call for the implementer:** commit signing is
**orthogonal** to what this framework does. Many projects that use this plugin
will not sign at all. The framework must therefore have no opinion about
signing — it neither requires it, nor reports on it, nor works around it.

**What's wrong:** Signing is currently mandated by the generic core and cannot
be turned off. A consuming project that does not sign inherits rules it cannot
disable, and its agents will report `signed: false` on every commit as though
something were broken.

**Evidence:** `docs/team-charter.md` § "Signing fallback" mandates signing, a
retry, a `--no-gpg-sign` fallback and a re-sign list; `agents/manager.md:37` and
`:302` require a "mechanically-derived unsigned-SHA re-sign list" in every status
report; `agents/merge-clerk.md:106-119` makes `cherry-pick -S` mandatory;
`agents/developer.md` and `agents/qa.md` each restate the fallback; the message
schemas carry `signed:` fields; `skills/team/SKILL.md:216-226` has the heartbeat
aggregate "unsigned SHAs to re-sign". Neither
`docs/project-profile.template.md` nor any profile has a signing section, so
none of this is configurable. That makes signing project-specific policy living
in the generic core — a direct violation of `docs/architecture.md` invariant 6
("no project-specific content… everything project-specific lives in the
consuming repo's profile").

**Consequence:** The framework imposes one project's git policy on every project
that installs it. It also creates a standing conflict with owner-level rules that
forbid `--no-gpg-sign`: in this repo's own run a Merge-Clerk blocked twice on
exactly that contradiction, and the deadlock was only breakable by the owner
fixing the underlying signing fault.

**Acceptance criteria:**

- **Signing rules are removed from the generic core**, not made configurable.
  Do **not** add a signing section to the profile template: asking every project
  to declare a signing stance is still the framework having an opinion. Agents
  simply run `git commit` and whatever the repo's own git configuration does
  (`commit.gpgsign`, `gpg.format`, …) happens invisibly.
- **No agent reports signedness.** The `signed:` fields leave the message
  schemas, the re-sign list leaves the Manager's status report and the team
  skill's heartbeat prompt, and no agent derives, aggregates, or mentions
  unsigned SHAs. `%G?` should not appear anywhere in the framework.
- **A signing failure is an ordinary tool failure.** If a commit fails because
  the signer is unavailable, the agent reports it like any other failed command
  and stops — it does not retry-then-bypass, and it never passes
  `--no-gpg-sign` on its own initiative. Removing the sanctioned bypass is the
  point: the framework should not carry a mechanism for silently disabling a
  check the owner's environment imposes.
- **Preserve one property, restated without mandating signing:** a rebase
  performed by the framework must not silently produce a *weaker* commit than
  the author's original. Today `agents/merge-clerk.md` achieves this with a
  mandatory `-S`. **Determine empirically** whether `git cherry-pick` honours a
  repo's configured `commit.gpgsign` (test it in a scratch repo with signing
  configured and working — do not reason about it from documentation), then:
  if it does, drop `-S` entirely and say the repo's config governs; if it does
  not, keep whatever minimal mechanism preserves the repo's *configured*
  behaviour, phrased as "do not degrade what the author produced" rather than as
  a signing requirement. **State which you found true and how you tested it.**
- The `git range-diff` patch-identity proof, the ff-only landing, the
  single-writer rule and every other pipeline invariant are **untouched** — this
  item removes signing policy only.
- `docs/architecture.md` is checked and updated if it references signing as a
  framework property.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading is
  the verification. The reviewer should grep the whole repo for
  `sign|%G\?|gpg` afterwards and confirm only genuinely repo-agnostic mentions
  remain.
- This item file is deleted by the merging commit.

**Exact files to change:** `docs/team-charter.md`, `agents/manager.md`,
`agents/merge-clerk.md`, `agents/developer.md`, `agents/qa.md`,
`skills/team/SKILL.md`, `skills/consistency-check/SKILL.md`, possibly
`docs/architecture.md`, `docs/backlog/P1-make-the-framework-signing-agnostic.md`
(delete).

**Suggested approach:** Cross-cutting but mechanical: signing rules come out,
one correctness property is restated environment-neutrally. This **supersedes**
`P2-resign-audit-recipe-unsafe` — that item fixes an audit recipe this one
deletes outright, so whichever lands second must delete the other's item file
too, or the Manager should assign both together. Land this one first if they are
not grouped.
