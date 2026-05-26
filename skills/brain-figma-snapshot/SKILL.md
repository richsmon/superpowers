---
name: brain-figma-snapshot
description: Use when the user wants to capture a Figma file's current state into the brain as a JSON+PNG snapshot. Says "snapshot figma", "pull figma to brain", "capture figma state", "refresh snapshot", "brain-figma-snapshot <feature>". One-way Figma → brain via REST API, read-only on Figma, additive in brain. Counterpart to brain-sync-design-to-figma (forward direction).
---

# Brain — Figma Snapshot (reverse capture)

## Overview

Captures a Figma file's current state into the brain as a JSON snapshot (REST API) plus optional page-level PNG renders. The canonical procedure lives in the brain at `meta/figma-to-brain-snapshot-workflow.md` — **that document is the source of truth**; this skill is the agent-actionable wrapper that loads it, enforces the policy decisions (PAT location, direct-to-main commit, signed commits), and walks the operator through invocation.

Direction: **Figma → brain only.** Read-only on Figma. No `use_figma` writes, no Figma-side mutation. Counterpart to `brain-sync-design-to-figma` which runs the opposite direction (brain → Figma) with the opposite commit policy (branch + PR).

## When to Use

- After a UX polish session in Figma, to capture what was changed before it drifts further from the brain's structural metadata
- Weekly cadence per workspace, or before every forward-pipeline re-run (so the inventory matches the live file)
- After spotting drift between `design/.sync-state/figma-ids.<feature>.json` and the actual Figma file
- When the operator types `/brain-figma-snapshot <feature-name>` or asks to "pull figma to brain", "snapshot figma", "refresh the figma snapshot"

**Don't use** for:

- Forward sync (use `brain-sync-design-to-figma` — opposite direction)
- `.fig` binary export (manual UX action in Figma UI: File → Save local copy)
- The daily session diary (use `session-end`)
- Editing the source HTML to match what UX did in Figma (out of scope per workflow doc — snapshot is read-only on the source HTML)

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/` + `meta/figma-to-brain-snapshot-workflow.md`)
2. If not, check siblings: `../brain`, `../../brain`
3. If not found, ask the user
4. **Sync state** — `git -C <brain> pull --rebase origin main` so the workflow doc and the workspace's `.figma.yaml` reflect the real tip. Verify `git -C <brain> branch --show-current` is `main` before proceeding.

## Git Safety Protocol

Snapshots commit **direct to main** (see Commit Policy below). Run all checks before fetching:

1. **Start on `main`** — `git -C <brain> branch --show-current` must return `main`. If on a stale branch, finish/abort it first.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main` (already done above; re-check no commits arrived since).
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Expected: at most the day's untracked session file. Unexpected: stale `.sync-state/snapshots/` artifacts from a half-run snapshot, or modified `.figma.yaml` — surface to operator before proceeding.
4. **Check branch state** — `git -C <brain> log --oneline origin/main..HEAD` must be empty.

## Commit Policy — DIRECT TO MAIN

**Snapshots commit straight to `main`. No feature branch. No PR.** This is the deliberate inverse of `brain-sync-design-to-figma` (which mandates branch + PR + never main).

Rationale, codified in workflow doc Phase 3:

- Snapshots are **append-only metadata**, not pipeline writes. New files in `design/.sync-state/snapshots/`, optional inventory refresh, optional `.figma.yaml` updates if UX renamed pages/sets.
- Low risk: read-only on Figma, additive in brain, no source-code changes, no review value.
- High volume: weekly cadence × N features = a PR per snapshot would drown the review queue.

**Commit shape** (one commit per substep, all on `main`):

- `brain-meta: figma-snapshot <feature> <YYYY-MM-DD>` — JSON + `.meta` sidecar
- `brain-meta: figma-snapshot PNGs <feature> <YYYY-MM-DD>` — PNG renders (if `--with-pngs`)
- `brain-meta: figma-snapshot inventory <feature> <YYYY-MM-DD>` — derived metadata refresh (Phase 2)
- `brain-meta: figma-snapshot diff <feature> <YYYY-MM-DD>` — visual+structural diff vs previous (Phase 4, if anything meaningful changed)

