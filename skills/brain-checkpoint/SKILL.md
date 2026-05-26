---
name: brain-checkpoint
description: Use when the user wants a quick mid-work commit / save / checkpoint of the brain, says "commit", "save", "checkpoint", "rýchly commit", "ulož progres", "I need to step away", or wants to preserve state without losing where they were even though work isn't finished.
---

# Brain — Checkpoint

Saves the current state of **non-diary, non-pipeline** brain work even when it isn't done. Detects what changed, identifies what hasn't been written down yet (session entry, unfinished todos, in-flight feature progress, stale `lastUpdated:`), prompts for the missing pieces, then commits locally.

This skill exists for the common case: you've been working for an hour, you need to step away (meeting, lunch, end of focused block), but you're not at a clean stopping point — and you don't want to lose context. `session-end` is too heavy for this; a raw `git commit -am "wip"` loses the state.

**Scope (under the daily-bookend commit policy):** brain-checkpoint commits things like meta docs, atomic notes, spec/plan drafts, entity updates, and cleanup. It does **NOT** commit:

- **Today's session diary** (`sessions/YYYY/MM/{today}-{user}.md`) — `session-end` owns that.
- **Today's `decisions/` or `blockers/` writes from `log-decision` / `log-blocker`** — `session-end` bundles them.
- **Pipeline-sync artifacts** (e.g. `.sync-state/`, brain-truth dumps, design→Figma sync outputs) — those land via a feature branch + PR, never via checkpoint.

