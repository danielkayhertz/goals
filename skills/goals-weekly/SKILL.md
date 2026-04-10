---
name: goals-weekly
description: >
  Weekly planning and review: look back at last week's unchecked items, then set
  this week's priorities through a guided interview. What are my priorities this week?
  Triggers on: "weekly review", "plan my week", "weekly check-in", "weekly priorities",
  "what are my priorities this week", /goals-weekly
triggers:
  - weekly review
  - plan my week
  - weekly check-in
  - weekly priorities
  - what are my priorities this week
  - /goals-weekly
---

# Goals: Weekly Review & Planning

## Overview
This skill guides you through a weekly planning loop. It checks last week's unchecked items, then runs a short interview to capture this week's priorities. Priorities are written to a dated weekly file.

## Execution Flow

### Step 1: Get Date Context

Run the following Python command (with fallback) to get today's date and ISO week:

```bash
python3 -c "from datetime import datetime, timedelta; d = datetime.now(); week_num = d.isocalendar()[1]; print(f'today={d.strftime(\"%Y-%m-%d\")}'); print(f'iso_week={d.strftime(\"%Y\")}-W{week_num:02d}'); quarter = (d.month - 1) // 3 + 1; print(f'quarter={d.strftime(\"%Y\")}-Q{quarter}'); prev_week = d - timedelta(weeks=1); prev_week_num = prev_week.isocalendar()[1]; print(f'last_week={prev_week.strftime(\"%Y\")}-W{prev_week_num:02d}')" 2>/dev/null || python -c "from datetime import datetime, timedelta; d = datetime.now(); week_num = d.isocalendar()[1]; print(f'today={d.strftime(\"%Y-%m-%d\")}'); print(f'iso_week={d.strftime(\"%Y\")}-W{week_num:02d}'); quarter = (d.month - 1) // 3 + 1; print(f'quarter={d.strftime(\"%Y\")}-Q{quarter}'); prev_week = d - timedelta(weeks=1); prev_week_num = prev_week.isocalendar()[1]; print(f'last_week={prev_week.strftime(\"%Y\")}-W{prev_week_num:02d}')"
```

If both fail, use the current date from the system context (2026-04-10) and calculate:
- `today=2026-04-10`
- `iso_week=2026-W15` (or calculate from context)
- `quarter=2026-Q2`
- `last_week=2026-W14`

**Note:** On date command failure, fall back to system context or ask the user for today's date as a last resort.

Store these four values for use in later steps.

### Step 2: Check for Context Filter

Ask the user (or check context from earlier conversation): "Are you setting priorities for **work**, **home**, or **both** this week?"

Record the filter. Rule for showing items:
- If filter is `work`: show items where `context == "work" OR context == "both"`
- If filter is `home`: show items where `context == "home" OR context == "both"`
- If filter is `both` (or no filter given): show all items

State this filter explicitly to the user: "I'll focus on [work/home/both] items."

### Step 3: Review Last Week's Unchecked Items (if available)

Read the file at: `C:/Users/bpi/Documents/Claude Code/goals/weekly/[LAST_WEEK].md`

For example, if `last_week=2026-W14`, read `C:/Users/bpi/Documents/Claude Code/goals/weekly/2026-W14.md`

Search for all unchecked items (lines starting with `- [ ]`) in the `## Daily Log` and `## Priorities` sections.

If file exists and unchecked items are found:
- Tell the user: "From last week, these items were unchecked: [list them]. We'll ask about rolling them over in a moment."

If file does not exist or has no unchecked items:
- Skip this step silently (no mention to the user).

### Step 4: Read or Create This Week's File

Read the file at: `C:/Users/bpi/Documents/Claude Code/goals/weekly/[ISO_WEEK].md`

For example, if `iso_week=2026-W15`, read `C:/Users/bpi/Documents/Claude Code/goals/weekly/2026-W15.md`

**If the file does NOT exist:**
- Create it with the following template structure:

```
---
context: both
week: [ISO_WEEK]
updated: [TODAY in YYYY-MM-DD]
---

## Priorities

## Daily Log
```

Replace `[ISO_WEEK]` with the value from Step 1 (e.g., `2026-W15`).
Replace `[TODAY]` with the value from Step 1 (e.g., `2026-04-10`).

