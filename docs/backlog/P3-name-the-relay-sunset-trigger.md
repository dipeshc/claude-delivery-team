---
by: owner
---

# The relay sunset tags name no observable trigger

**What's wrong:** Two `DELETE WHEN` tags are conditioned on "notifications route
to between-turn parents" — a phrasing that describes a hoped-for harness
behaviour rather than anything a reader can check. Nobody can tell whether the
trigger has fired, so the tags cannot do the job the charter gives them.

**Evidence:** `agents/manager.md:253-254` and `skills/team/SKILL.md:221-224`
both carry that trigger. The charter's *Sunset-tagged workarounds* section
requires a tag so the rule is "removed the moment its cause is gone", and the
`consistency-check` skill hunts `DELETE WHEN` tags whose trigger has fired —
both depend on the trigger being decidable. The scar-tissue audit reviewed all
tags and left every one in place precisely because none could be evaluated from
repo artifacts.

**Consequence:** The relay duty and its watchdog framing outlive their cause by
default rather than by decision. Both are load-bearing today — the relay hop
cost one run twelve idle minutes of a landed commit sitting unreported — so the
cost of them ossifying is real, not cosmetic.

**Acceptance criteria:**

- Both tags name a **checkable** trigger. Claude Code agent teams is the
  concrete successor: teammates are independent sessions that message each
  other by name, so a child's payload reaches the Manager without passing
  through root. The trigger should be satisfiable by observation — e.g. that
  teams are generally available (not behind an experimental flag) and that a
  child's completion reaches its spawner directly.
- The two tags stay consistent with each other: same trigger, worded the same
  way, so one cannot fire while the other is missed.
- The rules themselves are unchanged — this item edits trigger wording only.
  Removing the relay duty is what the trigger authorises later, not now.
- Nothing in the framework is made to *depend* on agent teams: the plugin must
  keep working unchanged where teams are unavailable.
- No test exists for this repo (profile: Quality gate `n/a`); reviewer reading
  is the verification, including confirming both tags match verbatim.
- This item file is deleted by the merging commit.

**Exact files to change:** `agents/manager.md`, `skills/team/SKILL.md`,
`docs/backlog/P3-name-the-relay-sunset-trigger.md` (delete).

**Suggested approach:** A wording change to two blockquotes. Do not restate the
successor's mechanics in either file — name the observable condition and stop.
