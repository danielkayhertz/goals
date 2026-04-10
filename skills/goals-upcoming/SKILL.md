---
name: goals-upcoming
description: >
  Show upcoming due dates and overdue items across all projects and weeks.
  Scans 4 weeks back for overdue/unchecked items and 12 weeks forward for
  milestones. Triggers on: "what's coming up", "what's due", "upcoming
  deadlines", "what's behind schedule", /goals-upcoming
triggers:
  - what's coming up
  - what's due
  - upcoming deadlines
  - what's behind schedule
  - overdue
  - /goals-upcoming
---

# Upcoming & Overdue

## Step 1: Compute date windows

Run:

```bash
python3 -c "
from datetime import date, timedelta
t = date.today()

weeks = []
for offset in range(-4, 13):
    d = t + timedelta(weeks=offset)
    iso = d.isocalendar()
    weeks.append((offset, f'{iso[0]}-W{iso[1]:02d}', d.isoformat()))

for offset, label, sample_date in weeks:
    print(f'{offset:+3d}  {label}  ({sample_date})')

print(f'today={t.isoformat()}')
" 2>/dev/null || python -c "
from datetime import date, timedelta
t = date.today()

weeks = []
for offset in range(-4, 13):
    d = t + timedelta(weeks=offset)
    iso = d.isocalendar()
    weeks.append((offset, f'{iso[0]}-W{iso[1]:02d}', d.isoformat()))

for offset, label, sample_date in weeks:
    print(f'{offset:+3d}  {label}  ({sample_date})')

print(f'today={t.isoformat()}')
"
```

This gives you week labels from 4 weeks ago to 12 weeks ahead. If Python is unavailable, use the `currentDate` context variable to compute approximate dates manually.

## Step 2: Check for a context filter

If the user specified "work" or "home" in their request, note the active filter. Apply it throughout: show an item if its `context` field equals the requested filter OR equals `"both"`. If no filter was requested, show all items.

## Step 3: Scan past weekly files for overdue items

For each of the 4 past week labels (offset -4 through -1) and the current week (offset 0):
- Try to read `C:/Users/bpi/Documents/Claude Code/goals/weekly/[WEEK_LABEL].md`
- If the file does not exist, skip it silently
- For past weeks (offsets -4 through -1): extract all `- [ ]` (unchecked) lines from `## Daily Log` and `## Priorities` sections
- For the current week (offset 0): extract all `- [ ]` lines from `## Priorities` only (not daily log — today's log items are not overdue yet)
- Record each unchecked item with its week label and apply the context filter if active

## Step 4: Scan project files for upcoming milestones

Read `C:/Users/bpi/Documents/Claude Code/goals/projects/INDEX.md`. If this file is missing, note "Project index not found" and skip to Step 5.

For each active project (status = active) listed in the index:
- Read `C:/Users/bpi/Documents/Claude Code/goals/projects/[slug].md`
- If the file does not exist, skip it silently
- Extract all rows from the `## Milestones` table where status = `pending` or `in-progress`
- Record: milestone date, milestone text, project name, and context field if present
- Apply the context filter if active

## Step 5: Categorize items

Sort all items into buckets based on their date or week label:

- **Overdue**: milestones with date < today (status not complete) + unchecked tasks from past week files (offsets -4 through -1)
- **This week**: milestones with date falling in the current ISO week (offset 0)
- **Next 4 weeks**: milestones with date in weeks +1 through +4
- **Weeks 5–12**: milestones with date in weeks +5 through +12

Within each bucket, sort by date ascending.

## Step 6: Output

Format the output as follows (do not wrap the output in a fenced code block):

## Upcoming & Overdue — [TODAY'S DATE]

If a context filter is active, add a line: Showing: [work/home] items only.

### Overdue

List each overdue milestone as:
  [Project name] — [Milestone text] (was due [date])

List each unchecked weekly task as:
  [Week label] — unchecked: [task text]

If nothing is overdue, write: Nothing overdue ✓

### This Week ([CURRENT WEEK LABEL])

List milestones due this week, one per line:
  [Project name] — [Milestone text] ([date])

If nothing is due this week, write: Nothing due this week.

### Next 4 Weeks

Group by week label, sorted by date. Format:

[WEEK LABEL]
  [Project name] — [Milestone text] ([date])

If nothing is due in the next 4 weeks, write: Nothing due in the next 4 weeks.

### Weeks 5–12

Group by week label, sorted by date. Format:

[WEEK LABEL]
  [Project name] — [Milestone text] ([date])

If nothing is due in weeks 5–12, write: Nothing due in weeks 5–12.
