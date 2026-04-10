---
name: goals-new-project
description: >
  Set up a new project workplan through a guided interview. Creates a project
  file with goals and milestones, adds it to the project index, and auto-plants
  milestone stubs in future week files. Triggers on: "set up a new project",
  "create a project workplan", "start a project", /goals-new-project
triggers:
  - set up a new project
  - create a project workplan
  - start a project
  - new project
  - /goals-new-project
---

# New Project Setup

## Step 1: Get current date context

Run the following to get today's date and derive the current quarter:

```bash
python3 -c "
from datetime import date
today = date.today()
q = (today.month - 1) // 3 + 1
print(f'DATE={today}')
print(f'QUARTER={today.year}-Q{q}')
" 2>/dev/null || python -c "
from datetime import date
today = date.today()
q = (today.month - 1) // 3 + 1
print('DATE=' + str(today))
print('QUARTER=' + str(today.year) + '-Q' + str(q))
"
```

If Python fails entirely, use the `currentDate` context variable to determine today's date and compute the current quarter manually.

## Step 2: Read quarterly goals for reference

Check if `~/Documents/Claude Code/goals/quarterly/[QUARTER].md` exists and read it if so. Extract any listed goals — you will offer these as linkage options in the interview.

## Step 3: Interview — ONE QUESTION AT A TIME

Ask the following questions in order. After each question, wait for the user's response before moving to the next. Do not batch questions together.

1. Ask: "What's the name of this project?" — Wait for the user's response before continuing.

2. Ask: "Is this a [work], [home], or [both] project?" — Wait for the user's response before continuing.

3. Ask: "In 1–2 sentences, what is this project trying to accomplish?" — Wait for the user's response before continuing.

4. Ask: "What does 'done' look like for this project? What are 2–4 key goals?" — Wait for the user's response before continuing.

5. Ask: "What are the key milestones? For each, give me a description and a target date." Accept multiple milestones in a single response (e.g. "Data cleaning by May 1, draft by June 15"). — Wait for the user's response before continuing.

6. If quarterly goals were found in Step 2, show the list of quarterly goals, then ask: "Which quarterly goal does this project roll up into? (Or none)" — Wait for the user's response before continuing. If no quarterly file was found, skip this question and set quarterly-goal to "none".

## Step 4: Generate project slug

Derive a slug from the project name: lowercase, replace spaces and special characters with hyphens, collapse multiple hyphens, strip leading/trailing hyphens, max 30 characters.

Example: "Missing Middle Housing Analysis" → "missing-middle-housing-an" (truncated at 30), or a sensible shortened form such as "missing-middle".

## Step 5: Generate stub IDs

Number milestones sequentially: [slug]-m1, [slug]-m2, etc.

## Step 6: Create the project file

Write `~/Documents/Claude Code/goals/projects/[SLUG].md` with these sections:

Frontmatter (YAML):
```
---
name: [PROJECT NAME]
context: [work|home|both]
status: active
slug: [SLUG]
quarterly-goal: [QUARTERLY GOAL or "none"]
updated: [TODAY'S DATE]
---
```

Then these sections in order:

- `## Goals` — a numbered list of the 2–4 key goals the user described
- `## Milestones` — a table with these columns:

```
| Date | Milestone | Status | stub-id |
|------|-----------|--------|---------|
| [date] | [description] | pending | [slug]-m1 |
```

One row per milestone, Status = "pending" for all new milestones.

- `## Notes` — empty section with no content (just the heading)

## Step 7: Add to INDEX.md

Read `~/Documents/Claude Code/goals/projects/INDEX.md`. If the file does not exist, create it with this header first:

```
| slug | name | context | status | quarterly-goal |
|------|------|---------|--------|----------------|
```

Then append a new row:

```
| [SLUG] | [PROJECT NAME] | [context] | active | [QUARTERLY GOAL] |
```

## Step 8: Plant milestone stubs in future week files

For EACH milestone, perform the following steps:

a. Compute the ISO week for the milestone's target date:

```bash
python3 -c "
from datetime import date
d = date.fromisoformat('[MILESTONE DATE]')
iso = d.isocalendar()
print(f'{iso[0]}-W{iso[1]:02d}')
" 2>/dev/null || python -c "
from datetime import date
d = date.fromisoformat('[MILESTONE DATE]')
iso = d.isocalendar()
print(str(iso[0]) + '-W' + str(iso[1]).zfill(2))
"
```

If Python fails, compute the ISO week manually from the `currentDate` context and the milestone date.

b. The target weekly file path is `~/Documents/Claude Code/goals/weekly/[YYYY-Www].md`.

c. Idempotency check: If the file exists, read its content and scan for the literal text `stub-id:[SLUG]-mN` (where N is the milestone number). If that exact string is found anywhere on a line, replace that entire line in place with the updated stub line. If it is not found, append the stub line to the `## Priorities` section.

d. If the file does not exist, create it with this structure:

```
---
context: both
week: [YYYY-Www]
updated: [TODAY'S DATE]
---

## Priorities

## Daily Log
```

e. Add the following line under `## Priorities`:

```
- [ ] [PROJECT NAME] — [MILESTONE DESCRIPTION] [stub-id:[SLUG]-mN]
```

## Step 9: Confirm

Tell the user the following (in plain text, not a code block):

- The project file was created at `~/Documents/Claude Code/goals/projects/[SLUG].md`
- The project was added to INDEX.md
- Milestone stubs were planted in these weekly files: list each `[YYYY-Www].md` on its own line
- Suggest next steps: "/goals-project-review [slug] to update status, /goals-upcoming to see all due dates"
