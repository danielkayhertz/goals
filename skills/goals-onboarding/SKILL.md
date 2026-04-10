---
name: goals-onboarding
description: >
  Set up the personal goal-tracking system for the first time. Run this once
  to create the goals directory, define your strategic pillars, and initialize
  the project index. Triggers on: "set up goals", "initialize goals system",
  "goals onboarding", /goals-onboarding
triggers:
  - set up goals
  - initialize goals system
  - goals onboarding
  - /goals-onboarding
---

# Goals System Onboarding

You are setting up a personal goal-tracking system. Follow these steps exactly.

## Step 1: Create the directory structure

First, check whether the goals system is already initialized. If `C:/Users/bpi/Documents/Claude Code/goals/pillars.md` already exists, warn the user:

> "Goals system already set up. Re-running will overwrite your pillars and project index."

Ask for explicit confirmation before proceeding. If they do not confirm, stop here.

If confirmed (or if the file does not exist), run:
```bash
mkdir -p "C:/Users/bpi/Documents/Claude Code/goals/quarterly"
mkdir -p "C:/Users/bpi/Documents/Claude Code/goals/weekly"
mkdir -p "C:/Users/bpi/Documents/Claude Code/goals/projects"
```

## Step 2: Interview the user about their strategic pillars

Explain: "Strategic pillars are 2–4 ongoing focus areas that don't change quarter to quarter. Examples: 'Research & Publications', 'Home Projects', 'Health & Fitness'. They anchor all your quarterly goals."

Collect pillars using the following per-pillar loop. Repeat until the user has defined all their pillars:

1. Ask: "What is this pillar's name and a one-sentence description?"
2. Ask: "Is this pillar primarily [work], [home], or [both]?"
3. Ask: "Do you have another pillar to add? (2–4 total recommended)"
   - If the user has defined only 1 pillar and says they're done, gently encourage at least one more: "Strategic pillars work best in pairs — would you like to add at least one more?"
   - If the user tries to define a 5th pillar, suggest consolidating: "You already have 4 pillars, which is the recommended maximum. Consider whether this new area fits under an existing pillar before adding another."
   - Stop when the user has defined 2–4 pillars or explicitly says they're done.

## Step 3: Write pillars.md

After collecting all pillar answers, write `C:/Users/bpi/Documents/Claude Code/goals/pillars.md`:

```markdown
---
updated: [YYYY-MM-DD]
---

## Pillars

[For each pillar, format as:]
### [Pillar Name] [[context tag: work|home|both]]
[One-sentence description]

```

## Step 4: Initialize the project INDEX

Write `C:/Users/bpi/Documents/Claude Code/goals/projects/INDEX.md`:

```markdown
# Project Index

| slug | name | context | status | quarterly-goal |
|------|------|---------|--------|----------------|
```

Column formats: `slug` is kebab-case; `context` is work/home/both; `status` is active/paused/complete; `quarterly-goal` is `YYYY-Qn — "goal text"`.

## Step 5: Confirm completion

Tell the user:
- The goals directory is set up at `C:/Users/bpi/Documents/Claude Code/goals/`
- Their pillars are saved in `pillars.md`
- Next steps: run `/goals-quarterly` to set this quarter's goals, then `/goals-weekly` to plan this week
- Available commands: `/goals-summary`, `/goals-daily`, `/goals-weekly`, `/goals-quarterly`, `/goals-new-project`, `/goals-project-review`, `/goals-schedule`, `/goals-upcoming`
