# Goals — Personal Project Management Skills for Claude Code

A suite of 9 Claude Code skills for structured goal tracking at every cadence, from daily to quarterly. Covers work and home contexts in a single unified system. Includes project workplanning with milestones, stub-based week-file integration, and an upcoming deadlines scanner.

Inspired by [Dex](https://github.com/davekilleen/Dex) and built skills-first (Phase 1). A session-start hook and scheduled reminders are planned for Phase 2 once the file structure is stable in real use.

---

## Installation

Clone this repo and copy the skill directories into your Claude Code skills folder:

```bash
git clone https://github.com/danielkayhertz/goals.git
cp -r goals/skills/goals-* ~/.claude/skills/
```

On Windows (PowerShell):
```powershell
git clone https://github.com/danielkayhertz/goals.git
Copy-Item -Recurse goals\skills\goals-* $env:USERPROFILE\.claude\skills\
```

Then open Claude Code and run `/goals-onboarding` to set up your data directory.

> **Note:** Skill files reference `~/Documents/Claude Code/goals/` as the data root. Before using, update this path in each SKILL.md to match your own username and preferred location.

---

## Quick Start

Run `/goals-onboarding` (or say "set up goals") once to initialize the system. It will interview you for 2–4 strategic pillars and create the data directory.

Recommended first-session flow after onboarding:
1. `/goals-quarterly` — set this quarter's goals per pillar
2. `/goals-weekly` — set this week's priorities
3. `/goals-daily` — plan today

---

## Skills

### Setup & Overview

**`/goals-onboarding`**
Run once to set up the system. Creates the `goals/` directory, interviews you for strategic pillars, and initializes the project index.
Triggers: `"set up goals"`, `"initialize goals system"`, `"goals onboarding"`

**`/goals-summary`** *(read-only)*
Orientation snapshot: your pillars, active quarterly goals, this week's priorities, and active projects.
Triggers: `"show my goals"`, `"what am I working on"`, `"goals summary"`, `"orientation"`

---

### Check-ins

**`/goals-daily`**
Plan your day. Full mode interviews you (top priorities, carryover); quick mode accepts a single line. Detects if today's entry already exists and offers to update it.
Triggers: `"plan my day"`, `"daily check-in"`, `"what should I focus on today"`
Quick mode keywords: `"quick"`, `"just log"`, `"briefly"`, `"one-line"`

**`/goals-weekly`**
Plan your week. Reviews last week's unchecked items, interviews you on rollover and new priorities, and writes a `## Priorities` section to the current week file.
Triggers: `"plan my week"`, `"weekly review"`, `"weekly check-in"`

**`/goals-quarterly`**
Plan your quarter. Reviews last quarter's progress, then interviews you on goals for each pillar. Handles update/start-fresh/add-to-them for existing quarter files.
Triggers: `"quarterly planning"`, `"quarterly review"`, `"set quarterly goals"`

---

### Upcoming & Scheduling

**`/goals-upcoming`**
Scans 4 weeks back for overdue/unchecked items and 12 weeks forward for milestones. Outputs four buckets: Overdue, This Week, Next 4 Weeks, Weeks 5–12.
Triggers: `"what's due"`, `"what's coming up"`, `"upcoming deadlines"`, `"what's behind schedule"`

**`/goals-schedule`**
Add a one-off to-do to a future week without setting up a project. Tasks are tagged `[scheduled]` to distinguish them from project milestone stubs.
Triggers: `"schedule a task"`, `"add a to-do for [date]"`, `"remind me to do X on [date]"`

---

### Projects

**`/goals-new-project`**
Set up a project workplan through a guided interview: name, context, description, goals, milestones with target dates, and quarterly goal linkage. Auto-plants milestone stubs in future week files. Idempotent — runs again without duplicating stubs.
Triggers: `"set up a new project"`, `"create a project workplan"`, `"start a project"`

**`/goals-project-review`**
Walk through a project's milestones: mark done/in-progress/blocked, change dates (stubs move automatically), or mark the project complete/paused (stubs removed from all future week files).
Triggers: `"review a project"`, `"update project status"`, `"check on project"`

---

## Data Directory Structure

Skills read and write markdown files in a shared data directory (default: `~/Documents/Claude Code/goals/`):

```
goals/
├── pillars.md              # Strategic pillars (2–4 focus areas)
├── quarterly/
│   └── YYYY-Qn.md          # Quarterly goals, organized by pillar
├── weekly/
│   └── YYYY-Www.md         # Week priorities + daily log entries
└── projects/
    ├── INDEX.md            # Project list (slug | name | context | status | quarterly-goal)
    └── [slug].md           # Project workplan (goals, milestones table, notes)
```

Weekly files use ISO 8601 week numbering (Monday = week start). No separate daily files — daily entries are sections within the week file.

---

## Key Concepts

**Strategic pillars** — 2–4 stable focus areas that don't change quarter to quarter (e.g., "Research & Publications", "Home Projects"). They anchor all quarterly goals.

**Context tags** — Every item is tagged `work`, `home`, or `both`. All check-in skills support filtering. Items tagged `both` appear under either filter.

**Stub IDs** — Each project milestone gets a unique `stub-id` (e.g., `missing-middle-m2`). When `/goals-new-project` plants a stub in a future week file, the line is tagged `[stub-id:missing-middle-m2]`. This lets `/goals-project-review` find and move or remove stubs when dates change or projects complete.

**`[scheduled]` tag** — Tasks added by `/goals-schedule` use `[scheduled]` instead of a stub-id, making them easy to distinguish from project milestones.

---

## Phase 2 (Planned)

- **Session-start hook** — injects a ~10-line summary (active quarterly goals, weekly priorities, overdue items) into every Claude Code session automatically
- **Scheduled reminders** — Monday morning triggers `/goals-weekly`; first day of quarter triggers `/goals-quarterly`
