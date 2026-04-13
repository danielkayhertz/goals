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

## Step 2: Detect context filter

Check the user's **invocation message** for whole-word occurrences of "both" and "home" (not subsequent messages in the session):

- If "both" is present (alone or alongside "home") → filter = **both**
- Else if "home" is present as a whole word → filter = **home**
- Otherwise → filter = **work** (default)

Tell the user: "Planning your **[work/home/both]** day. Say 'home' or 'both' when invoking to change."

Store CONTEXT_FILTER for use in Steps 5 and 6.

## Step 3: Read or create the current week file

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

## Step 4: Idempotency check

Look in the `## Daily Log` section for a heading matching `### [WEEKDAY] [TODAY]`.

**If found:**
1. Filter the existing entry's task lines to only those tagged `[[CONTEXT_FILTER]]` or `[both]`. Omit lines tagged with other contexts.
2. If no task lines remain after filtering, treat this as if no entry was found and proceed to Step 5.
3. Otherwise, show the user the filtered entry and ask: "I found today's log entry (showing [CONTEXT_FILTER] items). Would you like to add to it, replace it, or leave it as-is?"
4. Wait for the user's choice:
   - **Add**: Append new items to the existing `### [WEEKDAY] [TODAY]` section
   - **Replace**: Rewrite the entire section with new content
   - **Leave**: Exit without making any changes

**If not found**: Proceed to Step 5.

## Step 5: Detect mode

Check if the user's message contains any of these keywords: "quick", "just log", "briefly", "one-line".

**Quick mode detected:**
Ask the user: "What's your one-line log for today?"
- Accept a single line of input
- Skip the interview questions and priority tagging — all tasks default to `[P2]`
- Task line format: `- [ ] [task] [[CONTEXT_FILTER]] [P2]`
- Proceed directly to Step 6

**Full mode (default):**
Before the interview, check for yesterday's reflection: read yesterday's `### [Weekday DATE]` section from the current week file (or last week's file if today is Monday). Look for a `#### Reflection` block. If found and the 'Carry forward:' field is not '—', apply CONTEXT_FILTER: check whether yesterday's incomplete items (`- [ ]` lines) include any tagged `[[CONTEXT_FILTER]]` or `[both]`. Only surface the carry-forward text if at least one such item exists. When surfacing: 'From yesterday's reflection — carry forward: [text]. Keep this in mind as you plan today.'

Run an interview with ONE question at a time:
1. "What are your top 2–3 priorities for today?"

After the user provides their priorities, ask for each task one at a time: 'Is **[task name]** a must-do today? (P1) or normal priority? [default: P2]' Wait for response. Tag P1 → `[P1]`, P2 or no answer → `[P2]`. After all tasks tagged, count P1s. If > 3: 'You have [N] P1 tasks — recommended max is 3 per day. Your P1s: [numbered list]. Downgrade any to P2? (reply with numbers, or keep all)' Soft warning — user decides.

**Project tag assignment:** After P1/P2 tagging, read the project index at `~/Documents/Claude Code/goals/projects/INDEX.md`. For each priority that does not already have a `[proj:slug]` tag, propose one based on the slug list and task text. Present all proposals at once: "Suggested project tags — correct any or say 'none':" then list each task with its proposed `[proj:slug]`. Wait for confirmation before writing. Tag order per item: `[[context]] [proj:slug] [P1/P2]`.

2. "Anything carrying over from yesterday that you didn't finish?"

Wait for the user's response to each question before moving to the next.

## Step 6: Write today's entry

Add a new section to `## Daily Log` in the weekly file. Use CONTEXT_FILTER as the default context tag on every task line.

Format like this:

### [WEEKDAY] [TODAY]
- [ ] [Priority 1] [[CONTEXT_FILTER]] [proj:slug] [P1]
- [ ] [Priority 2] [[CONTEXT_FILTER]] [proj:slug] [P2]
*(carried over: [item])* — only if user mentioned a carryover

Rules:
- Replace `[WEEKDAY]` with the day name (e.g., "Thursday")
- Replace `[TODAY]` with the date in YYYY-MM-DD format
- Replace `[CONTEXT_FILTER]` with the active filter (e.g., `work`, `home`, or `both`)
- Use checkboxes (`- [ ]`) for each priority
- Tag order: `[[context]] [proj:slug] [P1/P2]`
- Priority tag (`[P1]` or `[P2]`) is always included
- If the user mentioned carryover items in the interview, add a line: `*(carried over: [item])*`
- If no carryover, omit that line

Update the `updated:` field in the frontmatter to today's date (YYYY-MM-DD).

## Step 7: Confirm

After writing the entry:
1. Show the user exactly what was written
2. Confirm the file path where it was saved
3. Remind the user: "/goals-upcoming to see due dates, /goals-weekly to review weekly priorities"

## Error handling

- If the goals directory does not exist, create it: `~/Documents/Claude Code/goals/weekly/`
- If a date command fails, check the `currentDate` context variable. If that's unavailable, ask the user for today's date in YYYY-MM-DD format
- If the user's input is unclear during the interview, ask for clarification rather than guessing
