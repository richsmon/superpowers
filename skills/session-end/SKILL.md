---
name: session-end
description: Use when ending your daily work session or wrapping up a focused work block. Performs mandatory repository audit (today's commits, PRs, branches) and records everything. Enforces English output.
---

# Session End

## Overview

Closes the daily session file with accomplishments, decisions, blockers, plans for tomorrow, **area-aware Repository Audit**, and PARA memory integration. At the start of session-end, the user selects (or confirms) which areas they worked in today from: `app`, `infra`, `platform`, `intelligence`, `brain`, `bot`, `superpowers`. The skill then audits only those repositories + the brain repo for today's commits, PRs, and branches. Records everything and persists durable facts using `para-memory-files`. Commits and pushes to the brain repository. Always outputs in English.

## When to Use

Use this skill at the end of your workday or focused session to:
- Let the user select/confirm which areas they worked in today (`app`, `infra`, `platform`, `intelligence`, `brain`, `bot`, `superpowers`)
- Perform area-aware repository audit only on selected areas + brain repo
- Cross-reference actual git commits with what the user reports
- Persist durable knowledge using `para-memory-files` skill (daily note + knowledge graph + tacit memory)
- Record decisions and blockers (or link to existing ones)
- Set clear direction for the next day
- Close the daily context with complete, accurate historical record
- Enforce English output regardless of input language (Slovak or other)

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`, `../tam-brain`, `../../tam-brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Locate Today's Session File(s)

Sessions live under `workspaces/{w}/sessions/{YYYY-MM-DD}-{github-username}.md` (one per workspace touched today). Find them by:

```bash
ls workspaces/*/sessions/$(date +%F)-*.md 2>/dev/null
```

If multiple workspaces have today's session, run this skill once per workspace (or aggregate per-area audit and write to each). If none exist, create them via `session-start` first. Never look for or create files under root `sessions/` or `sessions/YYYY/MM/`.

## Area-Aware Repository Audit

**Core requirement:** At the beginning of session-end, determine which areas the human partner worked on today.

**Defined Areas:**
`app`, `infra`, `platform`, `intelligence`, `brain`, `bot`, `superpowers`

**Process (do this first):**

1. Ask the user:  
   "Which areas did you work in today? (select all that apply from: app, infra, platform, intelligence, brain, bot, superpowers)"

2. Always include the `brain` repository.

3. For each selected area + brain:
   - Run git audit using Shell tool:
     - Today's commits: `git log --since="00:00" --oneline --pretty=format:"%h %s (%an, %ar)"`
     - Recent branches: `git branch --sort=-committerdate | head -8`
     - Status: `git status --short`
     - PRs (if gh available): `gh pr list --state=open --limit 5` or `gh pr list --author=@me`

4. Group and summarize results **per area**.

5. Present the summary to the user before proceeding with the normal questions.

6. Record the full per-area summary in the session file.

**Future improvement:** Add mapping file in brain repo (`audit-repos.yaml`) that maps area names to actual repository paths.

This directly solves the need to check different repositories based on role/context.

## Process

**First, run the Area-Aware Repository Audit** (see section above). Ask the user which areas they worked in today, run git checks on selected areas + brain, and show the grouped summary.

Then ask these questions **one at a time** (reference the audit results where relevant):

1. What did you work on today? (cross-reference with detected commits per area)
2. What decisions were made? (even small ones)
3. Were there any blockers? (or link to existing ones)
4. What is the plan for tomorrow?
5. Any additional context or corrections to the repository activity detected?

Always respond in English per the `always-english-output` rule, even if the human partner writes in Slovak.

## Template

Locate today's session file(s) at `workspaces/{w}/sessions/{YYYY-MM-DD}-{github-username}.md` (see "Locate Today's Session File(s)" above). If none exists for the relevant workspace, create one using the `session-start` template.

Update the "End of day" section with this **per-area structure**:

```markdown
## End of day

**Repository Activity (by area):**

**superpowers**
- Commits: 5 (including session-end improvements)
- PRs: 0 open
- Status: 3 modified files
- Notable: Updated session-end skill with area-aware audit

**brain**
- Commits: 2
- Status: clean

**app / platform**
- No activity detected today

**Worked on:** {answer to question 1, cross-ref with per-area commits}
**Decisions:** {answer to question 2, or "None" with reference to logged decisions}
**Blockers:** {answer to question 3, or "None"}
**Tomorrow:** {answer to question 4}
**Links/References:** {answer to question 5 + any PR/commit links, or "None"}
```

**IMPORTANT:** Always show activity grouped by the selected areas. This fulfills the "**record everything**" requirement per role/context.

## After Creation

- Incorporate the full Repository Audit summary into the session file (this fulfills the "**record everything**" requirement)
- **Use `para-memory-files` skill**:
  - Append key events + repo audit summary to today's daily note (`$AGENT_HOME/memory/YYYY-MM-DD.md`)
  - Extract 3-5 durable atomic facts (e.g. new skills implemented, English output rule created, repository audit added to session-end, user preference for Slovak input + English output) into appropriate PARA entity (likely `projects/superpowers` or `resources/ai-skills`)
  - Update `MEMORY.md` with any new tacit knowledge about user patterns (language preference, importance of repo transparency)
- If decisions or blockers were mentioned but not logged, offer to run `log-decision` or `log-blocker`
- If audit revealed uncommitted changes relevant to today's work, suggest committing them with appropriate messages before final push
- Commit the updated session file with message: `session(end): {github-username} — {YYYY-MM-DD} (with repo audit)`
- Push to remote — this is the one skill that always pushes. The brain repo now contains complete daily record including git activity.

## Red Flags

- Not asking the user which areas they worked in today (`app, infra, platform, intelligence, brain, bot, superpowers`)
- Auditing the wrong set of repositories (ignoring user selection)
- Not recording activity grouped by area
- Not recording *everything* ("record everything" requirement) in the session file
- Ending the day without performing the area-aware audit
- Ending the day without updating the session file
- Vague summaries instead of specific accomplishments cross-referenced with actual commits per area
- Forgetting to log important decisions or blockers during the day
- Not setting clear direction for the next day
- Not pushing the session context (including audit) to the shared brain repository
- "I'll remember what I did, no need to write it down" or "No changes today" without verifying with git
- Responding in Slovak or non-English

## Notes

- The skill is now **area-aware**. User selects areas at the start of session-end. Audit is performed only on selected areas + brain.
- Always trigger `para-memory-files` skill at the end (follow its rules exactly: write to daily note, extract atomic facts to `items.yaml`, update tacit knowledge in `MEMORY.md`).
- This skill always reads from and writes to (and pushes) the brain repository.
- Use `log-decision` and `log-blocker` *during* the day, not just at the end.
- The combination of per-area audit + session file + PARA memory creates accurate, role-specific persistent knowledge.
- For git command nuances on darwin (OS X), prefer `--since="00:00"` or `--since="midnight"`.
- Future: Add `brain/audit-repos.yaml` to map area names to full repository paths.
