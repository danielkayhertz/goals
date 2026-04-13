# Goals Skills — Personal Project Management System

A suite of 11 Claude Code skills for tracking goals at every cadence, from daily to quarterly. Covers both work and home contexts in a single unified system. Includes end-of-day and end-of-week reflection workflows and a P1/P2/P3 priority system with soft limits.

---

## Quick Start

Run `/goals-onboarding` (or say "set up goals") once to initialize the system. It will interview you for your strategic pillars and create the data directory.

After onboarding, the recommended first-session flow:
1. `/goals-quarterly` — set this quarter's goals per pillar
2. `/goals-weekly` — set this week's priorities
3. `/goals-daily` — plan today

---

## Skills

### Setup & Overview

**`/goals-onboarding`**
Run once to set up the system. Creates the `goals/` directory, interviews you for 2–4 strategic pillars, and initializes the project index.
- Triggers: "set up goals", "initialize goals system", "goals onboarding"

**`/goals-summary`** *(read-only)*
Orientation snapshot: your pillars, active quarterly goals, this week's priorities, and active projects. Good for starting a session or reorienting mid-week.
- Triggers: "show my goals", "what am I working on", "goals summary", "orientation"

---

### Check-ins

**`/goals-daily`**
Plan your day. Opens today's section in the current week file. Full mode interviews you (top priorities, carryover from yesterday) and tags each task `[P1]` or `[P2]`, with a soft warning if you exceed 3 P1s. Surfaces carry-forwards from yesterday's reflection. Quick mode accepts a single line. Defaults to work context; include "home" or "both" to override.
- Triggers: "plan my day", "daily check-in", "what should I focus on today"
- Quick mode: include "quick", "just log", "briefly", or "one-line" in your request

**`/goals-daily-review`**
End-of-day review. Shows today's unchecked tasks numbered (filtered to active context), marks done/not-done in a single reply, then asks for a takeaway and carry-forward. Appends a `#### Reflection` block to today's log entry. Defaults to work context; include "home" or "both" to override.
- Triggers: "daily review", "end of day", "how did today go"

**`/goals-weekly`**
Plan your week. Reviews last week's unchecked items, interviews you on what rolled over and what's new, tags priorities `[P1]`/`[P2]`, and writes a `## Priorities` section to the current week file. Warns if you exceed 5 P1s. Defaults to work context; include "home" or "both" to override.
- Triggers: "plan my week", "weekly review", "weekly check-in", "what are my priorities this week"

**`/goals-weekly-review`**
End-of-week reflection using the 4Ls framework (Liked, Learned, Lacked, Longed For). Reviews task progress for the week (filtered to active context), runs a structured 6-question interview, and writes a `## Weekly Reflection` section to the week file. Quick mode condenses to one question. Defaults to work context; include "home" or "both" to override.
- Triggers: "weekly review", "end of week", "week reflection", "how was my week"
- Quick mode: include "quick", "briefly", or "short" in your request

**`/goals-quarterly`**
Plan your quarter. Reviews last quarter's progress, then interviews you on goals for each pillar for the current quarter. Defaults to work context; include "home" or "both" to override. Handles update/start-fresh/add-to-them for existing quarter files.
- Triggers: "quarterly planning", "quarterly review", "set quarterly goals", "review my quarter"

---

### Upcoming & Scheduling

**`/goals-upcoming`**
Scans 4 weeks back for overdue/unchecked items and 12 weeks forward for upcoming milestones. Outputs a P1 summary block at the top (if any P1s exist), then four buckets: Overdue, This Week, Next 4 Weeks, Weeks 5–12. P1 items listed first within each section; P3 items counted not listed (except overdue P3s always show). Supports work/home filtering.
- Triggers: "what's due", "what's coming up", "upcoming deadlines", "what's behind schedule", "overdue"

**`/goals-schedule`**
Add a one-off to-do to a future week file without setting up a project. Tasks are tagged `[scheduled]` to distinguish them from project milestone stubs. Use `/goals-new-project` instead for project-linked milestones.
- Triggers: "schedule a task", "add a to-do for [week/date]", "remind me to do X on [date]", "schedule this for"

