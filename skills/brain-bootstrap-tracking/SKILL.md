---
name: brain-bootstrap-tracking
description: Use when the user wants to add the standard 5-artifact tracking system (BRIEF, OVERVIEW, STATUS, phases/, tasks.md) to a brain workspace, says "bootstrap tracking", "add tracking to workspace X", "set up status/tasks/phases for workspace", or whenever an active workspace lacks the workspace tracking pattern documented at `meta/workspace-tracking-system.md`.
---

# Brain — Bootstrap Tracking

Adds the standard workspace tracking system to an existing workspace. The pattern: 5 top-level artifacts (`BRIEF.md`, `OVERVIEW.md`, `STATUS.md`, `phases/phase-N-*.md`, `tasks.md`) backed by reusable templates in `meta/templates/workspace/`.

This skill scaffolds the artifacts from those templates, substitutes placeholders, drops the archive stop-sign if applicable, and suggests next-step prompts for the human (write BRIEF, draft phase 1 vision, seed tasks).

It does **not** auto-fill the human-authored content (BRIEF pitch, phase vision). Those slots are deliberately left as templates so the human writes them.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `meta/templates/workspace/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- An existing workspace becomes "active" and now needs tracking artifacts
- A workspace was created with `brain-new-workspace` but never got tracking
- The user explicitly says "bootstrap tracking for X" or "add status/tasks/phases to X"

**Don't use** for:

- Creating a brand-new workspace — use `brain-new-workspace` first, then this skill
- Adding a single feature to an existing workspace — use `brain-new-feature`
- Re-bootstrapping a workspace that already has tracking — refuse and tell the user (never clobber); offer to update individual files instead

## Required inputs

Ask only for fields not obvious from context:

1. **Workspace name** — kebab-case, must already exist at `workspaces/{name}/`
2. **Whether to seed `tasks.md` from existing roadmap** — yes/no (default yes if `architecture/roadmap.md` has a recognizable task list with `[x]/[ ]` checkboxes or a numbered task table)

## Process

### 1. Verify workspace exists and tracking does NOT already exist

```bash
ls workspaces/{name}/                           # must exist
ls workspaces/{name}/BRIEF.md 2>/dev/null && echo CLOBBER-RISK
ls workspaces/{name}/STATUS.md 2>/dev/null && echo CLOBBER-RISK
ls workspaces/{name}/tasks.md 2>/dev/null && echo CLOBBER-RISK
ls workspaces/{name}/phases/ 2>/dev/null && echo CLOBBER-RISK
```

If ANY of `BRIEF.md`, `OVERVIEW.md`, `STATUS.md`, `tasks.md`, or `phases/` already exists, abort. Tell the user which file blocks and ask whether to proceed file-by-file (skipping existing) or stop entirely.

### 2. Verify templates are available

```bash
ls meta/templates/workspace/
# Must contain: BRIEF.template.md, OVERVIEW.template.md, STATUS.template.md,
# tasks.template.md, phases/_template.md, archive/README.md
```

If templates are missing, abort and tell the user the brain doesn't have the template directory yet — they need to install it first (or this skill is being run on a brain that pre-dates the tracking system).

### 3. Copy templates into the workspace

For each of the 5 artifacts, copy the template to its final path and substitute `{WORKSPACE_NAME}` with the workspace's title (from `graph/entities/{name}.yaml` `title:` field, falling back to the kebab-case name capitalized).

```bash
cp meta/templates/workspace/BRIEF.template.md     workspaces/{name}/BRIEF.md
cp meta/templates/workspace/OVERVIEW.template.md  workspaces/{name}/OVERVIEW.md
cp meta/templates/workspace/STATUS.template.md    workspaces/{name}/STATUS.md
cp meta/templates/workspace/tasks.template.md     workspaces/{name}/tasks.md
mkdir -p workspaces/{name}/phases
cp meta/templates/workspace/phases/_template.md   workspaces/{name}/phases/_template.md
```

Then substitute the placeholder in each (use the writing tool of your environment; do not use `sed` if your environment provides a structured edit tool):

- Replace `{WORKSPACE_NAME}` → workspace title in: `BRIEF.md`, `OVERVIEW.md`, `STATUS.md`, `tasks.md`
- Replace `YYYY-MM-DD` → today's date in the "Last updated" lines

### 4. Drop the archive stop-sign (if archive exists)

```bash
if [ -d workspaces/{name}/archive ] && [ ! -f workspaces/{name}/archive/README.md ]; then
  cp meta/templates/workspace/archive/README.md workspaces/{name}/archive/README.md
