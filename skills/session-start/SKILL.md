---
name: session-start
description: Use when beginning your daily work session or starting a focused work block.
---

# Session Start

## Overview

Creates a structured daily session file in the brain repository. Always use English output per `always-english-output` rule (even for Slovak input). Pairs with enhanced `session-end` (which includes mandatory repo audit for commits/PRs/branches). Use `token-efficient-communication` for all output to human partner.

## When to Use

Use this skill at the beginning of your workday or when starting a focused work session to:
- Document priorities and context
- Create a reference point for the team and AI agents
- Establish the day's focus and constraints

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Process

Ask these three questions **one at a time**:

1. What is your main focus today?
2. Is there anything being carried over from yesterday?
3. Are there any important meetings, deadlines, or constraints today?

## Template

Creates `sessions/YYYY/MM/YYYY-MM-DD-{github-username}.md`:

```markdown
# {github-username} — {YYYY-MM-DD}

## Morning
**Focus:** {answer to question 1}
**Carried over:** {answer to question 2}
**Meetings/constraints:** {answer to question 3}

---

## During the day

_Use `log-decision` or `log-blocker` to add entries here._

---

## End of day

_Complete with `session-end`._
```

## After Creation

- Create necessary directory structure if needed
- Confirm the session has been started
- Do NOT commit — the session file will be committed with `session-end`

## Red Flags

- Starting work without creating a session context
- Writing vague or generic focus statements
- Skipping this step because "it's just another day"
- Not updating the session file during the day with decisions and blockers

## Notes

- If a session file already exists for today, offer to update it
- The "During the day" section should be populated using `log-decision` and `log-blocker`
- The "End of day" section is completed by `session-end`
- This skill always writes to the brain repository
