---
name: design-handoff
description: Use when a designer has completed work in Figma and the feature already has a spec.md.
---

# Design Handoff

## Overview

Creates a design specification (`design.md`) that complements an existing feature spec in the brain repository workflow.

## When to Use

Use this skill when:
- A designer has completed work in Figma
- You have an existing `spec.md` for the feature
- Need to document screens, components, interactions, and edge cases for the CTO review

**Must be run after `write-feature-spec`.**

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root)
2. If not, check common sibling locations: `../brain`, `../../brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## Pre-check

1. Ask which feature this design is for — must match an existing `features/{feature-name}/` folder
2. Verify that `spec.md` exists for this feature
3. If no spec exists, stop: PM needs to run `write-feature-spec` first

## Process

Ask each question **one at a time**:

1. What is the Figma link (file or specific frame)?
2. Which screens/views are included?
3. What components are used (new vs existing)?
4. What interaction details are needed (animations, states, transitions)?
5. How should it behave responsively across breakpoints?
6. What accessibility considerations are there?
7. Are any new design tokens required?
8. What edge cases need visual documentation?

## Template

Creates `features/{feature-name}/design.md` with this structure:

```markdown
# Design: {Feature Name}

**Created:** {YYYY-MM-DD}
**Designer:** {github-username}
**Figma:** {figma-link}
**Status:** draft

## Screens

### {Screen 1 Name}
**Description:** {what this screen shows}
**Figma frame:** {link to specific frame if available}

### {Screen 2 Name}
**Description:** {what this screen shows}
**Figma frame:** {link to specific frame if available}

## Component Inventory

| Component | New/Existing | Notes |
|-----------|-------------|-------|
| {component} | {new/existing} | {notes} |

## Interactions

{Animations, transitions, hover states, loading states}

## Responsive Behavior

| Breakpoint | Behavior |
|------------|----------|
| Mobile (<768px) | {behavior} |
| Tablet (768-1024px) | {behavior} |
| Desktop (>1024px) | {behavior} |

## Accessibility

{Color contrast, screen reader, focus order notes}

## Design Tokens

{New colors, spacing, typography — if any}

## Edge Case Visuals

| State | Design |
|-------|--------|
| Empty state | {description or figma link} |
| Error state | {description or figma link} |
| Loading/skeleton | {description or figma link} |
| Max content | {description or figma link} |
```

## After Creation

- Do NOT commit — the design document is a draft that may be iterated on
- Inform user that the next step is for the CTO to run `ready-for-dev`

## Red Flags

- Creating design document before the spec exists
- Missing Figma references
- Incomplete coverage of edge cases or accessibility
- Moving to development without CTO review
- "The design is in Figma, we don't need a design.md"

## Notes

- `design.md` must be created alongside `spec.md` in the same feature folder
- Both files are **required** before `ready-for-dev` can proceed
- This skill always writes to the brain repository
