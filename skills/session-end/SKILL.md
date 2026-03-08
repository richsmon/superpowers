---
name: session-end
description: "Use at the end of a work session to summarize work done, update project status, write handoff notes, commit everything, push to remote, and optionally notify the team via Slack."
---

# Session End

## Overview

You wrap up the work session cleanly: summarize what was accomplished, document the state for whoever picks this up next (including future you), commit and push everything, and optionally send a team notification. No loose ends, no "I'll remember tomorrow."

**Announce at start:** "Wrapping up session. Let me finalize the daily log."

## When to Use

- End of a work day or session
- Before switching to a different project/branch
- Before going on vacation or handing off work
- Invoked via `/koncime` command

## The Process

### Step 1: Identify Session Log

Find today's session log:

```
docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md
```

If no session log exists for today, create a minimal one (the developer may not have used `/zaciname`). Gather what you can from git history.

### Step 2: Gather Session Metrics

Run in parallel:

```bash
# Commits made during this session
git log --oneline --since="$(date -I)" --author="$(git config user.name)"

# Files changed today
git diff --stat $(git log --since="$(date -I)" --author="$(git config user.name)" --format="%H" | tail -1)^..HEAD 2>/dev/null || echo "No commits today"

# Current uncommitted changes
git status --short

# Current branch
git branch --show-current

# Test status (if test command is known)
# npm test / pytest / etc. — run if configured
```

### Step 3: Review Uncommitted Work

If there are uncommitted changes:

1. Show the user what's uncommitted
2. Ask: "Commit these changes before closing the session?"
3. If yes, stage and commit with a meaningful message
4. If no, note in session log as "Uncommitted work left in working tree"

### Step 4: Check Plan Completion

Read the "Plan for Today" from the session log:

- Mark completed items with `[x]`
- Items not done → carry forward to handoff note
- Calculate completion rate

### Step 5: Write Session Summary

Append to the session log:

```markdown
---

## Session Summary

**Ended:** HH:MM
**Duration:** ~Xh

### Completed
- <item 1> (`commit-hash`)
- <item 2> (`commit-hash`)

### Not Completed
- <item> — Reason: <why>

### Metrics
| Metric | Value |
|--------|-------|
| Commits | N |
| Files changed | N |
| Lines added | +N |
| Lines removed | -N |
| Tests added | N |
| Plan completion | N/M items (X%) |

### Handoff Note
> **If someone continues this work tomorrow (including future you):**
> - Current state: <where things stand>
> - Next step: <what to do first>
> - Watch out for: <gotchas, things in progress, fragile areas>
> - Blocked on: <anything waiting on external input>
```

### Step 6: Update Project Status

If `docs/STATUS.md` exists, update it with current state. If it doesn't exist, create it:

```markdown
# Project Status

**Last updated:** YYYY-MM-DD by <nick>
**Branch:** <current branch>
**Milestone:** <current milestone if known>

## Current State
<2-3 sentences about where the project stands>

## Recent Activity
- YYYY-MM-DD [<nick>]: <1-line summary>

## Active Branches
| Branch | Owner | Status | Last Activity |
|--------|-------|--------|---------------|
| <branch> | <nick> | <status> | YYYY-MM-DD |

## Known Issues / Blockers
- <issues from session>
```

### Step 7: Final Commit and Push

```bash
# Stage session log and status
git add docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md
git add docs/STATUS.md

# Commit
git commit -m "chore(session): end session YYYY-MM-DD [<nick>]

$(cat <<'SUMMARY'
Summary: <1-2 sentence summary>
Commits today: N
Files changed: N
Plan completion: X%
Next: <handoff note summary>
SUMMARY
)"

# Push to remote
git push origin $(git branch --show-current)
```

### Step 8: Slack Notification (Optional)

If the team uses Slack and a webhook or MCP is configured, send a structured report.

**Check for Slack webhook:**

```bash
# Check environment variable
echo "${SLACK_SESSION_WEBHOOK:-not configured}"
```

**If configured, send via curl:**

```bash
curl -s -X POST "$SLACK_SESSION_WEBHOOK" \
  -H 'Content-Type: application/json' \
  -d '{
    "blocks": [
      {
        "type": "header",
        "text": {"type": "plain_text", "text": "Session Report: <nick>"}
      },
      {
        "type": "section",
        "fields": [
          {"type": "mrkdwn", "text": "*Date:*\nYYYY-MM-DD"},
          {"type": "mrkdwn", "text": "*Branch:*\n<branch>"},
          {"type": "mrkdwn", "text": "*Duration:*\n~Xh"},
          {"type": "mrkdwn", "text": "*Commits:*\nN"}
        ]
      },
      {
        "type": "section",
        "text": {"type": "mrkdwn", "text": "*Completed:*\n- item 1\n- item 2"}
      },
      {
        "type": "section",
        "text": {"type": "mrkdwn", "text": "*Next:*\n<handoff summary>"}
      }
    ]
  }'
```

**If Slack MCP is available, use it instead:**

```
CallMcpTool(server="slack", tool="send_message", arguments={
  channel: "#dev-updates",
  blocks: [...]
})
```

**If no Slack is configured:** skip silently, no error. The session log in git is the primary record.

### Step 9: Present Closing Summary

Show the user a clean summary:

```
Session ended for <nick>.

Today:
  Commits: N
  Files changed: N
  Plan: X/Y completed (Z%)

Pushed to: origin/<branch>

Handoff: <1-2 sentence handoff note>

Have a good one!
```

## Handling Edge Cases

| Situation | Action |
|-----------|--------|
| No session log exists | Create minimal one from git history |
| No commits today | Log "exploration/planning session — no code changes" |
| Merge conflicts on push | Alert user, don't force push, leave for next session |
| Tests failing | Warn user, ask if they want to commit anyway, note in session log |
| Multiple sessions same day | Append to existing log, add "Session 2" header |
| User forgot `/zaciname` | Reconstruct from git log, create retroactive session log |

## Month-End Team Summary

If today is the last business day of the month and the user is a tech lead or wants a summary:

```markdown
# Monthly Team Summary: YYYY-MM

## Session Activity
| Team Member | Sessions | Commits | Top Focus Areas |
|-------------|----------|---------|-----------------|
| richsmon | 18 | 142 | Auth module, API redesign |
| peter | 16 | 98 | Frontend, Design system |
| jana | 20 | 167 | Database, Migrations |

## Key Decisions This Month
- <aggregated from session logs>

## Recurring Issues
- <patterns from Issues & Blockers sections>
```

This can be generated by reading all session logs in `docs/sessions/YYYY-MM/`.

## Red Flags

- **Pushing without session log** — Work is untracked
- **Empty handoff note** — Tomorrow-you will be lost
- **Uncommitted changes left behind** — Potential data loss
- **Tests failing on push** — Don't ship broken code
- **Skipping push** — Team can't see your progress
- **PII in session summary or Slack message** — Same rules as session-start

## Integration with Other Skills

- **superpowers:session-start** — Creates the session log this skill finalizes
- **superpowers:verification-before-completion** — Verify tests pass before final commit
- **superpowers:tech-lead** — Monthly summaries feed into team retrospectives
- **superpowers:compliance-officer** — Session logs are part of the ISO 27001 audit trail
- **superpowers:sre-engineer** — Session logs document incident investigations
