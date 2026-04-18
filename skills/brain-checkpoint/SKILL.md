---
name: brain-checkpoint
description: Use when the user wants a quick mid-work commit / save / checkpoint of the brain, says "commit", "save", "checkpoint", "rýchly commit", "ulož progres", "I need to step away", or wants to preserve state without losing where they were even though work isn't finished.
---

# Brain — Checkpoint

Saves the current state of the brain even when work isn't done. Detects what changed, identifies what hasn't been written down yet (session entry, unfinished todos, in-flight feature progress, stale `lastUpdated:`), prompts for the missing pieces, then commits locally.

This skill exists for the common case: you've been working for an hour, you need to step away (meeting, lunch, end of focused block), but you're not at a clean stopping point — and you don't want to lose context. `session-end` is too heavy for this; a raw `git commit -am "wip"` loses the state.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- "I need to step away — commit what I have"
- End of a focused block, before context-switching
- Mid-feature, when you've written specs/decisions but haven't dispatched yet
- Whenever `git status` shows uncommitted brain changes you want to preserve
- After a `brain-dispatch` returns and you've reflected — want to checkpoint before next dispatch

**Don't use** for:

- Routine end-of-day → use `session-end` (Superpowers) — it has the fuller protocol (End of Day section, summary, push prompt)
- Pushing to remote → this skill commits LOCALLY; pushing is explicit and not part of this skill
- Code repos → this skill is brain-only; for linked code repos, the dispatched agent commits its own work

## Process

### 1. Snapshot current state

```bash
cd {brain}
git status --porcelain
git diff --stat
git diff --cached --stat
```

Group changes by area:

- **Decisions** — `decisions/*.md`, `workspaces/*/decisions/*.md`
- **Specs / plans** — `workspaces/*/features/*/spec.md`, `plan.md`
- **Knowledge** — `workspaces/*/knowledge/*.md`
- **Entities** — `graph/entities/*.yaml`
- **Session** — `sessions/YYYY/MM/YYYY-MM-DD-{user}.md`
- **Meta** — `meta/*.md`, `AGENTS.md`, `REPO_KNOWLEDGE.md`, `templates/*`
- **Other** — config, `.cursor/`, etc.

### 2. Identify gaps (what hasn't been written down)

Walk the table. For each gap, ask the user (or auto-fix if trivial and obviously correct).

| Signal                                                              | Gap                                          | Suggested fix                                                              |
|---------------------------------------------------------------------|----------------------------------------------|----------------------------------------------------------------------------|
| Decisions modified, but today's session has no entry referencing them | Decision made, not logged in session       | Append `**[HH:MM] Decision:** {short}` → today's session "During the day"  |
| Feature `plan.md` modified (phases checked) but no progress entry   | Phase progress not logged                    | Append progress entry to today's session                                   |
| Modified entity `lastUpdated:` is stale (older than today)          | Stale metadata                               | Bump `lastUpdated:` to today                                               |
| New atomic notes added, no `related:` cross-links                   | Disconnected from graph                      | Ask user for at least one related entity id                                |
| TodoWrite has pending items related to brain                        | Incomplete brain work                        | Write them to today's session "Next" section                               |
| In-progress feature exists but no `spec.md`                         | Untracked feature                            | Suggest `brain-new-feature` BEFORE checkpointing                           |
| Linked code repo (per `repo:` block) has uncommitted changes        | Code work not committed (out of scope here)  | Note it in session "Next"; this skill does NOT commit code repos           |
| Big mixed change (decisions + meta + skills + sessions all together) | Risk of unreadable commit                  | Suggest splitting into multiple commits (offer to split or proceed mixed)  |
| No session note exists for today, but brain has changes today       | Missing daily log                            | Suggest running `session-start` first; offer to skip on user override      |

### 3. Show diff summary to the user

Compact, scannable. NOT raw `git diff`.

