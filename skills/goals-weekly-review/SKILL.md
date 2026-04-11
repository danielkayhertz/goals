---
name: goals-weekly-review
description: End-of-week reflection using the 4Ls framework (Liked, Learned, Lacked, Longed For). Reviews the week's progress and writes a reflection to the week file. Supports quick mode. Triggers on: "weekly review", "end of week", "week reflection", "how was my week", /goals-weekly-review
triggers:
  - weekly review
  - end of week
  - week reflection
  - how was my week
  - weekly reflection
  - /goals-weekly-review
---

# Goals Weekly Review Skill

End-of-week reflection using the 4Ls framework (Liked, Learned, Lacked, Longed For). Reviews the week's task progress and writes a structured reflection to the week file.

---

## Step 1: Get date context

Run the following Python snippet to get today's date, the current ISO week, and the previous ISO week:

```python
from datetime import date, timedelta
t = date.today()
iso = t.isocalendar()
prev = (t - timedelta(weeks=1)).isocalendar()
print(f'today={t.isoformat()}')
print(f'week={iso[0]}-W{iso[1]:02d}')
print(f'prev_week={prev[0]}-W{prev[1]:02d}')
```

Use `python3 -c "..."` with a fallback to `python -c "..."`:

```bash
python3 -c "
from datetime import date, timedelta
t = date.today()
iso = t.isocalendar()
prev = (t - timedelta(weeks=1)).isocalendar()
print(f'today={t.isoformat()}')
print(f'week={iso[0]}-W{iso[1]:02d}')
print(f'prev_week={prev[0]}-W{prev[1]:02d}')
" 2>/dev/null || python -c "
from datetime import date, timedelta
t = date.today()
iso = t.isocalendar()
prev = (t - timedelta(weeks=1)).isocalendar()
print('today=' + t.isoformat())
print('week=' + str(iso[0]) + '-W' + str(iso[1]).zfill(2))
print('prev_week=' + str(prev[0]) + '-W' + str(prev[1]).zfill(2))
"
```

If Python is unavailable, fall back to the `currentDate` value from the system context.

---

## Step 2: Detect quick mode

Check the user's original message for the words **"quick"**, **"briefly"**, or **"short"** (case-insensitive).

- If any of these words are present → **quick mode** is active.
- Otherwise → **full mode** (default).

Remember this for Step 5.

---

## Step 3: Detect context filter

Check the user's **invocation message** for whole-word occurrences of "both" and "home" (detection uses the invocation message only; it does not re-fire when the user responds to Step 4's week-selection question):

- If "both" is present (alone or alongside "home") → filter = **both**
- Else if "home" is present as a whole word → filter = **home**
- Otherwise → filter = **work** (default)

Tell the user: "Reviewing your **[work/home/both]** week. Say 'home' or 'both' when invoking to change."

Store CONTEXT_FILTER for use in Step 5.

---

## Step 4: Ask which week to review

Ask the user:

"Are you reviewing this week ([WEEK]) or last week ([PREV_WEEK])?"

Where `[WEEK]` and `[PREV_WEEK]` are the values from Step 1 (e.g., `2026-W15` and `2026-W14`).

Wait for the user's response. Use the selected week for all subsequent file reads and writes.

---

## Step 5: Read the week file and show summary

Read the file at:

```
~/Documents/Claude Code/goals/weekly/[SELECTED_WEEK].md
```

- If the file does not exist: tell the user "No file found for [SELECTED_WEEK]. Nothing to review." and stop.
- Count lines matching `- [x]` (completed tasks) and `- [ ]` (open tasks) anywhere in the file, filtered to CONTEXT_FILTER. Include untagged items in both counts (backward-compat).
  - Filter rule: `context == CONTEXT_FILTER OR context == "both" OR untagged`
- Show the user: "Week [SELECTED_WEEK] — [work/home/both]: [N] done, [M] open."

---

## Step 6: Reflection interview

### Full mode (default)

Ask each question one at a time and wait for the user's response before proceeding to the next.

1. "What went well this week? *(Liked — wins, good momentum, what worked)*"
2. "What did you learn or discover? *(Learned — insights, surprises, new information)*"
3. "What got in your way — blockers, missing resources? *(Lacked)*"
4. "What did you wish you had more of — time, focus, support? *(Longed For)*"
5. "What's the one thing that most needs your attention next week?"
6. "Anything to carry forward or flag for next week's planning?"

### Quick mode

Ask only one question and wait for the response:

"How was the week — any highlights or things to carry forward?"

Skip the remaining questions.

---

## Step 7: Write weekly reflection section

Check whether the selected week file already contains a `## Weekly Reflection` section (search for `## Weekly Reflection` anywhere near the end of the file).

- **If it already exists:** display the existing section to the user and ask "A weekly reflection already exists — overwrite it?" Wait for confirmation before replacing it.
- **If it does not exist:** append the section to the end of the file.

### Full mode — section to write

```markdown
## Weekly Reflection

**Liked:** [answer to Q1]
**Learned:** [answer to Q2]
**Lacked:** [answer to Q3]
**Longed For:** [answer to Q4]
**Focus for next week:** [answer to Q5]
**Carry-forwards:** [answer to Q6]
```

### Quick mode — section to write

```markdown
## Weekly Reflection

**Notes:** [answer]
```

Replace all bracketed placeholders with the user's actual responses before writing.

---

## Step 8: Confirm

Tell the user: "Weekly reflection saved to [SELECTED_WEEK].md."

---

## Quality standards

- Use `python3`/`python` fallback on all date commands (`2>/dev/null || python -c "..."` pattern).
- All file paths use forward slashes (`~/...`).
- If the week file is missing, stop gracefully — do not create an empty file.
- Ask one question at a time in full mode; wait for each response before continuing.
- In quick mode, ask only one question and skip the rest of the interview.
- Confirm with the user before overwriting an existing `## Weekly Reflection` section.
