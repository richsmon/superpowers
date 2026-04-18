---
name: brain-new-feature
description: Use when the user wants to start a new feature inside an existing brain workspace, says "create feature", "new feature spec", "plan feature X", "start feature Y in workspace Z", or whenever non-trivial work needs to be captured as a feature with spec/plan/decisions before any code is written.
---

# Brain — New Feature

Creates `workspaces/{workspace}/features/{feature}/` with `spec.md` and a `plan.md` stub, ready for the `writing-plans` skill to fill in. Optionally orchestrates the full `brainstorming → write-feature-spec → writing-plans` Superpowers chain.

## Locate Brain Repo

1. Check if current workspace is the brain
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- The user wants to start a meaningful piece of work in an existing workspace
- A feature is concrete enough to spec, but not yet implemented
- Before any `brain-dispatch` of implementation work

**Don't use** for:

- Bootstrapping a workspace itself — use `brain-new-workspace`
- Tiny tweaks / bug fixes — those don't need feature folders
- Recording a decision — use `log-decision` (Superpowers)

## Required inputs

Ask only for fields not obvious from context:

1. **Workspace** — must exist as `workspaces/{name}/`
2. **Feature name** — kebab-case (e.g. `mvp`, `capability-tokens`, `offline-outbox`)
3. **Domain / area** (optional, for the spec header)
4. **Whether to invoke `brainstorming` first** — recommended for non-trivial features

## Process

### 1. Verify workspace + feature path

```bash
ls workspaces/{workspace}/                    # must exist
ls workspaces/{workspace}/features/{feature}/ # must NOT exist
```

If the feature folder already exists, abort.

### 2. Optionally chain Superpowers skills

For non-trivial features:

- **First:** invoke `brainstorming` skill to crystallize the design — it will save a design doc the user can reference.
- **Then:** invoke `write-feature-spec` skill (the Superpowers one) — it knows the brain template and creates `features/{feature}/spec.md` with the standard sections (Problem, Goals, Success Criteria, Dependencies, References, Knowledge Graph Impact).

If the user wants a simpler / faster path (skip brainstorming), proceed directly to step 3.

### 3. Create the feature folder + files

```bash
mkdir -p workspaces/{workspace}/features/{feature}
```

`workspaces/{workspace}/features/{feature}/spec.md`:

```markdown
# Feature Spec: {Feature Title}

**Feature ID:** {workspace}-{feature}
**Workspace:** {workspace}
**Domain:** {domain or TBD}
**Priority:** {high/medium/low}
**Status:** Drafting (created {YYYY-MM-DD})

## Problem

{What problem are we solving? Why does it matter?}

## Goals

1. {Concrete, observable goal}
2. ...

## Success criteria

- {What must be true for this feature to be considered done?}

## Non-goals (out of scope)

- {What we are explicitly NOT building}

## Dependencies

- {Other features, decisions, or systems this depends on}

## References

- {Related entities, decisions, external docs}

## Knowledge graph impact

Entities to create / update:
- {entity ids}

---

**Next step:** Run `writing-plans` (Superpowers) to produce `plan.md`.
```

`workspaces/{workspace}/features/{feature}/plan.md` (stub):

```markdown
# {Feature Title} — Implementation Plan

**Feature:** {workspace}-{feature}
**Status:** Plan pending — generate via `writing-plans` skill

> This file will be filled in via the Superpowers `writing-plans` skill,
> which produces small bite-sized tasks (2–5 min each) with file paths
> and verification steps.
```

### 4. Update workspace entity (cross-references)

Open `graph/entities/{workspace}.yaml`. Append the feature to the entity's `content:` block as a reference, or add a feature-level entity if the feature is significant enough to be its own concept (e.g. `gotam-capability-token`).

Bump `lastUpdated:`.

### 5. Update today's session note

If a session file exists for today, add:

```markdown
**[HH:MM] Started feature:** {workspace}/{feature}
→ workspaces/{workspace}/features/{feature}/spec.md
```

### 6. Suggest next steps

```
Feature `{workspace}/{feature}` created.
- Spec: workspaces/{workspace}/features/{feature}/spec.md
- Plan: workspaces/{workspace}/features/{feature}/plan.md (stub — fill via writing-plans)

Next:
- writing-plans (Superpowers) → fill out plan.md
- brain-dispatch → once plan is ready, send Phase 0 / first batch to coder
```

## Red Flags

- Creating a feature without a clear problem statement
- Writing implementation in the spec instead of WHY/WHAT
- Skipping `brainstorming` for non-trivial features ("I already know what I want")
- Creating spec.md + immediately calling brain-dispatch — the plan must exist first

## Notes

- Related: `brain-new-workspace`, `brainstorming` (Superpowers), `write-feature-spec` (Superpowers), `writing-plans` (Superpowers), `brain-dispatch`.
- This skill writes to the brain. Do not auto-commit.
