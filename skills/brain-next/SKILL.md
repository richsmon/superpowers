---
name: brain-next
description: Use when the user asks "what's next", "what should I work on", "where are we", "next task", or wants a focused recommendation of the next concrete action across active brain workspaces. Read-only orientation skill — does not modify any files. Returns a single recommended task plus the context preview that brain-dispatch would auto-bundle if invoked next.
---

# Brain — Next

Reads all active workspaces' tracking artifacts (`STATUS.md`, `tasks.md`, active `phases/phase-N-*.md`) and surfaces ONE next concrete action: which task to pick, in which workspace, why, and what context `brain-dispatch` would auto-bundle for it.

This skill is **strictly read-only**. It does not write any files. It exists to eliminate "where do I start" friction at the top of a per-task loop tick.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `meta/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- "what's next?", "what should I work on?", "next task", "where are we?"
- After `session-start` to orient on today's work
- Before `brain-dispatch` (so dispatch knows which task and skips manual selection)
- When deciding between multiple in-flight tasks

**Don't use** for:

- Full state overview across everything — use `brain-status` instead
- Recommending NEW work that's not in any phase — use `brainstorming` first
- Picking work in a single specific workspace where you already know the next task (just open `tasks.md`)

## Optional inputs

- `--workspace W` — scope to one workspace; default = scan all workspaces with active phases
- `--include-blocked` — also surface blocked tasks (default: surface separately at end, never recommend)

## Process

### 1. Find active workspaces with active phases

For each `workspaces/*/STATUS.md` that exists:

- Parse the `**Active phase:**` line.
- If it says `none yet` (or any variation indicating no active phase) → skip this workspace from "candidates" but record for "needs phase" footnote.
- Otherwise extract the phase file path (e.g. `phases/phase-1-mobile-end-to-end.md`).

### 2. Score candidate tasks within each active phase

Read the phase's "Scope — tasks in this phase" table. For each task ID listed, look up that row in `tasks.md`:

Priority order (highest first):

1. **`in-progress`** — finish before starting new. If multiple, recommend the one with the most recent `Updated` date (continuation wins over context-switch).
2. **`planned`** with all dependencies satisfied — a dependency is satisfied iff every task ID it depends on has `status: done` in `tasks.md`. Pick lowest `T-NN`.
3. **`blocked`** — surface separately at end with the blocker reason from the `Notes` field. NEVER recommend a blocked task as next action; surface only so the human knows.

Skip from the recommendation pool: `done`, `archived`, and `planned`-but-deps-unsatisfied (those go in a "waiting on" footnote).

Dependencies live in `architecture/roadmap.md` "Depends on" column or in the per-task plan front-matter. If neither is parseable, fall back to `tasks.md` `Notes` field where dependencies are often noted as "depends on T-NN".

### 3. Build context preview for the recommended task

For the chosen task, surface what `brain-dispatch --task T-NN` would auto-bundle. Do NOT actually open or read every file — just confirm existence and report:

- Plan file path from `tasks.md` `Plan` column (existence-check it)
- Related feature plan (`workspaces/W/features/{slug}/plan.md` if any feature folder mentions this task ID)
- Last dispatch outcome from `tasks.md` "Dispatch history" section (latest entry mentioning this T-NN)
- Last `Verified` date from the row
- STATUS snapshot summary: `{N} in-flight, {M} recently shipped`

### 4. Output format

```
Workspace: {W}
Active phase: Phase {N} — {theme}
              → workspaces/{W}/phases/phase-{N}-{slug}.md

Recommended next: {T-NN} — {title}
  Status: {status}
  Plan: {workspaces/W/architecture/plans/{domain}/{task}.md}
  Last verified: {YYYY-MM-DD or "never"}
  Last dispatch: {YYYY-MM-DD dispatch #N — outcome} or "never"
  Why this one: {"in-progress, continue" | "lowest planned T-NN with deps met"}

Context brain-dispatch would auto-bundle if invoked:
  - {plan path}
  - {feature plan path or "(no related feature)"}
  - STATUS snapshot ({N} in-flight, {M} recently shipped)
  - Dispatch history for {T-NN}: {N} prior entries
  - Active phase scope + success criteria

Recommended action:
  brain-dispatch --task {T-NN}
  (or: continue editing manually if you're already mid-edit)

Other tasks in scope (in-phase candidates not picked):
  - {T-NN} planned (deps met) — {title}
  - {T-NN} planned (waiting on T-NN done) — {title}

Blocked in scope (do not recommend; surfaced for awareness):
  - {T-NN} blocked — {blocker reason from Notes}

Other workspaces with active phases:
  - {W2}: Phase {N} → {T-NN} (use `brain-next --workspace {W2}` for full preview)
```

If running with `--workspace W`, omit "Other workspaces" section.

### 5. Special cases

#### No active phase anywhere

```
No active phase across {N} workspace(s) with tracking.

Workspaces with tracking but no active phase (write Phase 1 vision to activate):
  - {W1} — workspaces/{W1}/phases/phase-1-<slug>.md (copy from phases/_template.md)
  - {W2} — ...

Workspaces without tracking yet:
  - {W3} — run `brain-bootstrap-tracking` for {W3} first

Recommended action: write a Phase 1 vision (5 lines, your authorship) in one of the
above workspaces, then re-run brain-next.
```

#### Active phase but all tasks done

```
Workspace: {W}
Active phase: Phase {N} — {theme}
Status: ALL TASKS IN SCOPE ARE DONE.

Verify success criteria in workspaces/{W}/phases/phase-{N}-{slug}.md.
If all checked, close the phase:
  - Mark phase log entry "phase closed YYYY-MM-DD"
  - Copy phases/_template.md → phases/phase-{N+1}-<slug>.md
  - Write Phase {N+1} vision (5 lines)
```

#### Multiple in-progress in one phase

Surface all in-progress tasks. Recommend the one with most recent `Updated`. Add a one-line warning:

```
WARNING: {N} tasks in-progress in this phase. Context-switching is expensive.
         Strongly consider finishing {T-NN} (most recent) before continuing others.
```

## Red Flags

- Writing to ANY file (this skill is strictly read-only — abort and apologize if you catch yourself about to edit)
- Recommending an in-progress task be skipped for a planned one (always finish in-progress first; the only way to skip is human override)
- Suggesting a `blocked` task as next action without surfacing the blocker reason
- Picking a task whose dependencies aren't met
- Cross-workspace recommendation when current cwd implies a specific workspace (focus, don't distract — default to that workspace)
- Reading `archive/` folders to look up old context (per `MEMORY.md` archive contract)

## Notes

- Related: `brain-status` (full overview, multiple workspaces at once), `brain-dispatch` (next step after this; consume the recommendation), `brain-reflect` (after dispatch returns; use `--task T-NN --outcome <state>`).
- This skill is the entry point of every per-task loop tick: `brain-next` → `brain-dispatch --task T-NN` → [read output] → `brain-reflect --task T-NN --outcome <state>` → `brain-next` again.
- If `tasks.md` does not yet exist for a workspace, the workspace lacks tracking — point at `brain-bootstrap-tracking`.
