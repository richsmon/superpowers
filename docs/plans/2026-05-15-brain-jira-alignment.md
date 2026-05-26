# Brain skills ↔ MarketClue brain alignment — remaining work

> Status doc. Captures what is done and what is left so the work survives a
> context reset. Created 2026-05-15.

## Context

`mc-superpowers` is the MarketClue fork of Superpowers. Its `brain-*` skills
were synced from `rs-superpowers`, but `rs-superpowers` was authored against a
personal brain repo (`tam-brain`) with a different, richer task-tracking model.
The real MarketClue brain (`github.com/market-clue/brain`) uses a simpler
model after its "Phase 1 restructure". A review found the synced skills did not
match the real brain repo; this doc tracks the alignment work.

The brain repo is designed as **source files + CI bridges to external
systems**: `.github/workflows/` has `jira-bridge.yml`, `knowledge-update.yml`,
`confluence-sync.yml`, `paperclip-trigger.yml`. Guiding principle:

> Skills write only **source files**. Derived / bridged artifacts
> (`REPO_KNOWLEDGE.md`, Jira, Confluence) are written by CI bridges.
> One writer per file = no drift.

## Decisions already made

1. **Jira is the system of record for tasks**, not the brain. The brain holds
   specs / decisions / knowledge. `tasks.md` per workspace is a lightweight
   rolling scratchpad, never the source of truth.
2. **Feature → Jira Epic; tasks → Stories** under that Epic. (`jira-bridge.yml`
   currently creates a Story per spec — to be changed to Epic.)
3. **Task-Stories under an Epic are created by the agent** via a Jira MCP
   server while expanding `plan.md` — no extra CI bridge for that.
4. **`REPO_KNOWLEDGE.md` is CI-owned.** Skills never edit it.
5. CI (`knowledge-update.yml`) indexes both top-level and `workspaces/*/`
   content.

## Done — Group A (merged)

- **market-clue/brain#4** (merged to `main`):
  - Added `templates/workspace/` — `BRIEF/OVERVIEW/STATUS/tasks/phases`
    templates + `archive/README.md` stop-sign.
  - Extended `.github/workflows/knowledge-update.yml` to scan `workspaces/*/`
    for features, decisions, blockers, sessions.
- **market-clue/superpowers#3** (merged to `develop`): synced the brain
  workflow skills + commands from `rs-superpowers`, genericized private
  project names, fixed session paths to `workspaces/{w}/sessions/`.
- **market-clue/superpowers#4** (merged to `develop`):
  - `brain-bootstrap-tracking`: template path → `templates/workspace/`.
  - `brain-new-workspace`: scaffolds the real 9-folder workspace shape.
  - Removed manual "update REPO_KNOWLEDGE.md" steps from
    `brain-new-workspace`, `brain-link-repo`, `brain-bootstrap-tracking`.
- **market-clue/superpowers#5** (merged to `develop`):
  - `brain-new-feature`: copies the brain repo's canonical
    `templates/feature/{spec,design,plan}.md` instead of an inline copy;
    now also scaffolds `design.md`.
  - `brain-bootstrap-tracking`: dropped the roadmap-seeding step
    (`architecture/roadmap.md`); phase naming → `phases/{name}.md`.

## Remaining — Group B (Jira alignment)

**Blocked on:** a running Jira instance, project key `MC`, an API token, and a
chosen Jira MCP server. Group B cannot complete until these exist.

### B1 — `jira-bridge.yml` → Epic (brain repo)
File: `market-clue/brain` → `.github/workflows/jira-bridge.yml`.
- Change issue creation from `Story` to `Epic` (one Epic per feature spec).
- Uncomment the `curl` block; wire up secrets `JIRA_BASE_URL`, `JIRA_USER`,
  `JIRA_TOKEN` (project key `MC`).
- The trigger marker stays `Route: human → Jira` written by `brain-ready-for-dev`.

### B2 — Add a Jira MCP server (harness)
- Interactive skills need to **read and transition** Jira issues, not just
  create them (CI handles creation).
- Pick a Jira MCP server (Atlassian official, or a connector), add it to the
  harness config used where the brain skills run.

### B3 — Rework 3 skills onto Jira keys (mc-superpowers)
Files: `skills/brain-next/SKILL.md`, `skills/brain-dispatch/SKILL.md`,
`skills/brain-reflect/SKILL.md` (+ their `commands/` wrappers).
- Replace the `T-NN` / structured-`tasks.md`-table assumption with **Jira
  issue keys** (`MC-123`) read via the Jira MCP.
  - `brain-next` — query Jira (project `MC`, my assignee, epic = active phase)
    → recommend the next issue. Drop `tasks.md` table parsing.
  - `brain-dispatch` task-mode — `--task MC-123`, fetch the Jira issue + the
    linked brain spec / decisions → bundle → dispatch.
  - `brain-reflect` task-outcome mode — transition the Jira issue status
    (To Do → In Progress → Done) instead of editing `tasks.md` rows.
- Phase concept → Jira **Epic**. `STATUS.md` "active phase" → Epic link.
- These are behavior-shaping skills; per `CLAUDE.md`, develop with
  `writing-skills` and pressure-test.

### B4 — Finalize workspace task/phase templates (brain repo)
File: `market-clue/brain` → `templates/workspace/`.
- `tasks.template.md` and `phases/_template.md` were already written in
  Group A with the Jira model in mind (tasks = rolling scratch; phase has a
  `Jira Epic:` field). Verify they still match once B1–B3 land; refine if the
  Epic/Story mapping changes anything.

## Remaining — MEDIUM review leftovers (fold into B3)

All remaining MEDIUM items live inside the Group B skills, so they are fixed
during the B3 rewrite (fixing them sooner would be throwaway work). The
non-Group-B MEDIUM items were resolved in superpowers#5.

- **`brain-next`** references `architecture/roadmap.md` and
  `workspaces/W/architecture/plans/{domain}/{task}.md`. Workspaces have no
  `architecture/` folder. Remove / repoint.
- **`brain-next`** phase naming assumes `phases/phase-N-<slug>.md`; the brain
  contract (`meta/workspace-tracking-system.md`) defines `phases/{name}.md`.
  Reconcile during the rewrite.
- **`STATUS.md` structure**: `brain-reflect` / `brain-dispatch` assume
  "In flight / Recently shipped / Blocked" tables + an "Active phase" line.
  The brain `STATUS.md` is looser. Either richen the STATUS template or loosen
  the skills.

## References

- Brain repo: `github.com/market-clue/brain`
  - `.github/workflows/jira-bridge.yml`, `knowledge-update.yml`
  - `meta/workspace-tracking-system.md` — the 5-artifact contract
  - `templates/workspace/` — workspace templates (added in brain#4)
- This repo: `github.com/market-clue/superpowers`
  - `skills/brain-*/SKILL.md`, `commands/brain-*.md`
- Note: the `paperclip` references in `brain-dispatch` are intentional —
  Paperclip is the planned agent-orchestration tool; left as-is for now.
