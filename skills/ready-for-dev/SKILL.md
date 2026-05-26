---
name: ready-for-dev
description: Use when both spec.md and design.md exist for a feature and the CTO needs to approve it for development.
---

# Ready for Dev

## Overview

The final CTO quality gate. Reviews spec and design completeness, then routes to either human developer (Jira) or AI agent (Paperclip).

## When to Use

Use this skill when:
- Both `spec.md` and `design.md` exist for a feature
- The CTO needs to perform the final review and routing decision
- Deciding between creating a Jira ticket or a Paperclip issue

This is the **only authorized entry point** to development.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Git Safety Protocol

This skill commits a gate transition. Before any read or write:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing — the gate transition belongs on main.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` so the status flip lands against a fresh tip.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Expected: at most the spec/design files for this feature (drafts in progress). Unexpected: today's session file (that belongs to `session-end`, not this commit) or pipeline artifacts — confirm with the user before staging.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` should be empty after pull.

When committing in step "After Creation", stage **only** `features/{feature-name}/spec.md` (and `design.md` if the gate flip also affects it). Do not `git add -A` — keep today's diary out of this commit.

## Signed Commits Required

**The gate-transition commit MUST be signed. Without a signature, do NOT commit.**

Before staging anything, verify signing is configured:

```bash
git -C <brain> config --get commit.gpgsign      # must print: true
git -C <brain> config --get user.signingkey     # must print a key id (not empty)
git -C <brain> config --get gpg.format          # 'openpgp' (default) or 'ssh'
```

If any of those return empty / not `true`, **STOP**. Do not commit. Tell the user signing is not configured and exit. Never bypass with `--no-gpg-sign`, `-c commit.gpgsign=false`, or `commit --no-verify`.

After committing, confirm: `git -C <brain> log -1 --show-signature` should show a "Good signature" line. Report the verified commit hash to the user.

## Pre-check

1. Confirm the feature folder and both required files exist
2. If either `spec.md` or `design.md` is missing, stop and direct to the appropriate skill
3. Present a summary of both documents for review

## Completeness Checklist

Present this checklist to the CTO for sign-off:

- Problem is clearly defined and represents real user/business value
- Success criteria are measurable and verifiable
- All major user flows are documented
- Design fully covers the documented flows and screens
- Edge cases (empty, error, loading, edge data states) are addressed
- No unresolved open questions
- Security, compliance, and technical risks are identified
- Sufficient context exists in AGENTS.md for an AI agent (if applicable)

## Routing Decision

Ask the CTO:

**Route to human (Jira) or AI agent (Paperclip)?**

**Decision Guidelines:**

- **Human (Jira):** Novel creative work, new technology, complex refactoring, security-sensitive code, high ambiguity
- **AI Agent (Paperclip):** Clear spec + design, well-defined acceptance criteria, standard patterns, tests, documentation

## Template

This skill does not create a new file — it updates the existing `spec.md` with routing metadata.

### If Human (Jira)

1. Ask: **Which repo?** — `app`, `platform`, `intelligence`, or `infra`
2. Ask: **Assignee?** — team member name
3. Update spec status: `draft` → `ready-for-dev`
4. Add routing info to the spec:

```markdown
## Routing

**Routed:** {YYYY-MM-DD}
**Route:** human → Jira
**Repo:** {repo}
**Assignee:** {assignee}
**Jira:** _Create story: [{repo}] {feature-name} — link to spec.md_
```

### If AI Agent (Paperclip)

1. Ask: **Which repo?** — `app`, `platform`, `intelligence`, or `infra`
2. Update spec status: `draft` → `agent-ready`
3. Add routing info to the spec:

```markdown
## Routing

**Routed:** {YYYY-MM-DD}
**Route:** agent → Paperclip
**Repo:** {repo}
**Branch:** {repo}/MC-XXX-{feature-name}
```

4. Prepare agent context block:

```markdown
## Agent Context

**SPEC:** features/{feature-name}/spec.md
**DESIGN:** features/{feature-name}/design.md (Figma: {link})
**AGENTS.MD:** Relevant sections for {repo}
**BRANCH:** {repo}/MC-XXX-{feature-name}
```

## After Creation

- Verify signing config per "Signed Commits Required". If unset, STOP and exit — do not commit.
- Stage only the spec/design files for this feature (per Git Safety Protocol), then commit with message: `ready-for-dev(human): {feature-name}` or `ready-for-dev(agent): {feature-name}`. Commit direct to `main` — this is a gate transition, not pipeline work, and not part of the daily diary. The commit MUST be signed; verify with `git log -1 --show-signature` after committing.
- For Jira: tell the CTO to create a story with prefix `[{REPO}]`, link to spec, assign to team member
- For Paperclip: tell the CTO to create an issue with the agent context block

## Commit Policy

**This skill commits the gate transition direct to `main` — no PR. The commit MUST be signed (see "Signed Commits Required" above).** It stages only the spec/design files for this feature (NOT today's session file, NOT pipeline artifacts). Push is the user's call; this skill commits locally and surfaces the verified commit hash.

## Red Flags

- Attempting to bypass this gate
- Routing incomplete work to an AI agent
- Approving with unresolved checklist items
- **Producing an unsigned gate-transition commit.** No signature, no commit. Never use `--no-gpg-sign` or `-c commit.gpgsign=false` to push through.
- "This is urgent, just start building"
- CTO reviewing their own work without separation of concerns

## Notes

- Nothing is allowed to enter development without passing this gate
- Both `spec.md` and `design.md` are strictly required
- Significant routing decisions should be logged with `log-decision`
- This skill always operates on the brain repository regardless of current workspace
- The `/brain-ready-for-dev` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.
