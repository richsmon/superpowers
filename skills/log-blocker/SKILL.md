---
name: log-blocker
description: Use when something is blocking progress on a task or feature and the team needs to know.
---

# Log Blocker

## Overview

Records blockers in the brain repository so the team knows what's stuck and who can unblock it.

## When to Use

Use this skill when:
- Something is blocking progress on a task or feature
- You need to make the team aware of a dependency or issue
- Want to formally track what needs to be resolved

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Process

Collect the following information:

1. What exactly is blocked?
2. What is the root cause or dependency?
3. Who can unblock this? (person or team)
4. How urgent is it? (`critical`, `high`, `medium`)

## Template

Creates `blockers/YYYY-MM-DD-{short-title}.md`:

```markdown
# Blocker: {Title}

**Date:** {YYYY-MM-DD}
**Reported by:** {github-username}
**Urgency:** {critical | high | medium}
**Status:** open
**Owner:** {who can unblock}

## What's Blocked

{What can't be done right now}

## Root Cause

{Why it's blocked}

## Resolution

_Pending._
```

## After Creation

- If a session file exists for today, add a reference in the "During the day" section:

```markdown
**[HH:MM] Blocker:** {short summary}
→ blockers/YYYY-MM-DD-short-title.md
```

- Commit with message: `blocker: {short-title}`

## Resolving Blockers

When resolved:
- Change `Status: open` to `Status: resolved`
- Fill in the Resolution section
- Commit with: `blocker(resolved): {short-title}`

## Red Flags

- Continuing to work around blockers instead of logging them
- Not assigning an owner
- Vague descriptions of what's blocked
- Not updating blocker status when resolved
- "I'll figure it out myself, no need to log it"

## Notes

- This skill works from any repository but always writes to the brain repo
- Critical blockers should be escalated appropriately
- Use `session-start` and `session-end` to track daily context around blockers
