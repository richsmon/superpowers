---
name: brain-dispatch
description: Use when the user wants to delegate coding work to an agent in a sibling code repo from inside the brain, says "dispatch", "send to agent", "send to coder", "implement spec X in repo Y", "run agent on workspace", "delegate to coder", or otherwise wants to do code work in a sibling repo without leaving the brain.
---

# Brain — Dispatch

Spawns a coding agent on a brain-workspace's linked code repo, with the relevant brain context (spec, plan, decisions, related entities, dependency manifest) bundled into the agent's prompt.

The human partner stays in the brain. The agent works in the code repo. Results flow back into the brain (session note, atomic notes, decision updates).

## Locate Brain Repo

Same as `log-decision` / `session-start`:

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- User says "dispatch on {workspace}", "implement Phase X", "send to coder"
- User describes work that belongs in a code repo, not in the brain
- User wants to delegate without opening the code repo themselves
- The task is concrete (has a spec or plan to point at)

**Don't use** for:

- Pure brain edits (decisions, atomic notes, session log) — just write directly
- Exploratory brainstorming — use `brainstorming` skill first to crystallize what to dispatch
- Tasks where the user wants to drive the work themselves

## Two execution modes

| Mode               | When                                                     | Mechanism                                                |
|--------------------|----------------------------------------------------------|----------------------------------------------------------|
| **Inline subagent** | Quick, scoped work; user wants iterative loop now        | `Task` tool with `subagent_type: shell` or `generalPurpose` |
| **Paperclip task**  | Longer work; needs governance / async; multi-heartbeat   | Create issue via Paperclip API (use `paperclip` skill)   |

If the user doesn't specify, ask: "Inline (blocking, results in this session) or Paperclip (async, tracked as issue, runs in heartbeats)?"

## Process

### 1. Identify workspace + linked repo

- Infer workspace from cwd if running inside `workspaces/{name}/`
- Otherwise ask which workspace
- Read `graph/entities/{name}.yaml` → `repo:` block
- If no `repo:` block exists, suggest running `brain-link-repo` first. STOP.
- If `repo.localPath` doesn't exist on disk, warn the user. STOP.

### 2. Identify the work

Possible inputs:

- Feature name → look in `workspaces/{name}/features/{feature}/(spec|plan).md`
- Spec / plan path
- Free-text task description
- Reference to a phase / checklist in plan.md

### 3. Bundle brain context

Construct a context bundle (read each, include relevant parts in the prompt):

1. **Mission summary** — one paragraph: who you are, where you are (`{repo.localPath}`), what to do, brain is the source of truth at `{brain-path}/workspaces/{name}/`.
2. **Spec** — relevant section of `workspaces/{name}/features/{feature}/spec.md`
3. **Plan** — relevant phase of `workspaces/{name}/features/{feature}/plan.md`
4. **Decisions** — list with one-line summaries:
   - All `workspaces/{name}/decisions/*.md`
   - Top-level `decisions/*.md` mentioning this workspace in their `Related` section
5. **Pinned dependency manifest** if applicable (`workspaces/{name}/knowledge/dependency-manifest.md`)
6. **Constraints** — repeat the most important global rules:
   - Pinned-deps policy (no `latest`, no SemVer ranges) where applicable
   - Brain-first reflection: decisions / learnings → brain BEFORE / DURING code, never silently
   - Anything else relevant from `AGENTS.md`
7. **Expected output** — what the agent should report back: file changes, commits made, blockers, learnings worth reflecting into brain.

### 4. Dispatch

#### Inline subagent

Use the `Task` tool. Choose `subagent_type`:

- `shell` — git ops, `npm install`, builds, native compiles
- `generalPurpose` — code writing + research mix
- `explore` — read-only exploration

The subagent must `cd {repo.localPath}` as its first action; subagents do not inherit your cwd:

```
Task({
  subagent_type: "...",
  description: "{phase} {workspace}",
  prompt: `{bundled context}

  Working directory: {repo.localPath}
  First action: cd {repo.localPath} && pwd && git status

  Then: {task description}

  When done, return:
  - Summary of what was done
  - Files created / modified (paths only)
  - Commits made (hash + subject)
  - Any blockers or open questions
  - Learnings worth reflecting into brain`
})
```

#### Paperclip task

Use the `paperclip` skill to create a task. Title format: `[{workspace}] {short task title}`. Description includes:

- Link to brain spec / plan path
- Bundled context (or summary + paths to read)
- Working directory hint
- Same expected-output checklist

Set `parentId` and `goalId` if user provides them. Always include `X-Paperclip-Run-Id` header per `paperclip` skill rules.

### 5. Reflect results into brain

When the inline subagent returns (or after a Paperclip heartbeat completes):

1. Append a "Dispatched work" entry to today's session (`sessions/YYYY/MM/YYYY-MM-DD-{user}.md`):

   ```markdown
   ### Dispatched: {workspace} — {task}
   - Mode: inline subagent / paperclip task {id}
   - Agent summary: {one-line}
   - Files touched: {list}
   - Commits: {list}
   - Brain follow-ups: {atomic notes / decisions to write}
   ```

2. If the agent reported new learnings, prompt the user (or directly create if user pre-approved) atomic notes in `workspaces/{name}/knowledge/`.
3. If a meaningful decision was made, prompt for a decision note in `workspaces/{name}/decisions/`. Consider invoking the `brain-reflect` skill explicitly for this step.
4. Update `graph/entities/{name}.yaml` → bump `lastUpdated:` and optionally `repo.lastDispatch:`.

## Red Flags

- Dispatching without checking that `repo:` link exists in the entity
- Stuffing the entire brain into the prompt (be selective: only what's relevant to THIS task)
- Letting the dispatched agent write brain files (separation of concerns — brain edits happen in parent agent / by the user)
- Forgetting to reflect results back into the session note
- Dispatching exploratory work — use `brainstorming` first to crystallize what you actually want

## Notes

- For long-running shell operations (npm install, builds), pass `block_until_ms` appropriately if using `Task` with `shell` subagent.
- Secrets: never include API keys / tokens in the prompt. The dispatched agent reads them from its own env / SecureStore equivalent.
- Related: `brain-link-repo` (must run once per workspace), `brain-status`, `brain-reflect`, `paperclip` (vendor skill, used in Paperclip mode), `dispatching-parallel-agents` (Superpowers, for the underlying multi-agent pattern).
