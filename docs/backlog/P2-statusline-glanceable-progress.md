---
by: owner
---

# Ship a glanceable statusline for team runs, with a one-time installer skill

**What's wrong:** There is no always-on, in-terminal view of a run. The
statusline is Claude Code's surface for exactly this — multi-line, refreshed on
events plus an optional `refreshInterval` timer whose documented purpose is
"background subagents change git state while the main session is idle" — but
plugins cannot ship a statusline: it is user-level `settings.json`
configuration, so without an installer the capability stays theoretical.

**Evidence:** `docs/backlog/P2-progress-board-and-state-ledger.md` (the state
ledger this renders; its schema is owned there); `skills/team/SKILL.md:96` (the
spawn step whose heartbeat is currently the only push channel to the owner).
Statusline facts verified against the Claude Code statusline documentation:
stdin JSON carries `workspace.project_dir`; scripts may print multiple lines;
`refreshInterval` minimum is 1 s; updates are debounced at 300 ms.

**Consequence:** Between heartbeats the owner has no signal at all without
asking. The cheapest possible surface — two lines at the bottom of the terminal
they are already looking at — goes unused.

**Constraints (owner rulings — do not re-decide):**

- **Shell-only.** No Node, no Python, no jq dependency unless it degrades
  gracefully when absent. The script may use git.
- **Silent outside team projects.** The script receives
  `workspace.project_dir` on stdin; when
  `<project_dir>/.claude/team-progress/state.js` is absent it prints nothing
  team-related (and ideally defers to whatever statusline the user already
  had — see acceptance).

**Acceptance criteria:**

- **The script.** A POSIX-shell statusline script ships as a plugin asset. When
  team state exists it renders up to two lines: (1) aggregate — items
  done/total with a bar, counts by stage, default-branch tip, ahead-of-origin
  marker, run elapsed; (2) the most attention-worthy detail — the item in
  review or blocked, with agent, model, and stage elapsed. It reads the state
  file owned by `P2-progress-board-and-state-ledger` and **does not** define
  its own schema. When the state is stale (`generatedAt` > ~90 s) it says so
  rather than rendering stale numbers as live.
- **The installer.** A `/delivery-team:statusline` skill copies the script to a
  stable user-level path (the plugin cache path changes per version, so
  settings must not point into the cache) and wires `statusLine` in
  `~/.claude/settings.json` with a small `refreshInterval`. If a statusline is
  already configured, it does not clobber it: it offers to chain (run the
  existing command and append the team lines) or to decline, and says exactly
  what it changed. Uninstall is documented in the same skill.
- **Degradation:** no state file → non-team output only (or empty); malformed
  state → empty rather than an error line; the script completes fast enough
  for a 300 ms-debounced caller (no network, at most one cheap git read).
- **Spec ships with the change:** `docs/architecture.md`'s component table
  gains the row; the README's "going deeper"/usage area mentions the installer
  in one line; the profile template needs nothing (the surface is user-level,
  not per-project).
- This item file is deleted by the merging commit.

**Exact files to change:** new plugin assets (e.g. `assets/statusline.sh`,
`skills/statusline/SKILL.md`), `docs/architecture.md`, `README.md`,
`docs/backlog/P2-statusline-glanceable-progress.md` (delete).

**Suggested approach:** Sequenced **after** `P2-progress-board-and-state-ledger`
— it consumes that item's schema and must not co-define it. The two lines in
the board item's mockup are the target rendering; fidelity to that shape
matters less than the silent/stale/absent behaviour, which is where a
statusline becomes trusted or ignored.
