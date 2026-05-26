---
name: brain-new-feature
description: Use when the user wants to start a new feature inside an existing brain workspace, says "create feature", "new feature spec", "plan feature X", "start feature Y in workspace Z", or whenever non-trivial work needs to be captured as a feature with spec/plan/decisions before any code is written.
---

# Brain — New Feature

Creates `workspaces/{workspace}/features/{feature}/` with `spec.md`, `design.md` and `plan.md` scaffolded from the brain repo's `templates/feature/`, ready for `write-feature-spec` / `writing-plans` to fill in. Optionally orchestrates the full `brainstorming → write-feature-spec → writing-plans` Superpowers chain.

## Locate Brain Repo

1. Check if current workspace is the brain
2. If not, check siblings: `../brain`, `../../brain`
3. If not found, ask the user
4. **Sync state** — `git -C <brain> pull --rebase origin main` so the new feature scaffold is created against the latest templates and workspace state. Verify `git -C <brain> branch --show-current` is `main`.

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

Copy the brain repo's canonical feature templates — do NOT hand-write them, so
the feature stays consistent with `templates/feature/`:

```bash
mkdir -p workspaces/{workspace}/features/{feature}
cp templates/feature/spec.md   workspaces/{workspace}/features/{feature}/spec.md
cp templates/feature/design.md workspaces/{workspace}/features/{feature}/design.md
cp templates/feature/plan.md   workspaces/{workspace}/features/{feature}/plan.md
```

If `templates/feature/` is missing, abort and tell the user the brain repo
doesn't have the feature templates yet.

Then substitute the template placeholders (use your environment's edit tool,
not `sed`):

- `{feature-slug}` → `{feature}`
- `{Feature Title}` → the human-readable feature title
- `{github-username}` → the DRI's GitHub username
- `{repo}` / `{repo-name}` → the workspace's linked code repo
- `repos: [app]` → the workspace's repo
- `{YYYY-MM-DD}` → today's date

Leave `jira:` as `null` (spec) / `MC-{ticket}` (plan) — it is filled when the
feature is routed ready-for-dev and `jira-bridge.yml` creates the Jira Epic.

Leave every section *body* as the template prose. Do NOT invent Goal,
Acceptance criteria, or design content — those are for the human or the
`write-feature-spec` skill (step 2) to fill.

This skill scaffolds a **single-repo** feature (everything under
`workspaces/{workspace}/features/{feature}/`). A cross-repo feature instead
keeps `spec.md` at root `features/{feature}/` with per-repo `plan.md` files —
see the "Where things live" note in the spec template.

### 4. Update workspace entity (cross-references)

Open `graph/entities/{workspace}.yaml`. Append the feature to the entity's `content:` block as a reference, or add a feature-level entity if the feature is significant enough to be its own concept (e.g. `acme-web-auth-token`).

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
- Spec:   workspaces/{workspace}/features/{feature}/spec.md
- Design: workspaces/{workspace}/features/{feature}/design.md
- Plan:   workspaces/{workspace}/features/{feature}/plan.md

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
