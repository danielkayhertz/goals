---
name: goals-schedule
description: >
  Quickly add a standalone to-do to a future week file without setting up a full
  project. Use for one-off tasks with no project parent. For project-linked
  milestones, use /goals-new-project instead. Triggers on: "add a to-do for
  [week/date]", "remind me to do X on [date]", "schedule a task", /goals-schedule
triggers:
  - add a to-do for
  - remind me to do
  - schedule a task
  - schedule this for
  - /goals-schedule
---

# Schedule a Task

This skill is for standalone to-dos with no project parent. For project-linked milestones, use /goals-new-project instead.

## Step 1: Parse the request

From the user's message, extract:
- Task text: what needs to be done
- Target date or week: when (e.g. "next week", "May 15", "week 18", "in 3 weeks")
- Context tag: work / home / both (default: both if not specified)

If task text or target date is unclear, ask for clarification before proceeding. Wait for response.

## Step 2: Compute target week

Run Python to compute the ISO week from the target date:

```python
python3 -c "
from datetime import date, timedelta
today = date.today()

# --- set target_date based on user input ---
# Examples:
# 'next week'   -> target_date = today + timedelta(weeks=1)
# 'in 3 weeks'  -> target_date = today + timedelta(weeks=3)
# 'next month'  -> target_date = today + timedelta(weeks=4)
# 'May 15'      -> target_date = date(today.year, 5, 15)
# 'week 18'     -> target_date = date.fromisocalendar(today.year, 18, 1)

target_date = today + timedelta(weeks=1)  # replace with computed value
iso = target_date.isocalendar()
print(f'{iso[0]:04d}-W{iso[1]:02d}')
print(target_date.isoformat())
" 2>/dev/null || python -c "
from datetime import date, timedelta
today = date.today()
target_date = today + timedelta(weeks=1)  # replace with computed value
iso = target_date.isocalendar()
print(f'{iso[0]:04d}-W{iso[1]:02d}')
print(target_date.isoformat())
"
```

Output: YYYY-Www label (line 1) and the target date in ISO format (line 2).

If Python is unavailable, use the `currentDate` context variable and compute the ISO week manually: find Monday of the target week, then use ISO 8601 week numbering (week 1 = week containing the first Thursday of the year).

## Step 3: Read or create the target week file

Read `~/Documents/Claude Code/goals/weekly/[WEEK].md` if it exists.

If it doesn't exist, create it with this structure:

```
---
context: both
week: [WEEK]
updated: [TODAY]
---

## Priorities

## Daily Log
```

## Step 4: Duplicate check

Scan `## Priorities` for any line containing the exact task text. If already present, tell user: "Already scheduled for [WEEK]" and stop.

## Step 5: Add the task

Append to `## Priorities` (after the last existing item, before `## Daily Log`):

```
- [ ] [TASK TEXT] [[CONTEXT]] [scheduled]
```

- If `## Priorities` is empty, place the line directly under the heading.
- Replace `[CONTEXT]` with `work`, `home`, or `both` based on what the user specified. Default to `both` if unspecified.
- The `[scheduled]` tag distinguishes these from project milestone stubs (which use `[stub-id:...]`).

## Step 6: Confirm

Tell user: "Added to [WEEK]. Use /goals-upcoming to see all scheduled items."