```
Changes since last commit on `{branch}`:

  + 3 new files
    decisions/2026-04-18-brain-skills-in-superpowers-fork.md
    meta/brain-first-workflows.md
    workspaces/mr-brainer/knowledge/some-note.md

  ~ 4 modified files
    AGENTS.md (+18 / -3)
    REPO_KNOWLEDGE.md (+9 / -4)
    sessions/2026/04/2026-04-18-richsmon.md (+24 / -0)
    graph/entities/mr-brainer.yaml (+1 / -1)

  - 3 deleted files
    .cursor/skills/brain-link-repo/ (workspace-scoped variant)
    .cursor/skills/brain-dispatch/
    .cursor/skills/brain-status/

Summary:
  - 1 new top-level decision
  - 1 new meta document
  - 1 atomic note
  - Updates to AGENTS.md and REPO_KNOWLEDGE.md (point to fork)
  - Today's session note updated with the move
  - Cleaned up workspace-scoped skill copies
```

### 4. Propose a commit message

Match the brain's existing commit style (look at `git log --oneline -20` if unsure).

Format:

```
{type}: {short imperative subject ≤ 72 chars}

{1-3 short bullets — what changed and why}

{optional: pointer to decision / spec / session entry}
```

Type prefix:

| Prefix      | When                                                           |
|-------------|----------------------------------------------------------------|
| `decision:` | Single decision being committed                                |
| `spec:`     | Feature spec added/updated                                     |
| `plan:`     | Feature plan added/updated                                     |
| `notes:`    | Atomic notes only                                              |
| `entity:`   | Entity files only                                              |
| `meta:`     | AGENTS.md, REPO_KNOWLEDGE.md, meta/*, templates/*              |
| `session:`  | Session notes only                                             |
| `wip:`      | Mid-work checkpoint, work explicitly not done                  |
| `chore:`    | Cleanup, file moves, no logic change                           |

For mixed commits, pick the dominant area or use `mixed:` and split the body by area.

For mid-work, use `wip:` and include a "**Continuing:** {what's left}" line in the body so future-you knows where to pick up.

```
wip: brain-checkpoint skill drafted, testing pending

- skills/brain-checkpoint/SKILL.md drafted (routing rubric + commit policy)
- commands/brain-checkpoint.md wrapper added
- CHANGELOG entry added

Continuing: actually pressure-test the gap-detection table with subagent
before declaring this skill done. See workspaces/superpowers/features/brain-checkpoint/plan.md
```

### 5. Confirm and commit

Show the proposed message. Ask: "commit? (y / edit / abort / split)"

- **y** → proceed
- **edit** → let user edit the message, then proceed
- **abort** → stop, report what would have been committed
- **split** → guide user through splitting into multiple commits (use `git add -p` mentally, or stage groups by area)

On confirmation:

```bash
cd {brain}
git add -A
git commit -m "$(cat <<'EOF'
{message}
EOF
)"
```

### 6. Report

```
Committed {short-hash} on {branch}.
  {N} files changed, {+ins} insertions, {-del} deletions.

Working tree clean.
NOT pushed (run `git push` manually if you want it on remote).

Today's session note now reflects: {one-line of what was added to session}.
```

If anything in the "Next" section was added during the gap-fix step, surface it:

```
Heads up — these were added to today's "Next" section:
  - {item}
  - {item}
```

## Red Flags

- Committing without checking if today's session reflects today's work (the brain becomes a mute archive)
- Auto-pushing — pushing is an explicit human decision
- Bundling clearly unrelated changes into one commit (offer to split)
- Hiding uncommitted changes in linked code repos (note them in session "Next")
- Treating `wip:` as an excuse to skip writing things down — `wip:` means "I'll continue", not "I'll forget"
- `git commit -am` shortcut without seeing the diff — always show the user what's going in
- Force-amending the previous commit just because the message is "nicer" — only amend if the previous commit was THIS session's AND not pushed (per general git safety rules)

## Notes

- Local commits only. Push is explicit and not part of this skill.
- Heavier end-of-day protocol (End of Day section, push, optional PR, Paperclip task close-outs) → use `session-end` (Superpowers).
- This skill never touches linked code repos. Code commits happen inside the dispatched agent or by the user in the code repo itself.
- Related: `brain-reflect` (capture learnings BEFORE checkpointing if there are unwritten insights), `session-end` (heavier end-of-day), `log-decision` (if a decision needs to be split out into its own file before commit).
