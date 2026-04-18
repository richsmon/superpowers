---
name: brain-new-workspace
description: Use when the user wants to create a new brain workspace, bootstrap a new project namespace inside the brain, says "create workspace X", "new workspace", "set up new project in brain", or whenever a new top-level area of work needs its own workspaces/{name}/ folder with the standard structure and entity stub.
---

# Brain — New Workspace

Bootstraps a new workspace inside the brain repo with the standard folder structure (`architecture/`, `decisions/`, `features/`, `knowledge/`, `sessions/`), a project-type entity stub in `graph/entities/`, and an entry in `REPO_KNOWLEDGE.md`.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- Starting a new project that should have a brain workspace
- Carving out a new long-term area of work (research, personal initiative, side product)
- The user explicitly says "new workspace X"

**Don't use** for:

- A new feature inside an existing workspace — use `brain-new-feature` instead
- A one-off note — use atomic note in an existing workspace
- A code-only experiment with no brain footprint — just create the repo

## Required inputs

Ask only for fields not obvious from context:

1. **Workspace name** — kebab-case, short (e.g. `mr-brainer`, `gotam`, `research`)
2. **Title** — human-readable name (e.g. "Mr. Brainer (mobile brain client)")
3. **Type** — usually `project` (also: `domain`, `research-area`)
4. **One-line description** — what is this workspace for?
5. **Tags** — comma-separated list (e.g. `mobile, ios, client`)
6. **Related entities** — which existing entities should this link to? (Always include `universal-brain` and at least one related concept.)

## Process

### 1. Verify workspace doesn't already exist

```bash
ls workspaces/{name}/
```

If it exists, abort and tell the user. Never overwrite.

### 2. Create folder structure

```bash
mkdir -p workspaces/{name}/{architecture,decisions,features,knowledge,sessions}
```

(Empty subfolders. Do not seed `.gitkeep` files unless the user wants to commit empty dirs immediately.)

### 3. Create the project entity

`graph/entities/{name}.yaml`:

```yaml
id: {name}
type: {type}
title: {Title}
aliases: []
tags: [{tags}]
projects: [{name}]
related:
  - universal-brain
  # plus user-provided related entities
created: {YYYY-MM-DD}
lastUpdated: {YYYY-MM-DD}
content: |
  {one-line description from user, possibly expanded into 2-3 sentences}

  {leave room for future content as the workspace evolves}
```

Do NOT add a `repo:` block here — that comes later via `brain-link-repo` once the actual code repo exists. (If the user already has a repo, suggest running `brain-link-repo` immediately after this skill.)

### 4. Update REPO_KNOWLEDGE.md

Add a row to the Active Workspaces table:

```markdown
| `{name}` | Planned | {one-line focus} | {priority} |
```

### 5. Update today's session note

If a session file exists for today, add a "During the day" entry referencing the new workspace:

```markdown
**[HH:MM] Created workspace:** `{name}` ({title})
→ workspaces/{name}/, graph/entities/{name}.yaml
```

### 6. Confirm and suggest next steps

```
Workspace `{name}` created.
- Folder: workspaces/{name}/
- Entity: graph/entities/{name}.yaml
- REPO_KNOWLEDGE.md updated.

Next steps you might want:
- brain-new-feature → create your first feature spec
- brain-link-repo → if you already have a code repo for this workspace
```

## Red Flags

- Overwriting an existing workspace
- Creating an entity without `related:` cross-links (the graph stays disconnected)
- Adding `repo:` block here — that's `brain-link-repo`'s job
- Creating workspace + immediately writing implementation — first capture WHAT and WHY (use `brain-new-feature` next)

## Notes

- Related: `brain-new-feature` (next logical step), `brain-link-repo` (when you have a code repo), `write-feature-spec` (Superpowers, used inside brain-new-feature).
- This skill writes to the brain. Do not auto-commit — let the user (or `session-end`) commit.
