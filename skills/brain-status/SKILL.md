---
name: brain-status
description: Use when the user asks for brain status, says "what's going on", "where am I", "lay of the land", "show me the brain", "give me the lay of the land", or at the start of a session to orient on active workspaces, linked repos, recent decisions, and today's session state.
---

# Brain — Status

Prints a compact, scannable status of the brain so the user (or agent) can orient quickly without manually reading multiple files.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`
3. If not found, ask the user
4. **Sync state** — `git -C <brain> pull --rebase origin main` so the status reflects the real tip. Verify `git -C <brain> branch --show-current` is `main`.

## When to Use

- Beginning of a work session, to remember context
- After a few days away from a project
- Before deciding what to work on next
- When the user asks "kde som / kde čo je / status / where am I"

## What to gather

Read or enumerate (don't read every file fully — use globs):

1. `REPO_KNOWLEDGE.md` — Active Workspaces table, Recent Decisions list
2. `graph/entities/*.yaml` — for each entity with `type: project`, extract `repo:` block (linked path + URL) if present
3. `workspaces/*/sessions/YYYY-MM-DD-{user}.md` — today's session (if exists)
4. `decisions/` — last 5 by date (sorted desc by filename)
5. For each workspace, count:
   - features in `workspaces/{name}/features/*/` (with spec.md)
   - decisions in `workspaces/{name}/decisions/*.md`
   - knowledge notes in `workspaces/{name}/knowledge/*.md`

## Output format

Compact markdown. NOT a wall of text. Target: fits one screen.

```markdown
# Brain status — {YYYY-MM-DD}

## Today's session
{1-line "Focus" from session OR "no session started yet — run session-start"}

## Active workspaces

| Workspace     | Repo linked?                      | Features | Decisions | Notes |
|---------------|-----------------------------------|----------|-----------|-------|
| `acme-app`    | ~/code/acme-app (planned)         | 1        | 4         | 2     |
| `acme-web`    | ~/code/acme-web                   | 3        | 0         | 0     |
| `paperclip`   | (no repo link)                    | 0        | 0         | 0     |

## Recent decisions (last 5)

- 2026-04-18 redis-cache-over-memcached (top-level)
- 2026-04-18 pinned-deps-policy (top-level)
- 2026-04-18 api-versioning-scheme (acme-app)
- 2026-04-18 ci-runner-choice (acme-app)
- 2026-04-16 brain-workflow-adoption (top-level)

## Suggested next actions
- {derived from session 'Next' section if present}
- {or: "run brain-dispatch to start work on {most recent feature}"}
```

## Red Flags

- Reading every YAML / markdown file (too slow — enumerate + read summary files only)
- Hiding gaps (if `repo:` link missing, say so plainly; if no session today, say so)
- Walls of text (one screen max; truncate lists at 5–10 items)
- Editing brain files (this skill is read-only — never modifies brain)

## Notes

- Pure read skill. Never modifies the brain.
- Related: `session-start` (if no session today), `brain-link-repo` (if a workspace has no `repo:`), `brain-dispatch` (to start actual work).
