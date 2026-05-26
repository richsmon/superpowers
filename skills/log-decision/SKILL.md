---
name: log-decision
description: Use when a process / business / ops / product decision has been made and needs to be recorded. For architectural / technical / security decisions with multiple interlocking sub-decisions, alternatives, and long-term consequences, use log-adr instead.
---

# Log Decision

## Overview

Records light decisions in the brain repository so the team and AI agents have context about why choices were made. For full architectural records (ADRs) with alternatives, supersedes, and amendments, use `log-adr` — both write into `decisions/` but the structure differs.

## When to Use

Use this skill when:
- Making a process / business / ops / product decision worth recording
- Wanting to document rationale and consequences for future reference
- A decision was made that future team members or agents should understand
- **Not for full ADRs.** For architectural / technical / security decisions with multiple interlocking sub-decisions and alternatives analysis, use `log-adr` — it has a separate template and process tuned for that genre. Both skills write into `decisions/`; the distinction is structural.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Git Safety Protocol

Before reading or writing in the brain repo:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the decision lands against a fresh tip and the session file you append to is the current one.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Today's session file should already be untracked from `session-start`; surprising modified files are not normal — confirm with the user.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` should be empty after pull.

## Process

Ask the user:

1. What was decided? (one-sentence summary)
2. Why was this decided? (rationale + trade-offs — for full alternatives analysis use `log-adr`)
3. What category is this? (`process`, `business`, `ops`, `product`) — for `architecture` / `technical` / `security` use `log-adr` instead
4. Who was involved in the decision? (use `Name (Role)` with Role matching `graph/entities/people/<id>.yaml`)

## Read Template

Read the template from the brain repo (single source of truth):

```
Read <brain>/templates/log/decision.md
```

If the file is missing, fail loudly with this message and stop:

> `templates/log/decision.md not found in <brain-path>. Either restore from git or update brain.`

Do NOT fall back to an embedded copy — there is no embedded copy.

The template defines the section structure (Decision, Context, Rationale, Consequences, optional Amendments, optional References) and the Date / Category / Participants / Status frontmatter. Fill placeholders from the answers to the Process questions above, then write the filled file to `decisions/YYYY-MM-DD-<slug>.md` in the brain repo.

**Participants Role:** the `Role` in `Participants: Name (Role)` MUST match the canonical role in `graph/entities/people/<id>.yaml`. Reconcile against the entity before writing.

**Cross-reference:** for full ADRs (architectural / technical / security with alternatives, supersedes, amendments) use `log-adr` instead.

## After Creation

- If a session file exists for today, add a reference in the "During the day" section:

```markdown
**[HH:MM] Decision:** {short summary}
→ decisions/YYYY-MM-DD-short-title.md
```

- **Do NOT commit.** See "Commit Policy" below.

## Commit Policy

**This skill does NOT commit.** It writes the decision file and appends a pointer to today's session diary, but stages nothing and commits nothing.

Why: `log-decision` fires repeatedly through the day. If each call committed, the daily diary would land 10+ tiny commits that all belong together as "today". `session-end` bundles every same-day write — session file, decisions, blockers — into one commit at end of day, alongside the repo audit.

If the decision is so consequential it needs to land immediately (rare), tell the user explicitly and let them decide whether to commit by hand. Do not auto-commit from this skill.

## Red Flags

- Making important decisions without documenting them
- Writing vague or generic decision records
- Not capturing alternatives that were considered
- Forgetting to update session file
- **Committing from this skill.** session-end bundles the daily writes — auto-committing here breaks the daily diary cadence.
- Skipping the Git Safety Protocol — appending to a stale session file or writing from the wrong branch.
- "It was obvious, no need to write it down"

## Notes

- Decisions are immutable. If reversed, create a new decision that supersedes the old one.
- Keep decisions atomic — one clear decision per file
- This skill always writes to the brain repository
- The `/brain-log-decision` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.
