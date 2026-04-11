---
name: goals-daily-review
description: "End-of-day review: check off completed tasks from today's log, capture a reflection with takeaways and carry-forwards. Triggers on: \"daily review\", \"end of day\", \"how did today go\", /goals-daily-review"
triggers:
  - daily review
  - end of day
  - how did today go
  - end of my day
  - /goals-daily-review
---

# Goals Daily Review Skill

End-of-day review: check off completed tasks from today's log, then capture a reflection with takeaways and carry-forwards.

---

## Step 1: Get date context

Run the following to get today's date, weekday name, and ISO week:

    python3 -c "from datetime import date; d = date.today(); print(d.strftime('%Y-%m-%d')); print(d.strftime('%A')); print(d.strftime('%G-W%V'))"

If python3 fails, retry with `python` instead. If both fail, fall back to the `currentDate` context variable for the date, and derive the weekday and ISO week from it.

Store:
- TODAY = YYYY-MM-DD (e.g. 2026-04-10)
- WEEKDAY = full name (e.g. Thursday)
- ISO_WEEK = YYYY-Www (e.g. 2026-W15)

---

## Step 2: Detect context filter

Check the user's **invocation message** for whole-word occurrences of "both" and "home" (not subsequent messages in the session):

- If "both" is present (alone or alongside "home") → filter = **both**
- Else if "home" is present as a whole word → filter = **home**
- Otherwise → filter = **work** (default)

Tell the user: "Reviewing your **[work/home/both]** day. Say 'home' or 'both' when invoking to change."

Store CONTEXT_FILTER for use in Step 5.

---

## Step 3: Read week file

Path: `~/Documents/Claude Code/goals/weekly/[ISO_WEEK].md`

Read the file. If it does not exist, tell the user:

    No week file found for [ISO_WEEK]. Run /goals-daily to start today's log first.

Then stop.

---

## Step 4: Find today's section

Look for a heading matching `### [WEEKDAY] [TODAY]` (e.g. `### Thursday 2026-04-10`).

The section begins at that heading and ends at the next line that starts with `##` or `###` — but NOT `####`. Lines starting with `####` are within the section, not boundaries.

If the heading is not found, tell the user:

    No entry found for today ([TODAY]). Run /goals-daily to log your day first.

Then stop.

---

## Step 5: Extract unchecked tasks

Within today's section only, find all lines matching `- [ ]`. Apply CONTEXT_FILTER: keep a task if its context tag (`[work]`, `[home]`, `[both]`) matches the filter, or if it has no context tag (untagged tasks always match).

- Filter rule: `context == CONTEXT_FILTER OR context == "both" OR untagged`

If none match after filtering, tell the user:

    No open [work/home] tasks for today — nothing to review.

Then stop.

---

## Step 6: Batch status check

Display the unchecked tasks numbered, like this:

    1. Submit draft [work] [P1]
    2. Reply to emails [work] [P2]
    3. Review PR [work] [P2]

Then ask once:

    Which are done? Reply with numbers (e.g. "1, 3"), "all", or "none".

Wait for the user's response. Map the selected numbers to `- [x]`; all others remain `- [ ]`.

---

## Step 7: Update file in place

- Read the full file content.
- Within today's section only, replace each original `- [ ]` task line with its updated version (checked or unchanged).
- Do not modify any lines outside today's section.
- Hold the updated content in memory — you will write it in Step 11.

---

## Step 8: Show summary

Tell the user:

    Today ([TODAY]) — [work/home/both]: [N] done, [M] not done.

(Use the active CONTEXT_FILTER label in place of [work/home/both].)

where N = tasks marked done, M = tasks left unchecked.

---

## Step 9: Ask for takeaway

Ask:

    Any quick wins or things worth noting from today?

Wait for the user's response. They may skip (blank response or "skip" or "none"). Store the response, or use `—` if skipped.

---

## Step 10: Ask for carry-forward

Ask:

    Anything to carry into tomorrow's planning?

Wait for the user's response. They may skip. Store the response, or use `—` if skipped.

---

## Step 11: Write reflection block

Note: Steps 9–10 questions are free-form and not context-filtered — they apply to the whole day regardless of active context.

Check whether `#### Reflection` already exists within today's section.

**If it already exists:** Display the existing block and ask:

    A reflection already exists for today — overwrite it?

Wait for confirmation. If the user says no, skip writing the reflection and proceed to Step 11.

**If it does not exist (or user confirmed overwrite):** Build the reflection block. Format like this (with a blank line before `#### Reflection` and a blank line after the last bullet):

    [blank line]
    #### Reflection
    - **Takeaway:** [answer from step 8, or —]
    - **Carry forward:** [answer from step 9, or —]
    [blank line]

**Where to insert it:**

- If today is NOT the last section in the file: insert the block after the last task line in today's section and before the blank line(s) leading into the next `##` or `###` heading.
- If today IS the last section (no subsequent `##` or `###` heading): append the block at the end of the file.

Example when today is not the last section:

    ### Thursday 2026-04-10
    - [x] Submit draft [work] [P1]
    - [ ] Reply to emails [work] [P2]

    #### Reflection
    - **Takeaway:** Focused morning was productive
    - **Carry forward:** Block 2hr tomorrow for data work

    ### Friday 2026-04-11

Example when today is the last section:

    ### Thursday 2026-04-10
    - [x] Submit draft [work] [P1]

    #### Reflection
    - **Takeaway:** —
    - **Carry forward:** —

(end of file — no trailing blank line needed beyond the block's own blank line)

If overwriting an existing reflection: replace the old `#### Reflection` block (from that heading through the blank line after the last bullet) with the new block in the same position.

---

## Step 12: Write updated file and confirm

Write the full updated file content to `~/Documents/Claude Code/goals/weekly/[ISO_WEEK].md`.

Tell the user:

    Review saved to [ISO_WEEK].md.