---

### Projects

**`/goals-new-project`**
Set up a new project workplan through a guided interview: name, context, description, goals, milestones with target dates, and quarterly goal linkage. After the interview it creates the project file, adds a row to the project index, and auto-plants milestone stubs in future week files. Idempotent — running again on the same project updates stubs in place without duplicating.
- Triggers: "set up a new project", "create a project workplan", "start a project", "new project"

**`/goals-project-review`**
Walk through a project's milestones and update statuses (done / in-progress / blocked / date-change / skip). Handles date changes by moving stubs to the new week. Marks projects complete or paused — which removes all pending stubs from future week files.
- Triggers: "review a project", "update project status", "check on project", "project review"

---

## Data Directory

All data files live at `C:\Users\bpi\Documents\Claude Code\goals\`:

```
goals/
├── pillars.md              # Strategic pillars (created by /goals-onboarding)
├── quarterly/
│   └── YYYY-Qn.md          # Quarterly goals, organized by pillar
├── weekly/
│   └── YYYY-Www.md         # Week priorities + daily log entries
└── projects/
    ├── INDEX.md            # Active project list (table: slug|name|context|status|quarterly-goal)
    └── [slug].md           # Project workplan (goals, milestones table, notes)
```

Weekly files use ISO 8601 week numbering (Monday = week start). Week labels look like `2026-W15`.

---

## Key Concepts

**Strategic pillars** — 2–4 stable focus areas that don't change quarter to quarter (e.g., "Research & Publications", "Home Projects"). They anchor all quarterly goals. Defined once in `pillars.md`; referenced by check-in skills.

**Context tags** — Every item is tagged `work`, `home`, or `both`. All check-in and review skills default to **work** — no need to say it. Include "home" or "both" when invoking to switch (e.g., `/goals-daily home`). Items tagged `both` appear under either filter. Untagged items always match any filter (backward-compat).

**Stub IDs** — Each project milestone gets a unique `stub-id` (e.g., `missing-middle-m2`). When `/goals-new-project` plants a milestone stub in a future week file, it tags the line `[stub-id:missing-middle-m2]`. This enables `/goals-project-review` to find and move or remove stubs when milestone dates change or projects complete.

**`[scheduled]` tag** — Tasks added by `/goals-schedule` are tagged `[scheduled]` instead of a stub-id, making them easy to distinguish from project milestones.

**Priority tags** — Task lines support optional `[P1]`, `[P2]`, or `[P3]` tags. Untagged items are treated as P2. `/goals-daily` and `/goals-weekly` ask for priority per task and warn at soft limits (3 P1s/day, 5/week). `/goals-upcoming` surfaces a P1 summary block at the top.

**Reflection blocks** — `/goals-daily-review` appends a `#### Reflection` block (takeaway + carry-forward) to today's log entry. The carry-forward surfaces automatically in tomorrow's `/goals-daily`. `/goals-weekly-review` appends a `## Weekly Reflection` section using the 4Ls framework.

---

## Phase 4 (Not Yet Built)

Deferred until the file structure is validated in real use:

- **Session-start hook** — injects a ~10-line summary (top pillar, quarterly goals, weekly priorities, overdue items) into every Claude Code session automatically
- **Scheduled reminders** — Monday morning triggers `/goals-weekly`; first day of quarter triggers `/goals-quarterly`; weekly `/goals-upcoming` digest
- **Mobile bot (Telegram)** — a lightweight server bot (e.g. Railway/Render) that allows check-ins from a phone when the PC is off. Architecture: Telegram → bot on server → pulls this GitHub repo → reads the relevant skill file as a system prompt → calls Claude API with `read_file`/`write_file` tools → commits and pushes changes back. Skills and data files both live in this repo, so skills stay in sync automatically. The bot exposes commands like `/daily`, `/weekly`, `/upcoming`. Main limitation vs. Claude Code: only tools explicitly defined in the bot are available (no agents, no spawning), but that's sufficient for read/write check-in workflows.
