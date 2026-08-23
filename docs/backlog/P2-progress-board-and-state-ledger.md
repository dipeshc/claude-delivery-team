---
by: owner
---

# Ship a visual progress board: a Manager-written state ledger plus a zero-runtime browser page

**What's wrong:** A run's only progress surfaces are prose status messages and
the harness's subagent tree, which the owner finds hard to keep tabs on. The
Manager already holds everything a visual board needs — its in-context ledger
(`agents/manager.md:201-203`: one row per item — slug, branch, worktree, agent,
model, state, stage timestamps) and its git reconciliation — but renders it only
as markdown tables pushed ~10-minutely (`agents/manager.md:309`). Nothing
machine-readable leaves the Manager, so nothing can render continuously.

**Evidence:** `agents/manager.md:201-203` and `:309` as above;
`skills/team/SKILL.md:96` (the spawn step that would open the board);
`docs/team-charter.md:250` (the Manager's scoped-writers entry, which the state
ledger extends); `docs/project-profile.template.md:122` (Worktree layout — the
sibling section pattern the new path declaration follows).

**Consequence:** The owner watches a delivery run through 10-minute prose
deltas or by asking. Progress that the system already knows is invisible
between reports, and a stalled run looks identical to a thinking one from the
outside.

**Constraints (owner rulings — do not re-decide):**

- **Zero-runtime floor.** Claude Code's native install guarantees no Node and
  no Python — only a shell and git, which are already this framework's
  substrate. The board must work with nothing else: a **static** HTML file
  opened over `file://`, no server, no package install.
- **Single writer, declared.** The Manager is the sole writer of the state
  ledger, as an explicit extension of its charter scoped-writers entry
  (runtime housekeeping, never committed). This is a writer-list change; the
  owner has commissioned it in this item, so implement it — do not re-escalate.

**Acceptance criteria:**

- **The state ledger.** The Manager writes
  `<repo>/.claude/team-progress/state.js` — a single statement,
  `window.TEAM_STATE = {…};` — atomically (write temp, `mv`) on **every
  reconcile/poll**, not on the report cadence. The schema is specified in the
  spec (not only in the Manager file) and carries at minimum: `generatedAt`
  (epoch), run args, per-item rows (slug, priority, stage, agent+model, branch,
  head, stage-entered-at), agent rows (role, state, last-liveness-signal), the
  default-branch tip and ahead-of-origin count, and a bounded event list
  (latest ~20 payload relays/merges). A `state.json` twin may be written for
  non-browser consumers; if so, `state.js` is derived from it mechanically.
- **The page.** A self-contained `dashboard.html` asset ships in the plugin and
  is **copied** (not referenced) into `.claude/team-progress/` beside
  `state.js` — co-location is what makes the `file://` script-tag reload work.
  It polls by re-injecting `<script src="state.js?t=<now>">` every few seconds
  (no `fetch` — CORS blocks it on `file://`), renders the pipeline board
  (columns matching the item lifecycle), agent rail, and event feed, sets
  `document.title` from live state, and shows an explicit staleness banner when
  `generatedAt` is older than ~90 s — a dead Manager must be visible, not a
  frozen page. Both light and dark themes.
- **Discovery.** (1) The team skill's spawn step writes the page if missing and
  opens it — `open` / `xdg-open` / `start` by platform, shell-only — at most
  once per run; (2) a `/delivery-team:board` skill opens it on demand, creating
  it if needed; (3) the Manager's status reports and the heartbeat prompt
  include the page's absolute path so terminal renderings are clickable.
- **Spec ships with the change:** the charter's Manager scoped-writers entry
  names the progress-ledger path; `docs/architecture.md`'s component table
  gains the board's row; the profile template documents the path under its own
  section with `n/a` semantics for projects that want it off.
- **Degradation is specified:** no state file → the board skill still opens a
  page that says no run is active; the page never errors on absent/partial
  state.
- Works with zero network access; no external assets; honours the repo's
  content rules (no project references, timeless prose).
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/manager.md`, `skills/team/SKILL.md`,
`docs/team-charter.md` (Manager entry), `docs/architecture.md`,
`docs/project-profile.template.md`, new plugin assets (the dashboard HTML and
the board skill — e.g. `assets/dashboard.html`, `skills/board/SKILL.md`),
`docs/backlog/P2-progress-board-and-state-ledger.md` (delete).

**Suggested approach:** The mechanism is settled above; the craft is in the
schema and the page. Keep the schema small enough that the Manager's write is
one shell heredoc from data it already holds. Related:
`P2-statusline-glanceable-progress` consumes the same state file — land this
item first so the schema has one owner, and do not let the two items define it
twice.
