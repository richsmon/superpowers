---
name: brain-sync-design-to-figma
description: Use when the user wants to sync a Claude Design HTML artifact (tokens + components + screens) from the brain into the workspace's Figma file. Says "sync to figma", "push design system to figma", "sync feature screens to figma", "run figma sync", or names the pipeline ("brain-sync-design-to-figma", "design-to-figma"). One-way brain → Figma, additive, pipeline-shaped — never the reverse direction.
---

# Brain — Sync Design to Figma

## Overview

Runs the one-way, additive **Claude Design → Figma** pipeline for a workspace. The canonical procedure lives in the brain at `meta/claude-design-to-figma-workflow.md` — **that document is the source of truth**; this skill is the agent-actionable wrapper that loads it, enforces the policy decisions documented across this PR, and walks the operator through invocation.

The pipeline has **two sibling sub-flows**:

- **`sync-design-system <repo>`** — pushes tokens + components + text/effect styles from `<FeatureName>.html` into the workspace's Design System Figma file.
- **`sync-feature-screens <repo> <feature>`** — pushes feature screens, using DS-library imports for components. Requires DS to already be synced + published.

Direction: brain → Figma only. Reverse sync is explicitly out of scope.

## When to Use

- The user wants to land a Claude-generated design artifact (under `workspaces/{repo}/design/{feature}/<FeatureName>.html`) into the workspace's Figma file
- A new feature's design needs token + component additions to the workspace's DS, followed by feature screens
- A re-run after the operator edited the source HTML (additive — Figma stays the canvas, brain is canonical for Claude-generated work)

**Don't use** for:

- Figma → brain reverse sync (out of scope per ADR)
- Editing existing Figma artifacts that weren't written by this pipeline (Figma stays the canvas for hand-crafted polish)
- The daily session diary (use `session-end` instead)

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/` + `meta/claude-design-to-figma-workflow.md`)
2. If not, check siblings: `../brain`, `../../brain`
3. If not found, ask the user
4. **Sync state** — `git -C <brain> pull --rebase origin main` so the workflow doc and the workspace's `.figma.yaml` reflect the real tip. Verify `git -C <brain> branch --show-current` is `main` before the next section moves you off it.

## Git Safety Protocol

Pipeline syncs land on a **dedicated feature branch + PR**, NOT on `main`. Run all checks before creating that branch:

1. **Start on `main`** — `git -C <brain> branch --show-current` must return `main`. If you find yourself on a stale sync branch from a previous run, finish it (or abort it) before starting a new one.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` (already done in Locate Brain Repo; re-check that no commits arrived since).
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Expected: at most the day's untracked session file. Unexpected: stale `.sync-state/` artifacts from a half-run pipeline, or modified `.figma.yaml` — surface these to the operator before proceeding. They probably belong on a prior sync branch.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` must be empty.

## Commit Policy

**Pipeline syncs go on a feature branch + PR. Never on `main`.** This is the key policy difference from `session-end` (which commits daily diary to main) and `brain-checkpoint` (which commits non-diary state to main).

- **Branch name:** `brain/figma-<repo>-<feature-or-ds>-sync-<YYYY-MM-DD>` (e.g. `brain/figma-app-design-system-sync-2026-05-21`, `brain/figma-app-onboarding-sync-2026-05-21`).
- **Per-substep commits** are required (per the brain workflow doc) and all land on the feature branch. Format: `figma-sync: ds 4a vars +<N>`, `figma-sync: <feature> 4d screen <Name>`, `brain-meta: <feature> inventory <date> for <repo>`, etc. — exact commit-message prefixes documented in the brain workflow doc per phase.
- **PR target:** `main`. One PR per sync cycle (Phase 0 through Phase 5). The session file is **never** part of this PR — it stays untracked on `main` and lands via `session-end`.
- **No commits to main from this skill, ever.** If `git status` on `main` shows pipeline artifacts (`.sync-state/`, modified `.figma.yaml`), STOP — the operator already started this off-policy. Move the work onto a branch before doing anything else.

## Signed Commits Required

**Every per-substep commit MUST be signed. Without a signature, do NOT commit.**