**If the file DOES exist:**
- Read its current `## Priorities` section and show it to the user (so they see what's already there).
- Tell the user: "Here's what's currently in this week's priorities. We can add, refine, or replace them."

### Step 5: Interview — ONE QUESTION AT A TIME

Wait for the user's answer after each question before moving to the next.

**Question 1:** "What did you accomplish last week? (Share 2–3 things you're proud of.)"

- Wait for the user to answer.
- Acknowledge briefly and move to the next question.

**Question 2 (ONLY if Step 3 found unchecked items):**
"These were unchecked last week: [list them]. Which ones do you want to roll over to this week?"

- Wait for the user's answer.
- Record which items they want to keep.
- Move to the next question.

**Question 3:** "What are your top 3–5 priorities for this week?"

- Apply the context filter from Step 2: only suggest items relevant to the chosen context (work/home/both).
- Wait for the user to provide their priorities.
- Move to the next question.

**Question 4:** "Any deadlines or milestones due this week I should note?"

- Wait for the user's answer.
- Note any dates mentioned.

### Step 6: Write the Priorities Section

Replace everything from the `## Priorities` heading up to (but not including) the `## Daily Log` heading. Do not modify the `## Daily Log` section or any of its contents. Only update the `updated:` value in the frontmatter.

Replace the `## Priorities` section in the week's file with a fresh bulleted list. Format like this:

- [ ] [Priority text] [[context tag if known]]
- [ ] [Deadline text] — due [date] [[context tag]]
- [ ] [Rolled-over item from last week] [[context tag]]

Include:
- The 3–5 priorities the user named in Question 3.
- Any deadlines from Question 4 (formatted as "— due [date]").
- Any rolled-over unchecked items from Question 2, if applicable.

Add a context tag in double brackets (e.g., `[work]` or `[home]`) if the user specified a context for that item. If the context filter was set in Step 2 (e.g., work-only session), default untagged items to that context tag rather than `[both]`. Only use `[both]` if context is genuinely ambiguous or the session filter was already `both`.

Update the frontmatter field `updated:` to today's date (from Step 1).

Save the file.

### Step 7: Confirm to User

Tell the user: "Priorities saved to `weekly/[ISO_WEEK].md`. Suggest running `/goals-upcoming` to see all upcoming due dates across weeks."

End the skill execution.

## File Paths (All absolute, forward slashes)

- Weekly files directory: `C:/Users/bpi/Documents/Claude Code/goals/weekly/`
- Last week's file: `C:/Users/bpi/Documents/Claude Code/goals/weekly/[LAST_WEEK].md`
- This week's file: `C:/Users/bpi/Documents/Claude Code/goals/weekly/[ISO_WEEK].md`

## Quality Standards Applied

1. **Python fallback:** Python command includes both `python3` and `python` with proper error handling (`2>/dev/null ||`).
2. **Date context:** If date command fails, fall back to system context or ask the user.
3. **No triple-backtick templates:** Priority list format shown as "Format like this:" with plain prose, no fenced code blocks.
4. **Path format:** All paths use `C:/Users/bpi/...` with forward slashes only.
5. **Context filter rule:** Explicitly stated—"show if `context == filter OR context == "both"`"—and applied in Question 3.
6. **YAML frontmatter:** `name: goals-weekly` included in header.
7. **Interview sequence:** One question at a time, wait for each answer, unambiguous decision points.
8. **Last-week review step:** Clear conditional logic—only shown if file exists AND unchecked items found; skipped silently otherwise.

## Example Execution (Illustrative)

**Input:** User says `/goals-weekly` on 2026-04-10.

**Step 1 output:**
```
today=2026-04-10
iso_week=2026-W15
quarter=2026-Q2
last_week=2026-W14
```

**Step 2:** User says "work". Filter set to work.

**Step 3:** `2026-W14.md` exists; unchecked item found: "- [ ] Finish report". User sees: "From last week, this item was unchecked: Finish report. We'll ask about rolling it over."

**Step 4:** `2026-W15.md` does not exist; created with template.

**Step 5 Interview:**
- Q1: User says "Shipped new feature, fixed 3 bugs, reviewed design docs."
- Q2: User says "Yes, roll over the report."
- Q3: User says "Finish report, start Q2 planning, review architecture docs."
- Q4: User says "Report due Friday, all-hands Thursday."

**Step 6:** Priorities section written:
```
## Priorities

- [ ] Finish report — due 2026-04-11 [work]
- [ ] Start Q2 planning [work]
- [ ] Review architecture docs [work]
- [ ] All-hands meeting — due 2026-04-17 [work]
```

**Step 7:** User sees: "Priorities saved to `weekly/2026-W15.md`. Suggest running `/goals-upcoming` to see all upcoming due dates across weeks."

---

**End of Skill Definition**
