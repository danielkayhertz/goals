---
name: goals-daily
description: >
  Daily check-in: plan today's priorities and log them in the current week file.
  Supports full interview mode or quick single-line logging. Triggers on:
  "daily check-in", "plan my day", "what should I focus on today", /goals-daily
triggers:
  - daily check-in
  - plan my day
  - what should I focus on today
  - daily plan
  - /goals-daily
---

# Daily Goals Check-In

## Step 1: Get date context

Run the following Python command to get today's date and week information:

```python
python3 -c "
import datetime
today = datetime.date.today()
weekday = today.strftime('%A')
iso_week = today.strftime('%G-W%V')
yesterday = (today - datetime.timedelta(days=1)).strftime('%Y-%m-%d')
print(f'today={today}')
print(f'weekday={weekday}')
print(f'iso_week={iso_week}')
print(f'yesterday={yesterday}')
" 2>/dev/null || python -c "
import datetime
today = datetime.date.today()
weekday = today.strftime('%A')
iso_week = today.strftime('%G-W%V')
yesterday = (today - datetime.timedelta(days=1)).strftime('%Y-%m-%d')
print(f'today={today}')
print(f'weekday={weekday}')
print(f'iso_week={iso_week}')
print(f'yesterday={yesterday}')
"
```

If the command fails, use the `currentDate` context variable (YYYY-MM-DD format) to derive values manually.

From this output, extract:
- `today` (YYYY-MM-DD)
- `weekday` (e.g., "Monday")
- `iso_week` (e.g., "2026-W15")
- `yesterday` (YYYY-MM-DD)

## Step 2: Read or create the current week file

The weekly file is located at: `~/Documents/Claude Code/goals/weekly/[ISO_WEEK].md`

**If the file does NOT exist**, create it with this template.

Format like this:

---
context: both
week: [ISO_WEEK]
updated: [TODAY]
---

## Priorities

## Daily Log

Replace `[ISO_WEEK]` with the week identifier (e.g., "2026-W15") and `[TODAY]` with today's date in YYYY-MM-DD format.

**If the file DOES exist**, read its full contents.

## Step 3: Idempotency check

Look in the `## Daily Log` section for a heading matching `### [WEEKDAY] [TODAY]`.

**If found:**
1. Show the user the existing entry.
2. Ask: "I found today's log entry. Would you like to add to it, replace it, or leave it as-is?"
3. Wait for the user's choice:
   - **Add**: Append new items to the existing `### [WEEKDAY] [TODAY]` section
   - **Replace**: Rewrite the entire section with new content
   - **Leave**: Exit without making any changes

**If not found**: Proceed to Step 4.

## Step 4: Detect mode

Check if the user's message contains any of these keywords: "quick", "just log", "briefly", "one-line".

**Quick mode detected:**
Ask the user: "What's your one-line log for today?"
- Accept a single line of input
- Skip the interview questions
- Proceed directly to Step 5

**Full mode (default):**
Run an interview with ONE question at a time:
1. "What are your top 2–3 priorities for today?"
2. "Anything carrying over from yesterday that you didn't finish?"

Wait for the user's response to each question before moving to the next.

## Step 5: Write today's entry

Add a new section to `## Daily Log` in the weekly file with this format.

Format like this:

### [WEEKDAY] [TODAY]
- [ ] [Priority 1]
- [ ] [Priority 2]
- [ ] [Priority 3]
*(carried over: [item])* — only if user mentioned a carryover

Rules:
- Replace `[WEEKDAY]` with the day name (e.g., "Thursday")
- Replace `[TODAY]` with the date in YYYY-MM-DD format
- Use checkboxes (`- [ ]`) for each priority
- If the user mentioned carryover items in the interview, add a line: `*(carried over: [item])*`
- If no carryover, omit that line

Update the `updated:` field in the frontmatter to today's date (YYYY-MM-DD).

## Step 6: Confirm

After writing the entry:
1. Show the user exactly what was written
2. Confirm the file path where it was saved
3. Remind the user: "/goals-upcoming to see due dates, /goals-weekly to review weekly priorities"

## Error handling

- If the goals directory does not exist, create it: `~/Documents/Claude Code/goals/weekly/`
- If a date command fails, check the `currentDate` context variable. If that's unavailable, ask the user for today's date in YYYY-MM-DD format
- If the user's input is unclear during the interview, ask for clarification rather than guessing
