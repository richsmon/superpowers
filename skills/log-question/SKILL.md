---
name: log-question
description: Use when there is an open question that needs human input (architectural, product, process, business). Logs the question in the brain repo, pushes a formatted block to Slack #questions for the team to answer in thread, and updates the question file with the Slack permalink.
---

# Log Question

## Overview

Records an open question in `brain/questions/` and broadcasts it to Slack `#questions` for the team to answer in thread. One-way push only; answers come back into brain via the future bidirectional listen-back pipeline (out of scope here — see `workspaces/superpowers/features/log-skills-and-templates/spec.md` open follow-ups).

## When to Use

Use this skill when:
- You have a specific open question that needs human input.
- The question is too important to leave as an ad-hoc Slack message (needs an audit trail + link from decisions / ADRs).
- The question is one question, not a bundle — split bundles into separate calls.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root).
2. If not, check common sibling locations: `../brain`, `../../brain`.
3. If not found, ask the user for the path.

All file operations happen in the brain repo.

## Git Safety Protocol

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main`. If not, ask the user before continuing.
2. **Pull the latest** — `git -C <brain> pull --rebase origin main`.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Today's session file should already be untracked from `session-start`.

## Read Template

Read the template from the brain repo:

```
Read <brain>/templates/log/question.md
```

If missing, fail loudly:

> `templates/log/question.md not found in <brain-path>. Either restore from git or update brain.`

## Process

Ask the user:

1. **Title** — short, in the form of a question (e.g. "Should we use Postgres or DynamoDB for events?").
2. **Category** — `technical | product | process | business | unsure`.
3. **Question** — the full question body. Be specific; ground in the situation.
4. **Context** — why you're asking, what's blocked or unclear, what you have already considered or ruled out.

Slug generation (auto): lowercase the title, strip punctuation, split on whitespace, remove stopwords (a, an, the, is, are, was, were, do, does, did, will, would, should, could, can, may, might, must, shall, of, to, for, in, on, at, by, with, about, into, onto, from, as, and, or, but, if, then, than), take the first 5 remaining tokens, join with `-`. If collision in `questions/`, append `-2`, `-3`, etc.

Example: `"Should we use Postgres or DynamoDB for events?"` → `should-use-postgres-dynamodb-events`.

## Write the File

Write to `<brain>/questions/YYYY-MM-DD-<slug>.md` with:
- Date: today.
- Asked by: the user's github username (look up from `graph/entities/people/`).
- Category: from the user's answer.
- Status: `open`.
- Slack thread: leave empty for now (filled in the next step).
- Question and Context: from the user's answers.
- Answer: `_Pending._`.

## Push to Slack

After the file is written, call the Slack MCP:

```
mcp__claude_ai_Slack__slack_send_message(
  channel: "#questions",
  text: <formatted block below>
)
```

Formatted block (substitute the placeholders):

```
:question: *New question from <github-username>*
*Category:* <category>
*Status:* open

> <question body, first 500 chars; truncate with "…" if longer>

📄 brain: `questions/YYYY-MM-DD-<slug>.md`
Reply in thread to answer.
```

From the MCP response, extract the Slack permalink or `thread_ts`. Construct a permalink in the form `https://<workspace>.slack.com/archives/<channel_id>/p<ts_no_dot>` if the MCP returns only `thread_ts`.

Update the question file: replace the line `**Slack thread:** {URL}` with the actual permalink. Example: `**Slack thread:** https://marketclue.slack.com/archives/C0123ABCD/p1737800000000000`.

If the Slack push fails (MCP error, missing channel, missing permissions):
- The question file stays written.
- The `Slack thread:` line stays as `{URL}` (placeholder).
- Tell the user: "Slack push failed: <error>. The question is logged in `questions/YYYY-MM-DD-<slug>.md` but not yet broadcast. You can retry by re-invoking log-question with the same title, or post manually."
- Do NOT delete the question file.

## After Creation

- Add a reference to today's session diary under "## During the day":

```markdown
**[HH:MM] Question:** {short title}
→ questions/YYYY-MM-DD-<slug>.md (Slack: <permalink, or "push failed">)
```

- **Do NOT commit.** See "Commit Policy" below.

## Commit Policy

This skill does NOT commit. `session-end` bundles the day's question writes alongside decisions / blockers / session file into one commit on `main`.

## Red Flags

- Bundling multiple sub-questions into one question file. Split them — each gets its own broadcast and its own answer thread.
- Inventing a github-username for "Asked by". Use the actual git/Slack identity; look up canonical id from `graph/entities/people/`.
- Committing from this skill.
- Skipping the Slack push silently. The file is the source of truth; the Slack push is the broadcast. If push fails, report it — do not silently proceed.
- Pushing the question without context. A bare question is not actionable; the Context section is what makes the team able to answer.

## Notes

- Bidirectional listen-back (answer in Slack thread → updates question file `Answer` section + flips Status to `answered`) is **out of scope** for this skill. It will land in a future sub-project. For now, when an answer comes in via Slack, the asker manually transcribes it into the question file's `Answer` section and changes `Status: open` → `Status: answered`.
- The `/brain-log-question` slash command (if it exists) is a thin wrapper that loads this skill.
