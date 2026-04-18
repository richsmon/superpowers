---
name: brain-link-repo
description: Use when the user wants to link a code repository to a brain workspace, register / connect / associate code with a workspace, set up a new project that needs both a brain namespace and a sibling code repo, or whenever future brain skills (brain-dispatch, brain-status) must know which folder corresponds to which workspace.
---

# Brain — Link Repo

Registers a code repository as the implementation target for a brain workspace. After linking, other brain skills (especially `brain-dispatch`) know which local folder to operate in when running agents on that workspace.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root, plus `graph/entities/`)
2. If not, check common sibling locations: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user for the path

All file operations happen in the brain repo.

## When to Use

- After creating a new code repo for a workspace (e.g. `mr-brainer`, `gotam`)
- When migrating an existing project into the brain workflow
- When the user says "link / register / connect this repo with workspace X"

## Required inputs

Ask only for fields not obvious from context. Use AskQuestion when available:

1. **Workspace name** — must already exist as `workspaces/{name}/`
2. **Local path to the code repo** — e.g. `/Users/{user}/code/mr-brainer`
3. **GitHub URL** (optional) — e.g. `https://github.com/{user}/mr-brainer`
4. **Default branch** — default `main`

## Process

### 1. Verify the workspace exists

```bash
ls workspaces/{name}/
```

If not, suggest running `brain-new-workspace` first. Stop. Do NOT silently create the workspace folder.

### 2. Verify the local path is a git repo

```bash
ls {localPath}/.git
```

If the path doesn't exist or isn't a git repo, ask the user whether to:

- skip the local path (link will be `localPath: null`, only `url` tracked), or
- abort and let the user create the repo first.

Never `git init` or `gh repo create` for the user automatically — those are explicit decisions.

### 3. Update the workspace entity

Find or create `graph/entities/{name}.yaml`. Add a `repo:` block at the top level (sibling to `id`, `type`, `title`...):

```yaml
repo:
  url: https://github.com/{user}/{repo}
  localPath: /Users/{user}/code/{repo}
  defaultBranch: main
  linkedAt: {YYYY-MM-DD}
  status: active
```

Bump `lastUpdated:` to today.

If the entity does not exist, create a minimal `project` type entity first using the same shape as other entities in `graph/entities/` (see `mr-brainer.yaml` as reference).

### 4. Update REPO_KNOWLEDGE.md

If the workspace is in the Active Workspaces table, ensure the row reflects active status. If not in the table, add it.

### 5. Confirm and report

```
Linked workspace `{name}` → {localPath}
GitHub: {url}
Entity: graph/entities/{name}.yaml updated.
You can now run `brain-dispatch` to send agents to this repo.
```

## Red Flags

- Silently `git init` or `gh repo create` for the user
- Linking a path that isn't a git repo without warning
- Overwriting an existing `repo:` block without confirming with the user
- Assuming the workspace exists — always verify first

## Notes

- Decisions about which repo a workspace points to are immutable in spirit. If a workspace is re-pointed to a new repo, write a decision note in `decisions/` explaining why.
- Related: `brain-dispatch` (uses the link), `brain-status` (shows all links), `brain-new-workspace` (creates the workspace before linking).
