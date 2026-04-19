---
name: brain-reflect
description: Use when the user wants to capture learnings, decisions, or follow-ups from a completed work session into the brain, says "reflect", "capture insights", "turn this into atomic notes", "what did we learn", or after a dispatched agent returns and there are takeaways worth persisting before they evaporate.
---

# Brain — Reflect

Captures learnings, decisions, and follow-ups from completed work into the brain in the right shape: atomic notes for reusable patterns, decisions for choices, entity updates for state changes, and follow-up TODOs in today's session.

This skill exists because the most expensive failure mode of brain-first workflow is doing the work and then NOT writing down what was learned — knowledge evaporates between sessions.

## Locate Brain Repo

1. Check if current workspace is the brain (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` + `graph/entities/`)
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- Right after a `brain-dispatch` returns
- End of a focused work block (before context-switching)
- When the user says "let's capture this", "reflect on what we did", "what should we remember"
- When a Paperclip task closes and there are insights worth keeping

**Don't use** for:

- Routine session wrap-up — use `session-end` (Superpowers) for that
- A single decision in isolation — use `log-decision` directly

## Required inputs

The skill reads context from:

1. **Today's session note** — `sessions/YYYY/MM/YYYY-MM-DD-{user}.md`
2. **The most recent "Dispatched: ..." entry** in that session, if present
3. **Free-text takeaways** — ask the user: "What's worth remembering from this work?"

## Task-outcome mode (auto-update tracker)

When invoked with a task ID and outcome — explicit `--task T-NN --outcome <state>`, or implicit after a `brain-dispatch` returned with task-mode context — perform tracker updates BEFORE the normal triage flow.

### Trigger detection

- Explicit: user says "reflect T-NN done", "reflect T-NN blocked", "reflect T-NN in-progress"
- Optional flag: `--task T-NN --outcome <state>` where state is one of `done` | `in-progress` | `blocked`
- Implicit: previous `brain-dispatch` invocation surfaced a task ID and the subagent has just returned

### Auto-tracker updates

For task `T-NN` in workspace `W` with outcome `O`:

1. **`workspaces/W/tasks.md` row update:**
   - `Status` column → `O`
   - `Updated` column → today's date
   - `Verified` column → today's date (ONLY if `O = done`)
   - `Notes` column → append a short note from the agent summary (truncate to ~80 chars; longer learnings go to atomic notes via the standard triage flow)

2. **`workspaces/W/STATUS.md` updates:**
   - `**Updated:**` line → today's date and time
   - If `O = done`: remove T-NN row from "In flight" table; add to "Recently shipped" with format `- YYYY-MM-DD — {title} (commit {hash} or PR link)`
   - If `O = in-progress`: ensure T-NN row exists in "In flight" with current `Owner` and `Started` date; refresh `Notes` cell
   - If `O = blocked`: remove from "In flight"; add to "Blocked / waiting" with explicit blocker reason

3. **`workspaces/W/tasks.md` "Dispatch history" append:**
   - Compute next dispatch number by counting existing entries in the section
   - Format: `- YYYY-MM-DD dispatch #N — T-NN <one-line subject> — <O> — see sessions/YYYY-MM-DD-*.md` (commit/PR link if available)

4. **`graph/entities/W.yaml` bump:** `lastUpdated:` → today's date. If `O = done`, optionally append a one-line milestone note to `content:` (do not silently rewrite history; append).

5. **THEN run the normal triage flow** (Step 2 onward of the standard process) for any additional learnings, decisions, atomic notes, follow-ups the user surfaces beyond the tracker mechanics.

### Refusing auto-mode

Abort and ask the user to clarify if:

- T-NN does not exist in `workspaces/W/tasks.md` (typo or wrong workspace)
- T-NN's status was already `O` (avoid double-counting in dispatch history)
- `O = done` but the agent summary did not include baseline test results (per `MEMORY.md` 2026-04-19 dispatch-slice rule, claiming `done` requires baseline verification)

### What NOT to do in auto-mode

- Never auto-commit. Tracker updates are git changes; the user (or `session-end`) decides when to commit.
- Never auto-progress to the next task. Each loop tick is a deliberate human decision.
- Never close a phase even if every scoped task is `done`. Phase closure requires verifying success criteria, which are behavioral checkboxes that need human judgment.

## Process

### 1. Identify the work being reflected on

- Default: the most recent `### Dispatched:` entry in today's session
- Otherwise: ask the user "what work are we reflecting on?" and have them name a feature, dispatch, or describe in prose
- Read that work's spec/plan/agent summary so the reflection is grounded, not abstract

### 2. Triage the takeaways

For each takeaway the user mentions, route it to the right destination using this table:

| Takeaway shape                              | Destination                                                  | Skill / file                                   |
|---------------------------------------------|--------------------------------------------------------------|------------------------------------------------|
| Reusable pattern / technique / "how-to"     | Atomic note                                                  | `workspaces/{w}/knowledge/{slug}.md`           |
| Choice between alternatives, with rationale | Decision                                                     | `log-decision` (Superpowers) → workspace decisions/ or top-level decisions/ |
| New fact / state change about an entity     | Entity update                                                | edit `graph/entities/{id}.yaml`                |
| Follow-up TODO that isn't done yet          | Session "Next" section + create issue / paperclip task       | edit today's session, optionally `paperclip`   |
| Cross-project insight (applies broadly)     | Top-level decision OR cross-project entity                   | `decisions/` or `graph/entities/`              |
| Blocker that stopped the work               | Blocker note                                                 | `log-blocker` (Superpowers)                    |

If the user gives a vague takeaway, ask one clarifying question to figure out the shape. Don't guess.

### 3. Write each captured item

For atomic notes, use the template in the brain (`templates/atomic-note.md` if present). Each note must include:

- Frontmatter: `id`, `type: knowledge`, `tags`, `related` (cross-links to entities), `created`, `lastUpdated`
- A single, atomic idea (one note = one concept)
- A "Why this matters" line connecting it back to the work that produced it

For entity updates, bump `lastUpdated:` and append to the `content:` block; do not silently rewrite history.

For decisions, delegate to `log-decision` skill (do NOT write the decision file directly here — that skill knows the template + commit policy).

### 4. Update today's session note

Append a "Reflection" entry under "During the day":

```markdown
**[HH:MM] Reflection:** {one-line summary of what was reflected on}
- Atomic notes: {list of paths}
- Decisions: {list of paths}
- Entity updates: {list of entity ids}
- Follow-ups: {list of TODO lines}
→ workspaces/{w}/...
```

If there are follow-up TODOs, also add them to the "Next" section of the session note.

### 5. Confirm and report

```
Reflection complete for {work}.
- {N} atomic notes created
- {N} decisions logged
- {N} entities updated
- {N} follow-ups added to today's "Next" section

Brain is up to date.
```

## Red Flags

- Writing a wall-of-text "session log" entry instead of atomic, routed pieces
- Inventing takeaways the user didn't actually say (only capture what was said / observed)
- Skipping cross-links (`related:`) — atomic notes without links rot fast
- Reflecting too late ("I'll remember to do this next time") — the cost of capture is small NOW, infinite LATER
- Creating "knowledge notes" that are actually decisions in disguise (route to `log-decision` instead)

## Notes

- This is a routing skill: it doesn't replace `log-decision`, `log-blocker`, or `session-end` — it orchestrates them.
- Related: `brain-next` (recommends the task that produced the work), `brain-dispatch` (produces the work being reflected on; use task-mode for auto-context), `log-decision`, `log-blocker`, `session-end`.
- Per-task loop: `brain-next` → `brain-dispatch --task T-NN` → [read output] → `brain-reflect --task T-NN --outcome <state>` → `brain-next` again.
- Reflection should happen WHILE memory is fresh — at the latest within the same session. Tomorrow is too late.
