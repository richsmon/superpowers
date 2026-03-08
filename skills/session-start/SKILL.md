---
name: session-start
description: "Use at the beginning of a work session to initialize the daily session log, establish context, and plan the day. Creates a structured audit trail of all work performed."
---

# Session Start

## Overview

You kick off a work session by establishing context, reviewing where things stand, and creating a session log that will track everything that happens today. This log is the team's audit trail — not just what was committed, but **why**, what was discussed, what decisions were made, and what problems were hit.

**Announce at start:** "Starting session. Let me set up the daily log."

## When to Use

- Beginning of a work day or session
- Resuming work after a break
- Starting a focused work block on a specific feature
- Invoked via `/zaciname` command

## The Process

### Step 1: Identify the Developer

Determine the developer's nick/username:

```bash
git config user.name
```

Use this as `<nick>` throughout. If the user provides a different nick, use that instead.

### Step 2: Determine Date and Paths

```
Date: YYYY-MM-DD (today)
Month folder: docs/sessions/YYYY-MM/
Session file: docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md
```

Create the month folder if it doesn't exist.

### Step 3: Check for Existing Session

If a session log for today already exists for this nick:
- Read it, acknowledge "Resuming existing session"
- Add a `### Resumed at HH:MM` entry to the Work Log
- Skip to Step 6

### Step 4: Gather Context

Run these in parallel to understand current state:

1. **Current branch and status:**
   ```bash
   git branch --show-current
   git status --short
   ```

2. **Recent work (last session's commits):**
   ```bash
   git log --oneline --since="2 days ago" --author="$(git config user.name)" -20
   ```

3. **Previous session log** (find most recent for this nick):
   - Read the last session's handoff note if it exists
   - Check what was planned vs what was done

4. **Project status:**
   - Read `docs/STATUS.md` if it exists
   - Check for open TODOs, blockers from last session

### Step 5: Create Session Log

Create `docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md` with this structure:

```markdown
# Session: YYYY-MM-DD — <nick>

**Branch:** <current branch>
**Started:** HH:MM
**Previous session:** [YYYY-MM-DD](<relative link>) | No previous session

## Context
<2-3 sentences: where we left off, what's the current state>
<Handoff note from previous session if available>

## Plan for Today
- [ ] <carried over from previous session or discussed with user>

## Work Log
<!-- Updated throughout the session by agent and user -->
<!-- Format: ### HH:MM — <what happened> -->

## Decisions Made
<!-- Architecture, technology, design decisions with brief rationale -->

## Issues & Blockers
<!-- Problems encountered, workarounds, unresolved items -->

## Notes
<!-- Freeform notes from user or agent -->
<!-- WARNING: Do NOT write PII, credentials, API keys, or customer data here -->
<!-- This file is committed to git and visible to the entire team -->
```

### Step 6: Present Summary to User

After creating the log, present a brief summary:

```
Session started for <nick> on <branch>.

Since last session:
- <N> commits on this branch
- <summary of recent changes>

Last session ended with:
- <handoff note or "no previous session found">

Uncommitted changes:
- <list or "clean working tree">

What's the plan for today?
```

Wait for the user to confirm or adjust the plan. Update the "Plan for Today" section.

### Step 7: Commit the Session Log

```bash
git add docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md
git commit -m "chore(session): start session YYYY-MM-DD [<nick>]"
```

## Session Log Updates During Work

Throughout the session, update the log at these moments:

| Event | What to Log | Section |
|-------|------------|---------|
| Completed a task/feature | What was done, files touched, commit hash | Work Log |
| Made a design decision | Decision + rationale (1-2 sentences) | Decisions Made |
| Hit a problem/blocker | What happened, workaround if any | Issues & Blockers |
| User adds a note | User's note verbatim | Notes |
| Branched or merged | Branch operation details | Work Log |
| Started brainstorming | "Started brainstorming for <topic>" | Work Log |
| Plan written | "Implementation plan created: <path>" | Work Log |

**Format for Work Log entries:**

```markdown
### HH:MM — <brief title>
<1-3 sentences of detail>
Files: `path/to/file.ts`, `path/to/other.ts`
Commit: `abc1234`
```

## PII Warning

<HARD-GATE>
NEVER write the following into session logs:
- Passwords, API keys, tokens, secrets
- Customer names, emails, phone numbers, or other PII
- Database connection strings
- Internal IP addresses or server names
- Financial data (credit cards, bank accounts)

If the user asks you to log something containing PII, refuse and explain why.
This file will be committed to git and visible to the entire team.
</HARD-GATE>

## Month-End Reminder

If today is the last business day of the month, add a note:

```markdown
> **Month-end note:** This month has N session logs from M team members.
> Consider reviewing `docs/sessions/YYYY-MM/` for patterns, recurring issues, or process improvements.
```

## Red Flags

- **No session log for days** — Team member working without audit trail
- **Empty Work Log at end of session** — Session log wasn't maintained
- **PII in session logs** — Must be scrubbed immediately (git history too)
- **No Plan for Today** — Working without direction
- **Previous session has unresolved blockers** — Address before starting new work

## Integration with Other Skills

- **superpowers:session-end** — Closes the session, creates summary, commits, pushes
- **superpowers:brainstorming** — Logs brainstorming sessions into the Work Log
- **superpowers:writing-plans** — Logs plan creation into the Work Log
- **superpowers:subagent-driven-development** — Logs task completion into the Work Log
- **superpowers:compliance-officer** — Session logs serve as audit trail for ISO 27001