Before staging anything, verify signing is configured:

```bash
git -C <brain> config --get commit.gpgsign      # must print: true
git -C <brain> config --get user.signingkey     # must print a key id (not empty)
git -C <brain> config --get gpg.format          # 'openpgp' (default) or 'ssh'
```

If any of those return empty / not `true`, **STOP**. Do not commit. Tell the operator signing is not configured and exit. Never bypass with `--no-gpg-sign`, `-c commit.gpgsign=false`, or `commit --no-verify`.

After each substep commit, verify with `git -C <brain> log -1 --show-signature` — must show a "Good signature" line before the next substep proceeds. Pipeline runs can produce many commits; one unsigned commit pollutes the whole PR.

## Process

### Step 1 — Load the canonical procedure

Open `meta/claude-design-to-figma-workflow.md` in the brain repo. Read it end-to-end before invoking any `mcp__plugin_figma_figma__*` tool. The phases, flags, anchor contracts, and failure modes documented there are mandatory — this skill does not re-derive them.

### Step 2 — Select the sub-flow

Ask the operator (or infer from the invocation):

- **`sync-design-system <repo>`** — DS flow. Inputs: `<repo>` (e.g. `app`). Requires `design/.figma.yaml` with `design_system.file_key` set, and at least one root-level `design/<feature>/<FeatureName>.html` to source tokens + components from.
- **`sync-feature-screens <repo> <feature>`** — feature flow. Inputs: `<repo>` `<feature>`. Requires `design/.figma.yaml` with `features.<feature>.file_key` set AND `design_system.library_key` set (DS file has been published as library).

If the feature flow is requested but the DS library is not yet published, abort with: *"Run `sync-design-system <repo>` and publish the library first."* (per Phase 0 step 4 of the workflow doc).

### Step 3 — Confirm flags

The brain workflow doc lists the shared flags. Confirm with the operator before starting:

- `--refresh-inventory` — force full Figma re-read even if the per-file lock is still valid (24h TTL)
- `--overwrite-tokens` (DS only) — overwrite Figma token values on conflict (default: skip + log)
- `--rebuild-components` (DS only) — regenerate component internals when same-name exists
- `--replace-screens` (feature only) — replace existing frames instead of appending ` — v{N} (from brain)`
- `--no-commit` — local trial runs only; never use on a real sync run (breaks the per-substep commit audit trail)
- `--dry-run` — execute Phases 0–3 only; print where Phase 4 would write

### Step 4 — Create the sync branch

Once the sub-flow + flags are confirmed and signing is verified:

```bash
cd <brain>
git checkout -b "brain/figma-<repo>-<feature-or-ds>-sync-$(date +%F)"
```

All subsequent commits land here. Do not return to `main` until Phase 5 completes and the operator opens the PR.

### Step 5 — Execute phases per the brain workflow doc

Run Phases 0 → 5 as documented in `meta/claude-design-to-figma-workflow.md`. Key reminders the workflow doc enforces:

- **Phase 0** aborts on missing anchors (`<style id="tokens-css">`, `data-screen-label`, `data-theme`). Surface the missing anchor and the source path to the operator.
- **Phase 1** is read-only; the inventory + lock pair lives under `design/.sync-state/`.
- **Phase 2 is the most failure-prone.** Mechanical transcription discipline applies — read the actual JSX/CSS, capture exact values, exact token bindings, exact copy, exact variants. **No decorative inventions. No "fits-the-style-better" replacements.** When in doubt, read more JSX.
- **Phase 3 ends in HUMAN GATE.** Stop. Commit the diff. Wait for operator approval. If `--dry-run`, end here.
- **Phase 4** runs per substep. Before each `use_figma` write, **re-fetch metadata** for the target node and abort if the inventory hash drifted (race-condition guard). Commit after each substep. Verify signature after each commit.
- **Phase 5** verifies by re-running Phase 1 + Phase 3 against the just-written file. "To add" sections must be empty (modulo accepted conflicts). For DS flow, the operator publishes the library in Figma UI and captures `library_key` back into `.figma.yaml`.

### Step 6 — Open the PR

After Phase 5 succeeds:

