---
name: brain-pr-review
description: Use when the user wants an auto-review of a GitHub PR against the brain's ADRs, plans, and specs. Triggers - "/brain-pr-review {workspace} {pr-number}", "review PR N in workspace X", "audit this PR against the spec". Auto-detects feature from PR branch/title, bundles brain context, invokes code-reviewer agent, writes findings to workspaces/{ws}/reviews/pr-{N}.md.
---

# Brain — PR Review

Generates a PR review document in the brain by dispatching the `code-reviewer` agent against the PR diff plus relevant brain context (ADRs, plans, specs). Writes findings to `workspaces/{workspace}/reviews/pr-{N}.md`.

## Locate Brain Repo

1. Check if cwd is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `templates/feature/`).
2. If not, check siblings: `../brain`, `../../brain`.
3. If not found, ask the user for the brain path.
4. Verify the brain is on `main`: `git -C <brain> branch --show-current`. If not, ask the user before proceeding.

## Parse Args

Accept these forms:

- `/brain-pr-review {workspace} {pr-number}` — primary.
- `/brain-pr-review {pr-number}` — only when cwd is the brain AND a workspace is inferrable from an open feature. Otherwise ask.
- `/brain-pr-review {URL}` — parse `https://github.com/{org}/{repo}/pull/{N}` and reverse-map repo → workspace via `grep -l "url: https://github.com/{org}/{repo}" graph/entities/*.yaml`. If multiple matches → ask.

Required after parsing: `{workspace}`, `{pr-number}`.

## Resolve Repo from Workspace

```bash
yq '.repo.url' graph/entities/{workspace}.yaml
```

If the `.repo` block is missing → abort with:

```
Workspace '{workspace}' has no linked repo. Run `/brain-link-repo {workspace}` first.
```

Extract `{org}/{repo}` from the URL.

## Fetch PR Metadata

```bash
gh pr view {N} --repo {org}/{repo} --json \
  title,author,baseRefName,headRefName,additions,deletions,files,commits,statusCheckRollup,reviewDecision,mergeable,body
```

On 404 or auth error → abort with:

```
Cannot access PR #{N} in {org}/{repo}. Check `gh auth status`.
```

## Detect Feature

1. From `headRefName` — strip `feature/` prefix → candidate `{feature}`.
2. If no match, parse `title` — match `^feat\((.+?)\):` → candidate `{feature}`.
3. If still no match → workspace-level review (no feature path).
4. If a candidate exists → verify `workspaces/{workspace}/features/{feature}/` exists. If not → candidate invalid, fallback to workspace-level.
5. If multiple ambiguous candidates → ask the user which feature.

## Bundle Context Paths

Collect file paths to send to the `code-reviewer` agent:

- `workspaces/{workspace}/BRIEF.md`
- `workspaces/{workspace}/OVERVIEW.md`
- `workspaces/{workspace}/STATUS.md`
- `workspaces/{workspace}/decisions/*.md` (glob)
- If feature detected: `workspaces/{workspace}/features/{feature}/*.md` (glob)

Skip non-existent paths silently. Cap at 30 files total; if exceeded, prefer feature-folder files over workspace-level decisions.

## Invoke code-reviewer Agent

Dispatch the `rs-superpowers:code-reviewer` agent via the Agent tool with this prompt:

```
Review PR {org}/{repo}#{N} against the brain context.

PR diff:
<<< output of `gh pr diff {N} --repo {org}/{repo}` >>>

Brain context (read these files for spec, ADRs, plan, design):
<<< newline-separated list of absolute paths from the bundle >>>

For each finding:
1. Cite the source-of-truth statement (file path + section reference).
2. Cite the contradicting code (file:line in the PR).
3. State the consequence (1-3 sentences).
4. Formulate one open question for the PR author.

Return markdown with two parts:
- A summary table: | # | Finding | Severity (🔴/🟡/🟢 + category) |
- Detailed findings per row, formatted as:
  ### A) {severity} {one-line headline}
  **What the spec says:** {bullet list}
  **What the PR does:** {bullet list}
  **Consequence:** {prose}
  **Open question for {author}:** {prose}

Do NOT write to disk. Return the markdown as your response.
```

## Write Doc

Path: `workspaces/{workspace}/reviews/pr-{N}.md` (create `reviews/` if missing).

### Initial review (doc does not exist)

1. Copy `templates/review.md` to the target path.
2. Fill frontmatter:
   - `feature`: detected feature, or empty.
   - `repo`: `{repo}`.
   - `pr`: `https://github.com/{org}/{repo}/pull/{N}`.
   - `reviewer`: current git user from `git -C <brain> config user.name`.
   - `author`: PR author handle from gh metadata.
   - `status`: `review-in-progress`.
   - `created`: today (YYYY-MM-DD).
   - `lastUpdated`: today.
   - `template`: `templates/review.md`.
3. Fill the source PR header line with title, branch, target, stats.
4. Paste the agent's findings table into the `## Findings` section.
5. Paste the agent's detailed findings into the `### A) … ### B) …` blocks.
6. Leave `## Pre-merge blockers`, `## Follow-up PR scope`, `## Open questions for {author}`, and `## Recommended next step` as template placeholders for the user to fill manually.
7. Print summary: total findings, severity breakdown, doc path.

### Re-review (doc exists)

1. Read existing doc.
2. Extract current findings table as the "original findings" list.
3. Send the agent the original findings + the latest PR diff. Ask it to:
   - Mark each original finding as `⏳` (still open), `✅` (fixed in PR), or `❌` (regressed / new evidence against it).
   - Surface any new findings since the last review.
4. Append a new section to the existing doc:

```markdown
## Re-review {YYYY-MM-DD}

### Status of original findings
- A: {⏳/✅/❌} {short note}
- B: {…}
- C: {…}

### New findings
- F: 🟡 {headline}
  {detail}
```

5. Update frontmatter: `lastUpdated` → today, `status` → `re-review-pending`.
6. Print summary: status changes, new findings count, doc path.

## No Commit

The skill does NOT commit. The user's `session-end` will bundle the new doc into the day's main commit per the brain commit policy.

## Failure Modes

| Situation | Action |
|---|---|
| Workspace dir missing | Abort with: "Workspace '{w}' not found. Try `/brain-status` to list active workspaces." |
| Entity YAML missing `repo:` block | Abort with: "Workspace '{w}' has no linked repo. Run `/brain-link-repo {w}` first." |
| `gh pr view` 404 or no auth | Abort with: "Cannot access PR #{N} in {org}/{repo}. Check `gh auth status`." |
| Ambiguous feature | Ask user which feature. |
| code-reviewer agent times out or errors | Write the doc with an empty findings table + warning note: "code-reviewer agent failed to return findings. Fill in manually." |

## Related skills

- `brain-link-repo` — wire a brain workspace to a code repo.
- `brain-status` — list active workspaces.
- `code-reviewer` (superpowers agent) — the underlying review engine.

The skill is the brain-facing surface; the actual review work happens in the agent.
