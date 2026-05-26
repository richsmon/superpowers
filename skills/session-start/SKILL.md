---
name: session-start
description: Use when beginning your daily work session or starting a focused work block.
---

# Session Start

## Overview

Creates a person-centric daily session file in the brain repository. **This is a per-DAY bookend, not a per-task ritual** — see "Per-Day Bookend" below. Always use English output per `always-english-output` rule (even for Slovak input). Pairs with `session-end` (mandatory repo audit). Use `token-efficient-communication` for all output to human partner.

## When to Use

At the beginning of a workday or focused work block to:
- Document priorities and context
- Create a reference point for the team and AI agents
- Establish the day's focus and constraints

## Per-Day Bookend (Not Per-Task)

**Run this ONCE per workday**, even if you will ship multiple features or PRs that day. `session-start` opens the daily diary; `session-end` closes it. Everything between them — decisions, blockers, multiple PRs — appends to the same file via `log-decision` / `log-blocker`.

Do NOT call `session-start` (or `/brain-session-start`):
- After each PR lands
- When starting a new feature mid-day
- When switching repos mid-day (use the `workspaces:` tag instead — see "Detect touched workspaces" below)
- "Just to be safe" mid-session

If a session file already exists for today, **update it** rather than creating a second file for the same person-day.

## Git Safety Protocol

Before reading or writing anything in the brain repo:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the session is created against a fresh tip.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Untracked session files from prior days are normal; surprising modified files are not — stop and confirm with the user.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` should be empty after pull. Surface any divergence before proceeding.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Resolve canonical username

The session filename uses the **canonical id from the people registry**, NOT a slug derived from `git config user.name`.

1. Read the current git identity: `git config user.email`.
2. Scan `graph/entities/people/*.yaml` for an entity whose `git_email` list contains that email.
3. If found → use that entity's `id` field as `{username}` for the session filename and H1.
4. If NOT found → **STOP and ask the user**:
   - "Your git email `<email>` is not registered. Which person entity does it belong to?"
   - List the existing people (`graph/entities/people/*.yaml` ids).
   - After the user confirms, **append the email to that entity's `git_email` list** so the next resolution succeeds without asking. If the user is new to the team, create a new entity from `templates/entities/person.yaml` with `git_email: ["<email>"]`.

Never invent a slug from `git config user.name`.

## Detect touched workspaces (`workspaces:` tag)

Human sessions are **person-centric**, not repo-centric — one file per person per day, regardless of how many repos were touched. The repo dimension goes into a `workspaces:` frontmatter tag.

1. If the user names workspaces in their request, use those.
2. Else read `REPO_KNOWLEDGE.md` / `AGENTS.md` for the active workspace(s) and propose.
3. Else infer from `.brain/config.json` or the cwd if it's inside a sibling code repo.
4. For brain-meta-only days, use `workspaces: []`.

The tag can be updated during the day or by `session-end` — it isn't locked at session-start.

## Process

Ask these three questions **one at a time**:

1. What is your main focus today?
2. Is there anything being carried over from yesterday?
3. Are there any important meetings, deadlines, or constraints today?

## Template

Creates `sessions/{YYYY}/{MM}/{YYYY-MM-DD}-{username}.md` at brain root.

**Body template — brain-side first, embedded fallback.** Before writing, look for a canonical template in the brain repo so format evolution does not require a fork PR:

1. If `<brain>/templates/session.md` exists, read it and substitute these placeholders verbatim (no other modification):
   - `{{WORKSPACES}}` — inline comma-separated workspace ids (e.g. `app, platform`); empty for brain-meta days
   - `{{USERNAME}}` — canonical id from people registry
   - `{{DATE}}` — `YYYY-MM-DD`
   - `{{FOCUS}}` — answer to question 1
   - `{{CARRIED_OVER}}` — answer to question 2
   - `{{MEETINGS_CONSTRAINTS}}` — answer to question 3
   Write the substituted result to the session path. Leave `## End of day` exactly as written in the brain template (it carries the placeholder text that `session-end` later replaces).

2. If `<brain>/templates/session.md` is missing, fall back to the embedded skeleton below.

If the brain repo's `AGENTS.md` "Session protocol" section specifies a different template path, that path wins — read from there.

**Embedded fallback (used only when no brain template is present):**

```markdown
---
workspaces: [{detected workspaces}]
---

# {username} — {YYYY-MM-DD}

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

- Create the directory if needed (`sessions/{YYYY}/{MM}/`).
- Confirm the session has been started: full path + resolved `{username}` + `workspaces:` tag.
- Do NOT commit — the session file is committed by `session-end` (see "Commit Policy" below).

## Commit Policy

**This skill does NOT commit.** The session file is created and left **untracked** until `session-end` bundles the whole day into a single commit at end of day.

Do not run `git add` or `git commit` here. Do not push. If you see yourself reaching for a commit, stop — `session-end` owns that.

## Red Flags

- Starting work without creating a session context.
- **Running `session-start` more than once per workday** — it is a per-DAY bookend, not a per-PR or per-task ritual.
- **Committing the session file from this skill.** session-end owns the commit; this skill only writes.
- **Guessing `{username}` from `git config user.name`** instead of resolving via the people registry.
- Writing the session under `workspaces/{repo}/sessions/` — that path is reserved for dispatched-agent run logs (`workspaces/{repo}/sessions/agents/`), not for humans.
- Skipping the `workspaces:` frontmatter tag.
- Writing vague or generic focus statements.
- Skipping this step because "it's just another day".
- Not updating the session file during the day with decisions and blockers.
- Skipping the Git Safety Protocol — creating the session against stale brain state.

## Notes

- If a session file already exists for today, **offer to update it** (do NOT create a second file for the same person-day).
- A multi-repo day stays in ONE file — append additional repos to the `workspaces:` tag.
- **Agent runs** (dispatched coding agents) are different: their logs live at `workspaces/{repo}/sessions/agents/`, not under the human `sessions/` tree.
- This skill always writes to the brain repository.
- The `/brain-session-start` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.
- Authoritative rule lives in the brain's `AGENTS.md` "Session protocol" section. If `AGENTS.md` contradicts this skill, AGENTS.md wins (the brain is the source of truth).
