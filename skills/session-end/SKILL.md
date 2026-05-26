---
name: session-end
description: Use when ending your daily work session or wrapping up a focused work block. Performs mandatory repository audit (today's commits, PRs, branches) and records everything. Enforces English output.
---

# Session End

## Overview

Closes the daily session file with accomplishments, decisions, blockers, plans for tomorrow, **area-aware Repository Audit**, and PARA memory integration. **This is a per-DAY bookend, not a per-task ritual** — see "Per-Day Bookend" below. At the start of session-end, the user selects (or confirms) which areas they worked in today from: `app`, `infra`, `platform`, `intelligence`, `brain`, `bot`, `superpowers`. The skill then audits only those repositories + the brain repo for today's commits, PRs, and branches — across the **WHOLE day**, not the last task. Records everything and persists durable facts using `para-memory-files`. Commits and pushes to the brain repository. Always outputs in English.

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

## Per-Day Bookend (Not Per-Task)

**Run this ONCE per workday**, at the end. The repository audit covers the **WHOLE day** (`--since="00:00"`), not just the latest task — so calling `session-end` mid-day truncates the audit and breaks tomorrow's "what did we do yesterday" review.

Do NOT call `session-end` (or `/brain-session-end`):
- After each PR
- After each feature
- When closing a "focus block" mid-day
- Multiple times in one day to "checkpoint"
- "Just to be safe" before lunch

If the urge to checkpoint mid-day appears, use `log-decision` or `log-blocker` instead — those are per-event writes that append to the same daily diary. `session-end` is the one closer at end of day.

## Git Safety Protocol

This skill is the one brain skill that commits and pushes. Run all four checks before writing anything:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing — the session diary belongs on main, not on a feature branch.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the audit and commit land against a fresh tip.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Expected: today's session file (untracked) plus whatever `log-decision` / `log-blocker` wrote during the day. Unexpected: modified pipeline artifacts or design files — stop and confirm with the user (those probably belong on a feature branch, not this commit).
4. **Check branch state** — after the pull, `git -C <brain> log --oneline origin/main..HEAD` should be empty. Surface any divergence before committing.

If pipeline-sync work is sitting on `main` instead of its own branch, **do not bundle it into the session commit**. Move it to a feature branch first, then return to session-end. The session commit must contain only the daily diary.

## Signed Commits Required

**Every commit MUST be signed. Without a signature, do NOT commit.**

Before staging anything, verify signing is configured:

```bash
git -C <brain> config --get commit.gpgsign      # must print: true
git -C <brain> config --get user.signingkey     # must print a key id (not empty)
git -C <brain> config --get gpg.format          # 'openpgp' (default) or 'ssh'
```

If any of those return empty / not `true`, **STOP**. Do not commit. Tell the user signing is not configured and exit the skill. Never bypass with `--no-gpg-sign`, `-c commit.gpgsign=false`, or `commit --no-verify`. Never amend an existing unsigned commit just to land work — the policy is "no signature, no commit".

After committing, confirm the signature landed: `git -C <brain> log -1 --show-signature` should show a "Good signature" line.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Resolve canonical username

Human sessions are **person-centric**: ONE file per person per day, regardless of how many repos were touched. The filename uses the canonical id from the people registry, NOT a slug derived from `git config user.name`.

1. Read the current git identity: `git config user.email`.
2. Scan `graph/entities/people/*.yaml` for an entity whose `git_email` list contains that email.
3. Use that entity's `id` field as `{username}`.
4. If no match → STOP and ask the user, then **append the email to that entity's `git_email`** so the next resolution succeeds without asking. (See `session-start` for the full bootstrap flow.)

Never invent a slug from `git config user.name`.

## Locate Today's Session File

Human sessions live at `sessions/{YYYY}/{MM}/{YYYY-MM-DD}-{username}.md` at brain root. Find today's file:

```bash
ls "sessions/$(date +%Y)/$(date +%m)/$(date +%F)-{username}.md" 2>/dev/null
```

If it does not exist, create it via `session-start` first — never create a new session file inside `session-end`.

**Dispatched-agent runs** are different: they live at `workspaces/{repo}/sessions/agents/` and are audited independently of human sessions.

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

Locate today's session file at `sessions/{YYYY}/{MM}/{YYYY-MM-DD}-{username}.md` (see "Locate Today's Session File" above). If it does not exist, run `session-start` first.

**Session-file skeleton** is owned by `<brain>/templates/session.md` (read by `session-start`) — do not rewrite that here, just edit the existing `## End of day` block. This skill only fills the End-of-day body using the structure below.

Update the "End of day" section with this **per-area structure** and refresh the `workspaces:` frontmatter tag to list every code repo audited today (use `workspaces: []` for brain-meta-only days):

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
- Verify signing config per "Signed Commits Required". If unset, STOP and exit — do not commit.
- Commit the updated session file with message: `session(end): {username} — {YYYY-MM-DD} (with repo audit)`. The commit MUST be signed; verify with `git log -1 --show-signature` after committing.
- Push to remote — this is the one skill that always pushes. The brain repo now contains complete daily record including git activity.
- After the push succeeds, follow the `## Slack Status Report` section below to optionally stream a per-person status to `#daily-summary`. Slack failures are non-fatal — the session has already been saved.

## Slack Status Report

After the push succeeds, prompt the user **once**: "Post status to Slack?"

If they decline → skip silently, end the skill.

If they accept:

1. **Read the brain-side template.** Open `<brain>/templates/slack-status-report.md` and use the body (everything below the closing `-->` of the header comment). The header comment documents placeholders and Slack mrkdwn rules — read it once for context, do not include it in the message.

   Substitute placeholders:
   - `{{USERNAME}}` — canonical id
   - `{{DATE}}` — `YYYY-MM-DD`
   - `{{WORKSPACES}}` — comma-separated workspace ids
   - `{{WORKED_ON_BULLETS}}` — multi-line, each line prefixed with `• `; if empty render literal `_none_`
   - `{{PR_BULLETS}}` — multi-line `• <url|label> — short description`; `_none_` if empty
   - `{{DECISION_BULLETS}}` — multi-line `• `; `_none_` if empty
   - `{{BLOCKERS}}` — inline phrase or `none`
   - `{{TOMORROW}}` — single sentence

   If `<brain>/templates/slack-status-report.md` is missing, fall back to this embedded shape (Slack mrkdwn, single-asterisk bold):

   ```
   *{{USERNAME}} — {{DATE}}* · _workspaces: {{WORKSPACES}}_

   *Worked on*
   {{WORKED_ON_BULLETS}}

   *PRs*
   {{PR_BULLETS}}

   *Decisions*
   {{DECISION_BULLETS}}

   *Blockers:* {{BLOCKERS}}

   *Tomorrow:* {{TOMORROW}}
   ```

2. **Pick a delivery transport — tool-agnostic, no hardcoded MCP tool name.** Detect what is available in the current host, in this order:
   - Claude Code MCP: `mcp__claude_ai_Slack__slack_send_message` (or any equivalent in another MCP namespace).
   - Cursor MCP: the host's Slack MCP server, whatever it is named.
   - Brain bot REST: if you are running inside the bot, you already have `notifyStatusReport` from `bot/src/notifications/channels.js` — use it.
   - Webhook: a `SLACK_WEBHOOK_URL` env var, if exposed.
   - Manual: if nothing is available, print the rendered message in a fenced block and ask the user to paste it themselves.

3. **Channel.** Default: `#daily-summary` (per-person inputs). Production channel id: `C0AM8DVGRLG`. **Do NOT post to `#team-status-report`** — that channel is reserved for the bot's AI-summarized team rollup, not for individual `session-end` posts. The authoritative channel→id table lives at `<brain>/.cursor/rules/slack-channels.mdc`; consult it if the names ever change again.

4. **Failures are non-fatal.** If the Slack post fails for any reason (auth, network, missing MCP tool), log it, surface it to the user, and continue. The session file commit + push has already succeeded — do not roll it back, do not retry forever.

5. **Output language is English** per `always-english-output`, even if the conversation is in Slovak or another language.

## Commit Policy

**This skill commits and pushes — direct to `main`, no PR. The commit MUST be signed (see "Signed Commits Required" above).**

Why direct to main? The session file is a personal diary, not reviewable code. There is nothing for a teammate to gate; treating it as a PR-grade artifact wastes review time and breaks the daily cadence.

What lands in this commit:
- The day's session file (with morning, during-the-day, and end-of-day sections filled in)
- Any `decisions/` or `blockers/` files written by `log-decision` / `log-blocker` during the day — they were left untracked on purpose, and **this is where they get bundled in**

What does NOT belong in this commit:
- Pipeline-sync artifacts (e.g. `.sync-state/`, generated design files, brain-truth dumps) — these go on a feature branch with a PR. If they're sitting in `git status`, move them to a branch BEFORE committing here.
- Spec/design drafts for features — those land via their own commits when explicitly authorized by the author.

## Red Flags

- Guessing `{username}` from `git config user.name` instead of resolving via the people registry.
- Looking for / writing the session file under `workspaces/{repo}/sessions/` — human sessions live at brain root `sessions/{YYYY}/{MM}/`. That workspaces path is reserved for dispatched-agent run logs (`workspaces/{repo}/sessions/agents/`).
- Creating a second session file when one already exists for the same person-day — always update the existing file.
- Not refreshing the `workspaces:` frontmatter tag with the repos actually touched today.
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
- **Running `session-end` more than once per workday** — it is a per-DAY bookend, not a per-PR or per-task ritual. Use `log-decision` / `log-blocker` for mid-day events.
- **Truncated audit** — running session-end mid-day so the `--since="00:00"` query only catches the morning's work, leaving the afternoon's commits orphaned in tomorrow's review.
- **Bundling pipeline-sync work into the session commit.** Pipeline artifacts belong on a feature branch with a PR; the session commit carries only the daily diary.
- **Producing an unsigned commit.** No signature, no commit — full stop. Never use `--no-gpg-sign` or `-c commit.gpgsign=false` to push through.
- Skipping the Git Safety Protocol — committing/pushing against stale brain state or from the wrong branch.
- Skipping the Slack status report prompt after a successful push — the skill must ask once, even if the answer is "no". Posting silently without asking is also wrong.
- Posting the per-person status report to `#team-status-report` instead of `#daily-summary`. The team channel is reserved for the bot's AI rollup.
- Hardcoding a specific Slack MCP tool name (`mcp__claude_ai_Slack__slack_send_message`, etc.) as the only delivery path. The skill must detect what is available in the current host and pick.
- Letting a Slack post failure roll back or block the session commit. The commit + push is the source of truth; Slack is decoration on top.
- Responding in Slovak or non-English

## Notes

- The skill is now **area-aware**. User selects areas at the start of session-end. Audit is performed only on selected areas + brain.
- Always trigger `para-memory-files` skill at the end (follow its rules exactly: write to daily note, extract atomic facts to `items.yaml`, update tacit knowledge in `MEMORY.md`).
- This skill always reads from and writes to (and pushes) the brain repository.
- The `/brain-session-end` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.
- Use `log-decision` and `log-blocker` *during* the day, not just at the end.
- The combination of per-area audit + session file + PARA memory creates accurate, role-specific persistent knowledge.
- For git command nuances on darwin (OS X), prefer `--since="00:00"` or `--since="midnight"`.
- Future: Add `brain/audit-repos.yaml` to map area names to full repository paths.
