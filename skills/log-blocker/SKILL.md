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

## Git Safety Protocol

Before reading or writing in the brain repo:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the blocker lands against a fresh tip and the session file you append to is the current one.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Today's session file should already be untracked from `session-start`; surprising modified files are not normal — confirm with the user.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` should be empty after pull.

## Process

Collect the following information:

1. What exactly is blocked?
2. What is the root cause or dependency?
3. Who can unblock this? (person or team)
4. How urgent is it? (`critical`, `high`, `medium`)

## Read Template

Read the template from the brain repo (single source of truth):

```
Read <brain>/templates/log/blocker.md
```

If the file is missing, fail loudly with this message and stop:

> `templates/log/blocker.md not found in <brain-path>. Either restore from git or update brain.`

Do NOT fall back to an embedded copy — there is no embedded copy.

The template defines the Date / Reported by / Urgency / Status / Owner frontmatter and the What's Blocked / Root Cause / Resolution sections. Fill placeholders from the answers to the Process questions above, then write the filled file to `blockers/YYYY-MM-DD-<slug>.md` in the brain repo.

## After Creation

- If a session file exists for today, add a reference in the "During the day" section:

```markdown
**[HH:MM] Blocker:** {short summary}
→ blockers/YYYY-MM-DD-short-title.md
```

- **Do NOT commit.** See "Commit Policy" below.

## Resolving Blockers

When resolved:
- Change `Status: open` to `Status: resolved`
- Fill in the Resolution section
- Append a `[HH:MM] Blocker resolved: ...` line to today's session diary
- **Do NOT commit.** Resolution updates follow the same policy — `session-end` bundles them with the rest of the day.

## Commit Policy

**This skill does NOT commit.** It writes the blocker file (and edits it on resolution) and appends to today's session diary, but stages nothing and commits nothing.

Why: `log-blocker` — both opens and resolutions — fires repeatedly through the day. If each call committed, the daily diary would land many tiny commits that belong together as "today". `session-end` bundles every same-day write — session file, decisions, blockers, resolutions — into one commit at end of day, alongside the repo audit.

If a blocker is so consequential it needs an out-of-band ping (e.g. CTO, oncall), do that *in addition to* logging it — but the commit still waits for `session-end`.

## Red Flags

- Continuing to work around blockers instead of logging them
- Not assigning an owner
- Vague descriptions of what's blocked
- Not updating blocker status when resolved
- **Committing from this skill (open or resolved).** session-end bundles the daily writes — auto-committing here breaks the daily diary cadence.
- Skipping the Git Safety Protocol — appending to a stale session file or writing from the wrong branch.
- "I'll figure it out myself, no need to log it"

## Notes

- This skill works from any repository but always writes to the brain repo
- Critical blockers should be escalated appropriately
- Use `session-start` and `session-end` to track daily context around blockers
- The `/brain-log-blocker` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.