Push after each commit OR batch-push at end of run — pick one strategy and stay consistent within the run. Do not push partial runs that leave the snapshot in a half-written state.

**Policy contrast — do not confuse:**

| Skill | Direction | Branch | Commit | PR |
|---|---|---|---|---|
| `brain-sync-design-to-figma` | brain → Figma | `brain/figma-<repo>-<…>-sync-<date>` | per-substep, signed | **required** to merge to main |
| `brain-figma-snapshot` (this) | Figma → brain | `main` directly | per-substep, signed | **never** |

## Signed Commits Required

**Every per-substep commit MUST be signed.** Without a signature, do NOT commit.

Before staging:

```bash
git -C <brain> config --get commit.gpgsign      # must print: true
git -C <brain> config --get user.signingkey     # must print a key id (not empty)
git -C <brain> config --get gpg.format          # 'openpgp' (default) or 'ssh'
```

If any return empty / not `true`, **STOP**. Do not commit. Tell the operator signing is not configured. Never bypass with `--no-gpg-sign`, `-c commit.gpgsign=false`, or `commit --no-verify`.

After each commit, verify with `git -C <brain> log -1 --show-signature` — must show "Good signature" before the next substep proceeds.

## PAT Loading

`FIGMA_PAT` is read from `<brain>/.env` (gitignored — `.env` and `.env.*` are in the brain's `.gitignore`). Pattern:

```bash
cd <brain>
set -a; source .env; set +a
```

Verify before the first REST call:

```bash
curl -sI -H "X-Figma-Token: $FIGMA_PAT" \
  "https://api.figma.com/v1/files/<file_key>" | head -1
# Expect: HTTP/2 200
```

If `FIGMA_PAT` is unset or empty:

```
abort with:
  FIGMA_PAT not loaded. Run from <brain>:
    echo 'FIGMA_PAT="figd_…"' >> .env
    set -a; source .env; set +a
  Confirm .env is gitignored (already in place per PR #16, 2026-05-22).
  Generate token at Figma → Settings → Account → Personal access tokens.
  Minimum scope: File content (read). Add File metadata (read) for image renders.
```

If `curl` returns 403 → PAT expired/invalid; rotate in Figma settings.
If 404 → file deleted in Figma; abort and surface to operator (brain has last snapshot but cannot refresh).
If 429 → rate limit (rare; 6000 req/min/token); backoff and retry per workflow doc.

## Process

### Step 1 — Load the canonical procedure

Open `meta/figma-to-brain-snapshot-workflow.md` in the brain repo. Read it end-to-end before fetching. The phases, flags, file naming convention, and failure modes documented there are mandatory — this skill does not re-derive them.

### Step 2 — Resolve the target

1. Read `design/.figma.yaml`. Resolve `features.<feature>.file_key`. Abort if `NOT-YET-CREATED`.
2. Confirm `design/.sync-state/snapshots/` exists; create if missing.

### Step 3 — Confirm flags

Workflow doc flags:

- `--with-pngs` — also fetch page-level PNG renders (one per page). **Default: on.** PNG diff capability is cheap and high-leverage.
- `--with-fig-note` — bump the snapshots README to note that UX should drop an updated `.fig` export.

Confirm with the operator before starting.

### Step 4 — Execute Phases 0 → 4 from the workflow doc

Run the phases as documented in `meta/figma-to-brain-snapshot-workflow.md`:

- **Phase 0** — env + path setup, file_key resolution.
- **Phase 1** — `GET /v1/files/{file_key}` → save raw JSON pretty-printed to `design/.sync-state/snapshots/{YYYY-MM-DD}-{feature}.json` + `.meta` sidecar (Last-Modified, content_hash, fetched_at, source endpoint, size, structural counts). If `--with-pngs`: enumerate `document.children[].id`, `GET /v1/images/{file_key}?ids={page_id}&format=png&scale=1` per page → download → save to `…-{feature}-png/{page-slug}.png`.
- **Phase 2** — refresh derived metadata: `figma-ids.<feature>.json`, `figma-inventory.<feature>.md` (append capture-history row), `figma-inventory.<feature>.lock` (new `captured_at` + `content_hash`, append to `history[]`), reconcile `.figma.yaml` `page_ids` / `component_sets` if UX renamed.
- **Phase 3** — commit direct to main per Commit Policy above. One signed commit per substep. Verify each signature.
- **Phase 4** (optional) — structural diff (JSON node IDs added/removed/changed) + visual diff (PNG comparison) vs previous snapshot for the same feature; write `design/.sync-state/history/{YYYY-MM-DD}-{feature}-snapshot-diff.md` if anything meaningful changed.

### Step 5 — Push

After all Phase 3 commits land and signatures verify:

```bash
git -C <brain> push origin main
```

No PR. No branch. The session file stays untracked on `main` and lands later via `session-end`.

## Failure Modes

The brain workflow doc enumerates snapshot-specific failure modes (PAT unset/expired, file deleted, rate limit, large file >10 MB). Follow those exactly.

This skill adds policy-level failure modes:

- **Operator on a non-main branch.** Snapshots are main-only. STOP. Switch to main (or finish the other branch) before snapshotting. Do not create a `brain/figma-snapshot-*` branch — that was a workflow doc fallback for the case of large `.figma.yaml` mutations, but the default and the policy this skill enforces is direct-to-main.
- **Signing config broken.** Snapshot runs produce 2–4 commits. If signing breaks mid-run, the next substep commit fails the check. STOP. Re-establish signing before resuming. Never produce unsigned commits to "finish the run".
- **Mixing snapshot commits with forward-sync work.** Different artifacts, different policy. If a forward-sync branch is in flight, finish it (or set it aside) before snapshotting. Don't co-mingle snapshot commits onto a `brain/figma-…-sync-…` branch.

## Red Flags

- **Creating a `brain/figma-snapshot-*` branch.** Wrong skill. Snapshots commit direct to main. Branching is the forward pipeline's policy.
- **Bundling snapshot commits into a forward-sync PR.** Different policy, different artifacts, different review needs. Keep them separate.
- **Skipping the `.meta` sidecar.** Loses Last-Modified + content_hash; future drift detection breaks. The `.meta` file is not optional.
- **`--no-gpg-sign` or `commit --no-verify`** to "finish the run". Never. Same rule as every other brain commit.
- **Calling `use_figma` or any `mcp__plugin_figma_figma__*` write tool from this skill.** Wrong direction. Snapshot is read-only on Figma. The only Figma calls are REST `GET /v1/files/…` and `GET /v1/images/…`.
- **Trying to update the source HTML to match the snapshot.** Out of scope per workflow doc. The snapshot is structural truth; if you want HTML to match, you rewrite the HTML by hand or rebuild from scratch — not as part of this skill.
- **`git add -A` mid-run.** Stage only the snapshot's intended paths (`design/.sync-state/snapshots/<date>-<feature>.*`, optional `-png/`, optional inventory updates). The session file lives in the same brain repo and must stay out of every snapshot commit.
- **Skipping Phase 2 inventory refresh.** The snapshot is the raw truth; derived metadata that doesn't follow becomes stale by the next forward sync.
- **Pushing a partial run.** If you've committed JSON but not PNGs, either finish the PNG substep + commit + push together, or hold the push until the run is complete. Don't leave `main` in a half-snapshotted state.

## Notes

- **Canonical procedure lives in the brain**, not here. If this skill and `meta/figma-to-brain-snapshot-workflow.md` ever drift, the brain doc wins. Update this skill to match — never silently override.
- **`<brain>/.env` is the PAT location** (codified after PR #16, 2026-05-22). Workflow doc says "or any gitignored location" — this skill pins `<brain>/.env` as the default. `.gitignore` already covers `.env` + `.env.*`.
- **ADR alignment:** `decisions/2026-05-20-claude-design-to-figma-pipeline.md` declared brain → Figma one-way. This skill is the **read-only counterpart**, not a reverse pipeline. It complements (does not violate) the directional contract — the forward pipeline still owns canonical authorship.
- **Related skills:** `brain-sync-design-to-figma` (forward, opposite direction, opposite commit policy — read both before mixing); `session-end` (where the daily diary commits — never inside this snapshot run); `brain-checkpoint` (mid-day non-pipeline checkpoint; explicitly excludes the artifacts this skill writes).
- **The `/brain-figma-snapshot` slash command is a thin wrapper that loads this skill — they are not duplicates.** The command is the entry point; this file is the spec.