```bash
git -C <brain> push -u origin "brain/figma-<repo>-<feature-or-ds>-sync-<date>"
gh -R <brain-remote> pr create --base main \
  --title "figma-sync(<repo>): <ds or feature-name> — <YYYY-MM-DD>" \
  --body "<summary referencing the Phase 5 audit at design/.sync-state/history/<date>-<file-slug>-sync.md>"
```

After the PR opens, **return to `main`**: `git -C <brain> checkout main`. The session file stays untracked on main and lands later via `session-end`.

## Failure Modes

The brain workflow doc enumerates pipeline-specific failure modes (anchor missing, Figma rate limit, inventory lock TTL, race condition, DS library not published, component not in DS library, Phase 2 transcription drift). Follow those exactly.

This skill adds two policy-level failure modes:

- **Operator started pipeline work on `main`.** Pipeline artifacts (`.sync-state/`, modified `.figma.yaml`) appear in `git status` on main. STOP. Move the work onto a `brain/figma-...` branch first (`git stash`, create branch, `git stash pop`, stage, signed commit, continue).
- **Signing breaks mid-pipeline.** Phase 4 produces many commits. If signing config gets removed or the agent's key becomes unreachable mid-run, the next substep commit fails the signature check. STOP. Tell the operator. Do not produce unsigned commits even to "finish the run" — re-establish signing and resume.

## Red Flags

- **Committing pipeline artifacts to `main`.** This skill never commits to main. Branch + PR only.
- **Bundling the daily session file into the sync PR.** The session file is `session-end`'s; it must not appear in this PR's diff.
- **Producing unsigned per-substep commits.** No signature, no commit — full stop. Never use `--no-gpg-sign` or `-c commit.gpgsign=false` to "finish the run".
- **Skipping Phase 3 HUMAN GATE.** Phase 4 may not begin without explicit operator approval on the diff.
- **Phase 2 shortcuts.** Using an Explore-agent summary instead of reading the actual JSX is the documented historical failure mode of this pipeline (2026-05-20 first execution). Mechanical transcription only.
- **Inventing tokens, copy, or variants** not present in the source HTML. Source has 5 circles → Figma has 5 circles.
- **Substituting tokens** (`primary-600` instead of the source's `mc-indigo`). Missing tokens are **added** during DS sync, not substituted.
- **`git add -A` mid-run.** Stage only the substep's intended paths. The session file lives in the same brain repo and must stay out of every pipeline commit.
- **`--no-commit` on a real run.** That flag is for local trials only; using it on a real sync destroys the audit trail.
- **Skipping the inventory re-fetch before a Phase 4 write.** Race-condition guard exists for a reason — without it, a concurrent Figma edit gets silently overwritten.
- **Returning to `main` before opening the PR.** Stay on the sync branch until the PR is up, otherwise the operator's mental model of "where is my work" breaks.

## Notes

- **Canonical procedure lives in the brain**, not here. If this skill and `meta/claude-design-to-figma-workflow.md` ever drift, the brain doc wins. Update this skill to match — never silently override.
- **The workflow expects 2–3 features of real-world exercise** before this skill stabilizes. The first exercise was the dashboard sync on 2026-05-20 (PR #12 / #13 in the brain). Phases, flags, or failure modes may evolve as more features run through.
- **ADRs that constrain this skill:**
  - `decisions/2026-05-20-claude-design-to-figma-pipeline.md` — direction (brain → Figma only), additive defaults
  - `decisions/2026-05-20-figma-multi-file-architecture.md` — multi-file split (DS file + per-feature files, library_key linkage)
  - `decisions/2026-05-21-design-source-at-root.md` — HTML source path (`design/<feature>/<FeatureName>.html` at root)
- **Related skills:** `figma:figma-use` (loaded by Phase 0 step 7 whenever a write is planned), `session-end` (where the operator's daily diary commits — never inside this pipeline), `brain-checkpoint` (mid-day non-pipeline checkpoint; explicitly excludes the artifacts this skill writes).
- **The `/brain-sync-design-to-figma` slash command is a thin wrapper that loads this skill — they are not duplicates. The command is the entry point; this file is the spec.**
