---
name: goals-quarterly
description: >
  Quarterly planning and review: review last quarter's progress, then set goals
  for this quarter organized by strategic pillar. Triggers on: "quarterly review",
  "quarterly planning", "set quarterly goals", "review my quarter", /goals-quarterly
triggers:
  - quarterly review
  - quarterly planning
  - set quarterly goals
  - review my quarter
  - /goals-quarterly
---

# /goals-quarterly — Quarterly Planning & Review

## Overview
Reviews last quarter's progress and captures this quarter's goals organized by strategic pillar. Reads from `C:/Users/bpi/Documents/Claude Code/goals/pillars.md` (created by `/goals-onboarding`) and reads/writes quarterly goal files.

## Workflow

### Step 1: Get Date Context
Run Python to determine current date, quarter, and previous quarter:

```python
from datetime import date
t = date.today()
q = (t.month - 1) // 3 + 1
prev_q = q - 1 if q > 1 else 4
prev_year = t.year if q > 1 else t.year - 1
print(f'today={t.isoformat()}')
print(f'quarter={t.year}-Q{q}')
print(f'prev_quarter={prev_year}-Q{prev_q}')
```

Use fallback: `python3 -c "..." 2>/dev/null || python -c "..."`. If both fail, use `currentDate` context variable (today's date is 2026-04-10).

### Step 2: Check Pillars Exist
Read `C:/Users/bpi/Documents/Claude Code/goals/pillars.md`. If missing, tell user:
> "Please run /goals-onboarding first to set up your strategic pillars."

Stop workflow if file is missing.

### Step 3: Ask for Context Filter (Optional)
Optionally ask user: "Are you reviewing work, home, or both goals this quarter?"
- If "work": apply context filter to show only `context: work` or `context: both` pillars
- If "home": apply context filter to show only `context: home` or `context: both` pillars
- If user says "both" (or doesn't respond): show all pillars

The rule: `context == filter OR context == "both"`. No filter = show all.

### Step 4: Extract Pillars
From `## Pillars` section in pillars.md, extract pillar names and their context tags (inline as `[work]`, `[home]`, or `[both]`). Apply context filter from Step 3.

### Step 5: Review Last Quarter (Optional)
If `C:/Users/bpi/Documents/Claude Code/goals/quarterly/[PREV_QUARTER].md` exists, read and summarize to user:
- Count completed goals (marked with `✅` or `- [x]`)
- Count incomplete goals (marked with `- [ ]`)
- Report pattern: "[N] completed, [M] incomplete. Here are the highlights: [brief list]"

If file doesn't exist, skip silently.

### Step 6: Load or Create This Quarter's File
Path: `C:/Users/bpi/Documents/Claude Code/goals/quarterly/[QUARTER].md`

If file exists, show user the current goals and ask: "Update these goals, start fresh, or add to them?"

Branch behavior:
- **Update**: Pre-populate the interview with each pillar's existing goals. For each pillar, show the current goals and ask: "Here are your current goals for [Pillar]. Any changes?" User can revise, keep as-is, or add. Final output replaces the goals section with the revised goals.
- **Start fresh**: Discard existing goals. Run the full interview for each pillar from scratch as if the file were new.
- **Add to them**: Skip pillars that already have goals; only ask the goal question for pillars that have no goals in the existing file. Append new goals to the goals section without disturbing existing entries.

If file doesn't exist, create skeleton:
```
---
context: both
quarter: [QUARTER]
updated: [TODAY]
---

## Goals
```

### Step 7: Interview — One Question Per Pillar
For each filtered pillar (in order):

Ask: "For your **[Pillar Name]** pillar [context] — what are your 1–3 goals this quarter? Be specific about what 'done' looks like."

Wait for the user's response before asking about the next pillar. Do not ask multiple pillar questions in the same message.

Collect user's response for each pillar before moving on.

After all pillars, ask: "Any major deadlines or external commitments this quarter I should note?"

### Step 8: Build Goals Section
Write goals in this format (no fenced code blocks, plain prose):

**## Goals** (section heading, preserving everything before it)

**### [Pillar Name 1] [context]**
- [ ] [Goal 1] — [what done looks like]
- [ ] [Goal 2] — [what done looks like]

**### [Pillar Name 2] [context]**
- [ ] [Goal 1] — [what done looks like]

**### Deadlines & Commitments**
- [YYYY-MM-DD]: [commitment description]

Replace from `## Goals` heading to end of file with new content. Preserve frontmatter.

### Step 9: Update Frontmatter
Set `updated: [TODAY]` in frontmatter to today's date (YYYY-MM-DD format).

### Step 10: Write File
Write updated quarterly file to `C:/Users/bpi/Documents/Claude Code/goals/quarterly/[QUARTER].md`.

### Step 11: Confirm
Tell user: "Goals saved to `quarterly/[QUARTER].md`. Tip: use `/goals-new-project` to set up a workplan for any of these goals."

## File Paths
All paths must use forward slashes:
- Pillars: `C:/Users/bpi/Documents/Claude Code/goals/pillars.md`
- Quarterly goals: `C:/Users/bpi/Documents/Claude Code/goals/quarterly/[QUARTER].md`

## Quality Standards
- **Python fallback**: Always use `python3 -c "..." 2>/dev/null || python -c "..."` for cross-platform compatibility
- **Date handling**: Use Python output; if both Python commands fail, fall back to `currentDate` context variable
- **Format**: No fenced code blocks in frontmatter or goal sections — use plain prose formatting (e.g., "Format like this: - [ ] ...")
- **Empty sections**: If no items under a section, write "[no items under ## [Section heading]]"
- **Context filter rule**: State explicitly: "`context == filter OR context == \"both\"`"
- **Section boundaries**: When replacing `## Goals` section, specify what to preserve (frontmatter) vs. replace (everything from `## Goals` onward)

## Frontmatter Template
```yaml
---
context: both
quarter: YYYY-Qn
updated: YYYY-MM-DD
---
```

## Execution Notes
- This skill is part of the goals system created by `/goals-onboarding`
- Data directory: `C:/Users/bpi/Documents/Claude Code/goals/`
- No git repo required
- Works with existing quarterly files or creates new ones
