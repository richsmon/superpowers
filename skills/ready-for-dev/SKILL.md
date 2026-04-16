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

- Commit with message: `ready-for-dev(human): {feature-name}` or `ready-for-dev(agent): {feature-name}`
- For Jira: tell the CTO to create a story with prefix `[{REPO}]`, link to spec, assign to team member
- For Paperclip: tell the CTO to create an issue with the agent context block

## Red Flags

- Attempting to bypass this gate
- Routing incomplete work to an AI agent
- Approving with unresolved checklist items
- "This is urgent, just start building"
- CTO reviewing their own work without separation of concerns

## Notes

- Nothing is allowed to enter development without passing this gate
- Both `spec.md` and `design.md` are strictly required
- Significant routing decisions should be logged with `log-decision`
- This skill always operates on the brain repository regardless of current workspace