If `git status` shows ONLY these three categories, `brain-checkpoint` has nothing to do — defer to `session-end` (for the diary) or move pipeline work to a branch first.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`
3. If not found, ask the user

## Git Safety Protocol

Before snapshotting or committing:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not (e.g. you're on a pipeline-sync branch), this skill is the wrong tool — finish that branch via its PR flow instead.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the checkpoint commit lands on top of teammates' work, not behind it.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. This is what step 1 of the Process formalizes; surface anything surprising.
4. **Check branch state** — after the pull, `git -C <brain> log --oneline origin/main..HEAD` should be empty. If not, you have a prior local checkpoint that hasn't pushed — note it in the report.

## Signed Commits Required

**Every commit produced by this skill MUST be signed. Without a signature, do NOT commit.**

Before staging anything, verify signing is configured:

```bash
git -C <brain> config --get commit.gpgsign      # must print: true
git -C <brain> config --get user.signingkey     # must print a key id (not empty)
git -C <brain> config --get gpg.format          # 'openpgp' (default) or 'ssh'
```

If any of those return empty / not `true`, **STOP**. Do not commit. Tell the user signing is not configured and exit. Never bypass with `--no-gpg-sign`, `-c commit.gpgsign=false`, or `commit --no-verify`.

After committing, confirm: `git -C <brain> log -1 --show-signature` should show a "Good signature" line. Report the verified commit hash in the final summary.

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
- **Today's diary** (session file + today's `decisions/` / `blockers/` writes) → those wait for `session-end`
- **Pipeline-sync work** (`.sync-state/`, brain-truth dumps, design→Figma syncs) → move that work to a feature branch and ship it via PR; do NOT bundle it into a checkpoint on main

## Process

### 1. Snapshot current state

```bash
cd {brain}
git status --porcelain
git diff --stat
git diff --cached --stat
```

Group changes by area, then **partition into commit-eligible vs. deferred**:

**Commit-eligible** (brain-checkpoint may commit these):

- **Specs / plans** — `workspaces/*/features/*/spec.md`, `plan.md` (draft additions, not gate-status flips)
- **Knowledge** — `workspaces/*/knowledge/*.md`
- **Entities** — `graph/entities/*.yaml`
- **Meta** — `meta/*.md`, `AGENTS.md`, `REPO_KNOWLEDGE.md`, `templates/*`
- **Past-day decisions/blockers** — `decisions/*.md` / `blockers/*.md` that were authored on an earlier date (today's writes are deferred — see below)
- **Other** — config, `.cursor/`, etc.

**Deferred** (do NOT commit here):

- **Today's session file** — `sessions/{YYYY}/{MM}/{today}-{user}.md` → owned by `session-end`
- **Today's `decisions/` writes** — files dated today, written by `log-decision` → bundled by `session-end`
- **Today's `blockers/` writes** — files dated today, written by `log-blocker` → bundled by `session-end`
- **Pipeline-sync artifacts** — `.sync-state/`, generated design files, brain-truth dumps → belong on a feature branch + PR

If the deferred set is non-empty, **list it to the user** as "leaving these for session-end / a feature branch" and proceed only with the commit-eligible set. If the commit-eligible set is empty, this skill has nothing to do — tell the user and exit.

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
    workspaces/acme-app/knowledge/some-note.md

  ~ 4 modified files
    AGENTS.md (+18 / -3)
    REPO_KNOWLEDGE.md (+9 / -4)
    workspaces/acme-app/sessions/2026-04-18-richsmon.md (+24 / -0)
    graph/entities/acme-app.yaml (+1 / -1)

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

On confirmation, FIRST re-verify signing config (per "Signed Commits Required"). If unset, STOP — do not commit. Otherwise:

```bash
cd {brain}
# Stage ONLY commit-eligible paths (see step 1 partition).
# Do NOT `git add -A` — that would pull in today's diary and any pipeline artifacts.
git add <commit-eligible paths>
git commit -m "$(cat <<'EOF'
{message}
EOF
)"
# Verify the signature landed.
git log -1 --show-signature
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

## Commit Policy

**This skill commits locally to `main`. The commit MUST be signed (see "Signed Commits Required" above). It does NOT push.** Push is owned by `session-end` (and only `session-end` pushes the brain repo on the daily cadence).

What this commit MUST NOT contain:

- Today's session file
- Today's `decisions/` or `blockers/` writes (from `log-decision` / `log-blocker`)
- Pipeline-sync artifacts (`.sync-state/`, generated design files, brain-truth dumps)

If any of those slip in, `session-end`'s diary commit will collide with this one or duplicate entries. Partition strictly in step 1.

## Red Flags

- **Auto-adding today's session file or today's decisions/blockers** — those wait for `session-end`. Use partitioned `git add`, not `git add -A`.
- **Committing pipeline-sync work on main** — move it to a feature branch first; it ships via PR, not via checkpoint.
- **Producing an unsigned commit.** No signature, no commit. Never use `--no-gpg-sign` or `-c commit.gpgsign=false` to push through.
- Committing without checking if today's session reflects today's work (the brain becomes a mute archive)
- Auto-pushing — pushing is an explicit human decision
- Bundling clearly unrelated changes into one commit (offer to split)
- Hiding uncommitted changes in linked code repos (note them in session "Next")
- Treating `wip:` as an excuse to skip writing things down — `wip:` means "I'll continue", not "I'll forget"
- `git commit -am` shortcut without seeing the diff — always show the user what's going in
- Force-amending the previous commit just because the message is "nicer" — only amend if the previous commit was THIS session's AND not pushed (per general git safety rules)
- Skipping the Git Safety Protocol — committing against stale brain state or from the wrong branch.

## Notes

- Local commits only. Push is explicit and not part of this skill — `session-end` is the daily pusher.
- Heavier end-of-day protocol (End of Day section, push, optional PR, Paperclip task close-outs) → use `session-end` (Superpowers).
- This skill never touches linked code repos. Code commits happen inside the dispatched agent or by the user in the code repo itself.
- Related: `brain-reflect` (capture learnings BEFORE checkpointing if there are unwritten insights), `session-end` (heavier end-of-day, owns the daily diary commit), `log-decision` / `log-blocker` (per-event writes that wait for session-end).
