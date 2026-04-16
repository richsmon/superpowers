---
name: log-decision
description: Use when an important architectural, technical, or business decision has been made and needs to be recorded.
---

# Log Decision

## Overview

Records decisions in the brain repository so the team and AI agents have context about why choices were made.

## When to Use

Use this skill when:
- Making a significant technical, architectural, or business decision
- Wanting to document trade-offs and rationale for future reference
- A decision was made that future team members or agents should understand

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Process

Ask the user:

1. What was decided? (one-sentence summary)
2. Why was this decided? (rationale, alternatives considered, trade-offs)
3. What category is this? (`architecture`, `business`, `technical`, `process`, `security`)
4. Who was involved in the decision?

## Template

Creates `decisions/YYYY-MM-DD-{short-title}.md` with this structure:

```markdown
# {Decision Title}

**Date:** {YYYY-MM-DD}
**Category:** {category}
**Participants:** {who was involved}
**Status:** accepted

## Decision

{One sentence summary of what was decided}

## Context

{Why this decision was needed}

## Rationale

{Why this option was chosen, what alternatives were considered and why they were rejected}

## Consequences

{What changes as a result of this decision}
```

## After Creation

- If a session file exists for today, add a reference in the "During the day" section:

```markdown
**[HH:MM] Decision:** {short summary}
→ decisions/YYYY-MM-DD-short-title.md
```

- Commit with message: `decision: {short-title}`

## Red Flags

- Making important decisions without documenting them
- Writing vague or generic decision records
- Not capturing alternatives that were considered
- Forgetting to update session file
- "It was obvious, no need to write it down"

## Notes

- Decisions are immutable. If reversed, create a new decision that supersedes the old one.
- Keep decisions atomic — one clear decision per file
- This skill always writes to the brain repository
