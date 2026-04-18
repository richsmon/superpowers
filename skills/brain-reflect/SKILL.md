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
- Related: `brain-dispatch` (produces the work being reflected on), `log-decision`, `log-blocker`, `session-end`.
- Reflection should happen WHILE memory is fresh — at the latest within the same session. Tomorrow is too late.
