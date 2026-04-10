---
name: goals-project-review
description: >
  Review and update a project workplan: walk through milestones, update statuses,
  handle date changes, and mark projects complete or paused. Triggers on:
  "review a project", "update project status", "check on project", /goals-project-review
triggers:
  - review a project
  - update project status
  - check on project
  - project review
  - /goals-project-review
---

# Project Review

## Step 1: Get current date

Run to get today's ISO date:

```
python3 -c "from datetime import date; print(date.today().isoformat())" 2>/dev/null || python -c "from datetime import date; print(date.today().isoformat())"
```

If both fail, use the `currentDate` value from the system context.

## Step 2: Select a project

Read `C:/Users/bpi/Documents/Claude Code/goals/projects/INDEX.md`.

If the file is missing, tell user: "I couldn't find INDEX.md at C:/Users/bpi/Documents/Claude Code/goals/projects/INDEX.md. Please check that the goals system is set up."

If the user named a project in their request, match it by slug or name (case-insensitive). If no match is found, tell the user and list active projects from INDEX.md.

If no project was named, list all active projects from INDEX.md and ask: "Which project would you like to review?" Wait for response.

## Step 3: Read and display the project file

Read `C:/Users/bpi/Documents/Claude Code/goals/projects/[SLUG].md`.

If the file is missing, tell user: "I couldn't find a project file for [SLUG] at C:/Users/bpi/Documents/Claude Code/goals/projects/[SLUG].md."

Display:
- Project goals (from ## Goals section)
- Milestones table showing: date, milestone text, status, stub-id

For each milestone where date < today AND status is not "complete", flag it with a warning: "OVERDUE: [milestone text] was due [date]"

## Step 4: Walk through milestones ONE AT A TIME

For each milestone in the table, ask:

"[Milestone text] — target [date]. Status: [status]. Update? Reply with: done / in-progress / blocked / date-change / skip"

Wait for user's response before asking about the next milestone.

Handle each response:

**done** → set status = complete

**in-progress** → set status = in-progress

**blocked** → ask: "What's blocking it?" Wait for response. Append a note to ## Notes in the project file: "BLOCKED [stub-id] ([today's date]): [user's answer]"

**date-change** → ask: "New target date? (YYYY-MM-DD)" Wait for response. Then:
- Update the milestone date in the project file
- Compute the ISO week for the new date:
  ```
  python3 -c "from datetime import date; d = date.fromisoformat('YYYY-MM-DD'); iso = d.isocalendar(); print(f'{iso[0]}-W{iso[1]:02d}')" 2>/dev/null || python -c "from datetime import date; d = date.fromisoformat('YYYY-MM-DD'); iso = d.isocalendar(); print(str(iso[0]) + '-W' + str(iso[1]).zfill(2))"
  ```
  This prints the week label, e.g. `2026-W22`. The new week file path is: `C:/Users/bpi/Documents/Claude Code/goals/weekly/[WEEK_LABEL].md`
- Stub move: List all files in `C:/Users/bpi/Documents/Claude Code/goals/weekly/`. For each file, scan for any line containing `[stub-id:[STUB-ID]]`. If found, remove that line from the file. Then find or create the new week file and add the stub line under `## Priorities`:
  `- [ ] [Project Name] — [Milestone text] [stub-id:[STUB-ID]]`
  If the new week file doesn't exist, create it with a `## Priorities` section containing the stub line.

**skip** → leave unchanged, move to next milestone

## Step 5: Ask about overall project status

After all milestones are reviewed, ask:

"Is this project still active, or has its status changed? Reply with: active / paused / complete"

Wait for response.

If **complete** or **paused**:
- Update `status:` in the project file frontmatter to the new value
- Update the status column for this project's row in `C:/Users/bpi/Documents/Claude Code/goals/projects/INDEX.md`
- Stub removal: List all files in `C:/Users/bpi/Documents/Claude Code/goals/weekly/`. For each file, scan for any line containing `[stub-id:[SLUG]-` (prefix match that captures all milestones for the project). Remove every such line from every weekly file.

If **active**:
- No status change needed

## Step 6: Write updates

Write the updated project file with:
- Updated ## Milestones table (new statuses and/or dates)
- Any new notes appended to ## Notes
- `updated:` frontmatter set to today's date

## Step 7: Confirm

Tell user what was changed:
- Which milestones were updated and to what status
- Any milestone dates that changed
- Any stubs that were moved (old week → new week)
- Any stubs that were removed (if project is complete/paused)
- Current project status