fi
```

If `archive/` exists but lacks the stop-sign README, drop the template in. Then briefly enumerate what's in `archive/` (one `ls`) and append a "What lives here" section to the dropped README, listing the top-level entries with one-line descriptions where obvious.

### 5. Optionally seed tasks.md from existing roadmap

If the user said yes (or if `architecture/roadmap.md` has a parseable task list and the user did not say no):

1. Read `architecture/roadmap.md` and extract the task list (look for numbered tables `| # | Task | File | ... |` and `[x] #N Task Name` checklists).
2. Replace the placeholder row in `tasks.md` with one row per discovered task. Default `Phase` to `—`, default `Status` to `done` if the source had `[x]`, otherwise `planned`. Default `Verified` and `Updated` to `—` (or today's date for items derived from a `[x]` checkbox).
3. If the source roadmap had a `[x]/[ ]` state checklist that you used as the source of truth, REMOVE it from the roadmap (replace with a pointer to `tasks.md`) so there is no drift. Make this a separate edit; preserve the dependency graph and milestone structure.

### 6. Update OVERVIEW.md "State in one paragraph"

Replace the placeholder paragraph in `OVERVIEW.md` with a concrete one-paragraph state derived from:

- The number of tasks and how many are `done` / `in-progress` (from `tasks.md`)
- The active phase (none yet — say "no phase active; create the first phase via the human writing 5-line vision in `phases/phase-1-<slug>.md`")
- The linked code repo path (from `graph/entities/{name}.yaml` `repo.localPath` if present)

Keep it factual; do not invent traction or progress claims.

### 7. Update REPO_KNOWLEDGE.md

Append a one-line entry to "Recent Decisions":

```markdown
- YYYY-MM-DD: Bootstrapped workspace tracking (5-artifact pattern) for `{name}` — `workspaces/{name}/{BRIEF,OVERVIEW,STATUS,tasks}.md` + `phases/`
```

### 8. Update today's session note

If a session file exists for today in `workspaces/{name}/sessions/`, append:

```markdown
**[HH:MM] Bootstrapped tracking for `{name}`** — BRIEF/OVERVIEW/STATUS/tasks/phases scaffolded from meta/templates/workspace/.
Next: human writes BRIEF pitch + Phase 1 vision (5 lines) in `phases/phase-1-<slug>.md`.
```

### 9. Confirm and surface human-required next steps

```
Tracking bootstrapped for `{name}`.

Files created:
  workspaces/{name}/BRIEF.md       (TEMPLATE — needs your pitch)
  workspaces/{name}/OVERVIEW.md    (state-paragraph filled, index ready)
  workspaces/{name}/STATUS.md      (TEMPLATE — fill in flight after first dispatch)
  workspaces/{name}/tasks.md       ({N} tasks seeded from roadmap | TEMPLATE)
  workspaces/{name}/phases/_template.md
  workspaces/{name}/archive/README.md  (if archive existed)

Human-required next steps:
  1. Write the BRIEF pitch (~300 words) — answer WHAT, WHY NOW, FOR WHOM, HOW
  2. Pick a phase 1 theme and write the 5-line vision in
     workspaces/{name}/phases/phase-1-<slug>.md
     (copy from phases/_template.md and rename)
  3. Once vision is set, ask an agent to expand into epics + stories +
     pick which tasks from tasks.md belong in this phase

The agent NEVER fills in BRIEF or phase vision content. Those slots are
human authorship; the agent only expands once the vision exists.
```

## Red Flags

- Clobbering existing `BRIEF.md` / `STATUS.md` / `tasks.md` / `phases/` — never; abort instead
- Inventing content for `BRIEF.md` (pitch is human authorship) or `phases/phase-N-*.md` vision (human authorship); only fill `OVERVIEW.md` state-paragraph and seed `tasks.md` from existing data
- Frame phases in time units ("Phase 1: Q1", "2-week sprint") — phases are scope-bound; reject and rewrite if the human's draft uses time language
- Duplicating task state in `roadmap.md` checklist after seeding `tasks.md` — strip the checklist (Step 5) so there's only one source of truth
- Reading `archive/` to enumerate it — the stop-sign exists for a reason; just `ls` (one level), do not open files inside

## Notes

- Related: `brain-new-workspace` (must run first if workspace doesn't exist), `brain-new-feature` (use after first phase has tasks scoped), `brain-status` (will surface workspaces missing tracking).
- Contract docs: `meta/workspace-tracking-system.md` in the brain repo.
- This skill writes to the brain. Do not auto-commit — let the user (or `session-end`) commit.
- If the user's brain pre-dates this template directory, point them at `meta/workspace-tracking-system.md` to install templates first.
