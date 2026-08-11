# Project profile — `<project name>`

Copy this file to `<repo>/.claude/project-profile.md` and fill it in. The
global delivery machinery (the delivery-team plugin: its charter, skills,
and agents) is **generic and identical for every project**; this
file is the *only* place a project's own specifics live. Every global agent
reads this file first and treats it as authoritative for its project.

A section left as `n/a` means "this project has no such requirement" — that is
a valid, complete answer, not an omission. An agent must never invent a rule
for a section marked `n/a`, and must never assume a convention from some other
project applies here.

---

## Repo

- **Root path:** `/absolute/path/to/repo`
- **Default branch:** `main`
- **Package manager / runtime:** e.g. `pnpm` + Node 22 / `uv` + Python 3.14
- **Layout:** the packages/apps that exist and what each is, one line each.

## Quality gate

The exact commands that must pass before any change is submitted. Give the
scoped (per-package) form and the full form separately — scoped is the default
for a normal change; full is for cross-cutting work and the QA loop.

- **Scoped:** e.g. `pnpm --filter <pkg> typecheck && pnpm --filter <pkg> test`
- **Full:** e.g. `pnpm typecheck && pnpm test && pnpm build`
- **Notes:** anything load-bearing — tests that need a display, a container, a
  network service, or that are known-slow.

## Backlog

- **Location:** e.g. `docs/backlog/` (per-item files, priority in the filename)
- **Conventions doc:** e.g. `docs/backlog.md`, or `n/a` if the convention is
  just "one file per item."
- **Item lifecycle:** how an item is marked done (e.g. "the merging commit
  deletes the item file"), or `n/a` if the project uses an external tracker.

## Specification source of truth

- **Docs are spec?** yes / no. If yes: which directory, and what a doc-vs-code
  mismatch means (which side is wrong).
- **Docs root:** e.g. `docs/`, or `n/a`.
- **Decision records:** e.g. `docs/decisions/` (ADRs), or `n/a`.
- **Conventions doc:** where this repo's code style, commit-message format, and
  naming conventions live (e.g. `docs/development/conventions.md`), or `n/a` if
  the convention is simply "match the surrounding code and existing git log."

## Repo-wide invariant guards

Tests that are **repo-wide invariants no per-package scope covers** — so
scoped verification silently skips them. Keep this list short and named; each
entry says what triggers it.

- `path/to/guard.test.ts` — what it guards, and when it MUST be run (e.g.
  "whenever a change adds or edits test fixtures or sample data").
- `n/a` if the project has none.

## QA watch loop

- **Runner script:** the project's own watch-and-test script (e.g.
  `scripts/qa/watch-and-test.sh`) — a shell loop that pulls each new
  default-branch tip, runs the **full** gate, and wakes the QA agent only on a
  new failure or a green heartbeat. Per-project by nature (paths, test
  commands, log parsing); the loop's *design* and exit-code contract are in the
  QA agent definition.
- **Known-failures file:** where already-known/accepted failures are recorded so
  the loop wakes on *new* ones only (e.g. `scripts/qa/known-failures.txt`).
- `n/a` if this project has no watch loop — QA then runs the full gate on
  demand instead.

## UI surfaces

For visual-QA sweeps (the `ui-inspector` agent). Omit or mark `n/a` for a
project with no UI.

- **How to boot the app locally:** the exact command, plus the URL/port.
- **Routes to sweep:** the significant routes, or where the router is defined
  so they can be derived.
- **Viewports:** the widths that matter (e.g. mobile 390, tablet 834, desktop
  1440).
- **Roles / auth states:** the distinct user roles or signed-in/out states that
  change what renders.
- **Themes:** which themes exist (light, dark, both) — an agent must not judge
  contrast against a theme the app doesn't implement.

## Exploration log

- **Location:** where feature-exploration spikes are recorded (e.g.
  `docs/exploration.md`), or `n/a` to default to `<docs-root>/exploration.md`.

## Cross-surface parity

If a user-facing change must land on several clients/platforms to be complete:

- **Surfaces:** e.g. web, iOS, Apple TV.
- **Parity ledger:** where an intentional divergence is recorded.
- **Backstop:** the script/test that enforces it.
- `n/a` if the project has a single surface.

## Sanctioned direct-write paths

Paths the root/orchestrating instance may commit to directly, outside the
normal review pipeline (everything else lands via the pipeline).

- e.g. `docs/backlog/`, `scripts/qa/`, `.claude/`
- `n/a` if every change goes through the pipeline.

## Worktree layout

- **Where worktrees are created:** e.g. `<repo>/.claude/worktrees/<name>/`
- **Branch naming:** e.g. `item/<slug>`
- **Landing style:** e.g. fast-forward only, one commit per item.

## Project-specific content rules

Any rule about what may or may not appear in code, docs, tests, or commit
messages that is unique to this project (naming conventions, forbidden
vocabulary, licensing/attribution requirements, placeholder conventions).

- `n/a` if none.

## Verification environment

- Anything an agent needs to know to run this project locally: required
  services, mounted media, credentials that exist vs. are absent, and which
  test layers can/can't be run in the current environment.
- `n/a` if a plain checkout runs everything.
