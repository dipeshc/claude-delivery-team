---
by: owner
---

# Replace docs/backlog/.gitkeep with a README documenting the directory

**What's wrong:** `docs/backlog/` exists only as an empty directory held in git
by a `.gitkeep`; nothing in the directory itself tells a contributor what
belongs here or how items are shaped.

**Evidence:** `docs/backlog/.gitkeep` (zero-byte placeholder). The conventions
live only in `docs/team-charter.md` § "Backlog conventions" and
`.claude/project-profile.md` § "Backlog".

**Consequence:** someone filing an item cold has to discover the charter
section by accident; a wrongly-shaped item wastes a dispatch cycle.

**Acceptance criteria:**

- `docs/backlog/README.md` exists: a short note (≤ 15 lines of prose) that
  opens by saying it is documentation, not a work item; states one file per
  item with priority in the filename (`P0`–`P3`); names the `blocked/` and
  `deferred/` subdirectory meanings; and links to the team charter's "Backlog
  conventions" section for the full item shape. It points at the conventions —
  it does not restate or extend them.
- `docs/backlog/.gitkeep` is deleted in the same commit.
- This item file is deleted by the merging commit.
- No test suite exists in this repo (profile: Quality gate `n/a`); reviewer
  reading against the charter is the verification.

**Exact files to change:** `docs/backlog/README.md` (new),
`docs/backlog/.gitkeep` (delete), `docs/backlog/P3-backlog-dir-readme.md`
(delete).

**Suggested approach:** doc-only, single `docs(backlog): …` Conventional
Commit.

**Owner note (lane):** deliberately routed through the **team lane** as a
pipeline shakedown, despite being direct-lane-sized. Do not re-triage the lane.
