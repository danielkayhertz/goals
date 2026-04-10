---
name: goals-summary
description: >
  Show a read-only orientation summary of goals and active work: strategic pillars,
  active quarterly goals, this week's priorities, and active projects. Use to get
  a quick overview of what you're working on. Triggers on: "show my goals",
  "what am I working on", "goals summary", "what are my goals", /goals-summary
triggers:
  - show my goals
  - what am I working on
  - goals summary
  - what are my goals
  - my orientation
  - goals orientation
  - /goals-summary
---

# Goals Summary

You are generating a read-only orientation summary. **Do not modify any files.**

## Step 1: Get current date context

Run:
```bash
python3 -c "
from datetime import date
t = date.today()
iso = t.isocalendar()
q = (t.month - 1) // 3 + 1
print(f'today={t.isoformat()}')
print(f'iso_week={iso[0]}-W{iso[1]:02d}')
print(f'quarter={t.year}-Q{q}')
" 2>/dev/null || python -c "
from datetime import date
t = date.today()
iso = t.isocalendar()
q = (t.month - 1) // 3 + 1
print(f'today={t.isoformat()}')
print(f'iso_week={iso[0]}-W{iso[1]:02d}')
print(f'quarter={t.year}-Q{q}')
"
```

If the command fails, use the `currentDate` context variable if available (format: YYYY-MM-DD) to derive iso_week and quarter manually. If date context is unavailable, state this and skip quarterly/weekly sections.

## Step 2: Check for a context filter

Check whether the user specified "work" or "home" in their request. Note the filter if present.
**Filtering rule:** Show items where `context == requested_filter OR context == "both"`.
If no filter specified, show everything.

## Step 3: Read available files

Attempt to read each file. If a file does not exist, note it as missing — do not error.

1. `~/Documents/Claude Code/goals/pillars.md`
2. `~/Documents/Claude Code/goals/quarterly/[QUARTER].md` (use quarter value from Step 1)
3. `~/Documents/Claude Code/goals/weekly/[ISO_WEEK].md` (use iso_week value from Step 1)
4. `~/Documents/Claude Code/goals/projects/INDEX.md`

## Step 4: Format and output the summary

Format your output like this:

## Goals Summary — [TODAY]
[If context filter active: "(Showing: [work/home] items only)"]

### Strategic Pillars
[For each pillar from pillars.md ## Pillars section:]
- **[Pillar Name]** [context tag] — [description]
[If pillars.md missing or empty:] "No pillars defined yet. Run /goals-onboarding to set up."

### This Quarter ([QUARTER])
[For each goal in the quarterly file's ## Goals section, grouped by pillar, filtered by context:]
**[Pillar Name]**
- [ ] or [x] [Goal text]
[If file does not exist, or exists but has no items under ## Goals:] "No quarterly goals yet. Run /goals-quarterly."

### This Week ([ISO_WEEK])
[List all items from the weekly file's ## Priorities section, filtered by context:]
- [ ] or [x] [Priority text]
[If file does not exist, or exists but has no items under ## Priorities:] "No weekly plan yet. Run /goals-weekly."

### Active Projects
[Render the INDEX.md table rows where status = active, filtered by context. Apply the same context filtering rule from Step 2: show rows where context column matches requested filter, or equals 'both'.]
| slug | name | context | status | quarterly-goal |
|------|------|---------|--------|----------------|
[rows...]
[If no active projects:] "No active projects. Run /goals-new-project to set one up."

Apply context filtering to all four sections when a filter is active.
