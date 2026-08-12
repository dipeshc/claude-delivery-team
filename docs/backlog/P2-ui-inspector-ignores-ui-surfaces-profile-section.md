---
by: audit
---

# ui-inspector never consults the profile's `UI surfaces` section

**What's wrong.** The project-profile template ships a `UI surfaces` section
that exists "for visual-QA sweeps (the `ui-inspector` agent)" — boot command,
routes to sweep, viewports, roles/auth states, themes — but the ui-inspector
agent definition never reads it: it derives all of those itself and attributes
the boot instructions to a different section.

**Evidence** (verified by reading):

- `docs/project-profile.template.md:75-88` — the `UI surfaces` section declares
  boot command + URL/port, routes, viewports, roles/auth states, and themes,
  explicitly for the `ui-inspector` agent.
- `agents/ui-inspector.md:30-33` — the profile sections the agent says it will
  use are `Repo → Layout`, `Verification environment`, `Quality gate → Notes`,
  and `Backlog`; `UI surfaces` is absent.
- `agents/ui-inspector.md:87-99` — booting the app is sourced from the e2e
  harness / `Verification environment`, never from `UI surfaces → How to boot
  the app locally`.
- `agents/ui-inspector.md:125-136` — routes are derived from the router source,
  viewports are hardcoded (~390/768/1440px), roles are found in auth docs/role
  enums — with no mention of the profile fields that declare each.
- `agents/ui-inspector.md:79-83` — themes are established by reading the
  theme/palette source; the profile's `Themes` field is never mentioned.
- `grep -rn 'UI surfaces' agents skills docs` matches only the template: the
  section has zero consumers anywhere in the repo.

**Consequence.** An owner who fills in `UI surfaces` — the field the template
tells them configures visual QA — has it silently ignored: the agent re-derives
routes/roles (possibly missing ones the owner named), sweeps hardcoded viewports
instead of the declared ones, and the template documents a contract the
implementation does not honour. Per `docs/architecture.md`, a doc-vs-definition
mismatch means the definition is wrong.

**Acceptance criteria** (this repo's gate is n/a; reading is the verification):

- `agents/ui-inspector.md` names **UI surfaces** among the profile sections it
  reads first.
- Boot instructions prefer `UI surfaces → How to boot the app locally` where
  given, falling back to the existing e2e-harness/`Verification environment`
  path where not.
- Routes, viewports, roles, and themes are taken from the declared `UI surfaces`
  fields where present; the current derivation (router source, hardcoded
  viewport widths, auth docs, theme source) remains only as the fallback where
  a field is absent.
- The agent states what an `n/a`/omitted `UI surfaces` section means for it
  (per the template: the project has no UI), consistent with the charter's
  "never invent a rule for `n/a`" stance.
- No other file changes; `grep -n 'UI surfaces' agents/ui-inspector.md` has at
  least one hit afterwards.

**Exact files to change.** `agents/ui-inspector.md` only.

**Suggested approach.** Add `UI surfaces` to the sections list at the top, then
weave "profile first, derive as fallback" into §1 (boot) and §2 (the matrix:
routes, viewports, roles) and the themes paragraph — mirroring how
`agents/exploration.md` sources its exploration-log location from the profile
with an `n/a` default. Keep the derivation text: it is the correct behaviour
for a profile that leaves fields sparse.
