# Changelog

## [richsmon-fork] - 2026-04-18

### Added (fork-only — brain-first workflow)

- **8 brain-* skills** for the brain-first workflow, where the user stays in the `tam-brain` repo and dispatches agents to sibling code repos:
  - `brain-link-repo` — register a code repo as a workspace's implementation target (`repo:` block in `graph/entities/{name}.yaml`)
  - `brain-dispatch` — spawn a coding agent (inline subagent or Paperclip task) on the linked repo with bundled brain context (spec, plan, decisions, dependency manifest, constraints)
  - `brain-status` — one-screen overview of active workspaces, linked repos, today's session, recent decisions
  - `brain-new-workspace` — bootstrap `workspaces/{name}/` with standard structure + `project` entity stub + REPO_KNOWLEDGE.md row
  - `brain-new-feature` — create `workspaces/{w}/features/{f}/` with spec.md + plan.md stub, optionally chaining `brainstorming` → `write-feature-spec` → `writing-plans`
  - `brain-reflect` — triage takeaways from completed work into atomic notes / decisions / entity updates / follow-ups
  - `brain-route` — triage where a piece of work belongs (brain vs code repo) using a routing rubric
  - `brain-checkpoint` — quick mid-work commit: detects state, fills gaps (session entry, "Next" items, stale `lastUpdated:`), proposes message, commits locally (no auto-push)
- **8 thin slash-command wrappers** in `commands/` (matches existing `commands/brain-*` convention from upstream brain wrappers like `brain-log-decision`, `brain-session-start`)

### Notes

- These are fork-only extensions; not for upstream PR (per upstream `CLAUDE.md` — "Domain-specific skills … belong in their own standalone plugin").
- Designed to compose with vendor `paperclip*` skills (installed by `paperclipai` CLI in `~/.cursor/skills/`) — `brain-dispatch` invokes `paperclip` skill in Paperclip mode, no copy/vendor needed.
- Source of truth for the brain-first workflow design: `tam-brain/meta/brain-first-workflows.md` and `tam-brain/decisions/2026-04-18-brain-skills-in-superpowers-fork.md`.

## [5.0.5] - 2026-03-17

### Fixed

- **Brainstorm server ESM fix**: Renamed `server.js` → `server.cjs` so the brainstorming server starts correctly on Node.js 22+ where the root `package.json` `"type": "module"` caused `require()` to fail. ([PR #784](https://github.com/obra/superpowers/pull/784) by @sarbojitrana, fixes [#774](https://github.com/obra/superpowers/issues/774), [#780](https://github.com/obra/superpowers/issues/780), [#783](https://github.com/obra/superpowers/issues/783))
- **Brainstorm owner-PID on Windows**: Skip `BRAINSTORM_OWNER_PID` lifecycle monitoring on Windows/MSYS2 where the PID namespace is invisible to Node.js. Prevents the server from self-terminating after 60 seconds. The 30-minute idle timeout remains as the safety net. ([#770](https://github.com/obra/superpowers/issues/770), docs from [PR #768](https://github.com/obra/superpowers/pull/768) by @lucasyhzhu-debug)
- **stop-server.sh reliability**: Verify the server process actually died before reporting success. Waits up to 2 seconds for graceful shutdown, escalates to `SIGKILL`, and reports failure if the process survives. ([#723](https://github.com/obra/superpowers/issues/723))

### Changed

- **Execution handoff**: Restore user choice between subagent-driven-development and executing-plans after plan writing. Subagent-driven is recommended but no longer mandatory. (Reverts `5e51c3e`)
