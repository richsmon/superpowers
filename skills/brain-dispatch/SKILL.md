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

## Task-mode dispatch (auto-context)

When the user references a tracker task ID (`dispatch T-03`, `dispatch next`, or you have a recommendation from `brain-next`), use auto-context bundling instead of manual file selection. This is the standard mode for any workspace that has the 5-artifact tracking system (`tasks.md` + `phases/`).

### Trigger detection

- Explicit: user says "dispatch T-NN", "dispatch next", "implement T-NN"
- Implicit: just returned from `brain-next` with a recommendation in the conversation
- Optional flag: `--task T-NN`

### Process (auto-context)

For task `T-NN` in workspace `W`:

1. **Read tracker row.** Open `workspaces/W/tasks.md`, find the `T-NN` row. Extract: `Title`, `Phase`, `Status`, `Plan` (path), latest entries from "Dispatch history" section mentioning T-NN.

2. **Verify in-phase scope.** Read `workspaces/W/STATUS.md` → "Active phase" line → read that phase file → confirm T-NN is in the "Scope — tasks in this phase" table. If NOT in active phase, ask the user to confirm out-of-phase dispatch (could be intentional, could be a mistake).

3. **Refuse if status conflicts.** If T-NN is `done` (already finished) or `archived`, abort and ask the user to clarify intent. If `blocked`, surface the blocker reason and ask whether to proceed anyway.

4. **Auto-bundle context** in this order:

   - **Task header:** id, title, current status, plan path
   - **Plan content:** full content of the plan markdown from `Plan` column
   - **Active phase context:** vision (the 5-line block), success criteria, this task's "why in this phase" cell
   - **Related feature plan:** if `workspaces/W/features/{slug}/plan.md` references T-NN, include the relevant phase section
   - **STATUS snapshot:** current "In flight" + last 3 "Recently shipped" rows (so subagent knows recent context)
   - **Dispatch history for T-NN:** all rows from `tasks.md` "Dispatch history" mentioning this task (so subagent doesn't repeat past attempts or contradict prior outcomes)
   - **Baseline rule (REQUIRED):** explicit instruction per `MEMORY.md` 2026-04-19 rule — *"Before touching code, run the test suite and capture the baseline. STOP and report if the baseline is red; do not heroically fix unrelated tests."*
   - **Standard constraints:** pinned-deps policy, brain-first reflection (no brain edits from inside the agent), repo working dir
   - **Expected output:** subagent must return: outcome (one of: `done` | `in-progress` | `blocked`), files touched, commits made (hash + subject), baseline test counts before/after, learnings worth reflecting

5. **Show the assembled prompt to the user for approval.** Do not auto-fire — the human must see what's bundled and confirm. Token cost of an unwanted dispatch is high.

6. **Dispatch** via the chosen mechanism (Inline subagent or Paperclip — same as the existing modes). Pass the bundled prompt verbatim.

7. **After return, hand off to `brain-reflect`** with the task context preserved. Strongly suggest the user invoke `brain-reflect --task T-NN --outcome <state>` so the tracker is updated automatically. Do NOT manually edit `tasks.md` from this skill — that's `brain-reflect`'s job in task-outcome mode.

### Default values when not in task-mode

If the user dispatches without a task ID (free-text task description, exploratory work, or a workspace that doesn't have tracking), fall through to the manual "Process" section below. Task-mode is opt-in by referencing T-NN.

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
- Related: `brain-next` (recommends T-NN for task-mode), `brain-link-repo` (must run once per workspace), `brain-status`, `brain-reflect` (use task-outcome mode after dispatch returns), `paperclip` (vendor skill, used in Paperclip mode), `dispatching-parallel-agents` (Superpowers, for the underlying multi-agent pattern).
- Per-task loop: `brain-next` → `brain-dispatch --task T-NN` → [read output] → `brain-reflect --task T-NN --outcome <state>` → `brain-next` again.
