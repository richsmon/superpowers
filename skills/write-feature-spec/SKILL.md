---
name: write-feature-spec
description: Use when a new feature needs formal requirements before design or development can begin.
---

# Write Feature Spec

## Overview

Creates a complete feature specification (`spec.md`) that serves as the single source of truth in the brain repository workflow.

## When to Use

Use this skill when:
- A new feature needs to be formally defined
- PM, CEO, or product stakeholder wants to document requirements
- Starting the brain workflow (spec → design → CTO gate)

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Process

**1. Feature Name**
Ask for a short, kebab-case name (e.g. `market-alerts`, `user-onboarding`). This determines the folder name `features/{name}/`.

**2. Requirements Gathering (one question at a time)**

1. What problem does this solve?
2. Who is the target user?
3. What does success look like? (measurable criteria)
4. What are the key user flows?
5. What data is required?
6. What are the edge cases and error states?
7. What constraints exist (technical, timeline, dependencies)?
8. What is the priority level?

**3. Review & Create**
Present the compiled specification, get confirmation, then create the file.

## Template

Creates `features/{feature-name}/spec.md` with the following structure:

```markdown
# Feature: {Feature Name}

**Created:** {YYYY-MM-DD}
**Author:** {github-username}
**Priority:** {priority}
**Status:** draft

## Problem

{description}

## Target User

{description}

## Success Criteria

- [ ] {measurable criterion 1}
- [ ] {measurable criterion 2}

## User Flows

### Flow 1: {name}

1. {step}
2. {step}

## Data Requirements

{inputs, outputs, APIs, data sources}

## Edge Cases

{error states, limits, failure scenarios}

## Constraints

{technical, legal, timeline, dependencies}

## Open Questions

{anything unresolved}
```

## After Creation

- Do NOT commit — the spec is a draft that may be iterated on
- Next steps: Run `design-handoff`, then `ready-for-dev`

## Red Flags

- Starting implementation before spec is reviewed by CTO
- Vague or missing success criteria
- Skipping this step because "it's obvious"
- Not resolving open questions before moving forward
- Bypassing the brain workflow

## Notes

- This is the entry point to the brain repository workflow
- The spec remains a living document until `ready-for-dev` is completed
- All subsequent skills depend on this file existing in `features/{name}/spec.md`
- This skill always writes to the brain repository
