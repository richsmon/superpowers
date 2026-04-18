---
name: brain-route
description: Use when the user is unsure whether a piece of work belongs in the brain (knowledge / decision / spec) or in a code repo (implementation / config / test), says "where does this go", "is this a brain thing", "should I write this down", "brain or code", or wants triage on how to split a chunk of thinking-and-doing between brain and code.
---

# Brain — Route

Helps the user (and the agent) decide where a piece of work, thought, or artifact belongs: in the brain (workspaces, decisions, knowledge, entities) or in a code repository (implementation, configs, tests, docs).

This skill exists because the most common failure of brain-first workflow is putting the wrong thing in the wrong place — implementation details in the brain (rot), or design decisions in the code (lost in commit history).

## Locate Brain Repo

1. Check if current workspace is the brain
2. If not, check siblings: `../brain`, `../../brain`, `../tam-brain`
3. If not found, ask the user

## When to Use

- The user is mid-thought and asks "where does this go?"
- An agent finishes work and isn't sure what to write back to the brain vs leave in the code
- Before starting a new piece of work, to decide its split
- When something feels like it belongs in both places (often it does — both, but in different shapes)

**Don't use** for:

- Obvious cases (a code function clearly goes in the code repo, a session log clearly goes in the brain)
- Replacing judgement entirely — this skill gives a routing rubric, not the final answer

## Routing rubric

Ask the user (or yourself) these questions in order. The first "yes" determines primary destination.

| Question                                                                | If yes →                                                                           |
|-------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Is this a **choice** between alternatives with rationale (decision)?    | **Brain**: `decisions/` (top-level if cross-project, else workspace decisions/)    |
| Is this a **reusable pattern / insight / how-to** (knowledge)?          | **Brain**: `workspaces/{w}/knowledge/` atomic note                                 |
| Is this a **state change** about an entity (project, person, concept)?  | **Brain**: edit `graph/entities/{id}.yaml`                                         |
| Is this a **WHAT/WHY** (spec) for future work?                          | **Brain**: `workspaces/{w}/features/{f}/spec.md`                                   |
| Is this a **HOW** for a specific feature (plan)?                        | **Brain**: `workspaces/{w}/features/{f}/plan.md`                                   |
| Is this **executable code, config, test, build script, native asset**?  | **Code repo** of the workspace (use `brain-dispatch` to actually do the work)      |
| Is this **API docs / runtime contract / OpenAPI / schema**?             | **Code repo** (source of truth = code), but link from brain spec                   |
| Is this a **session log / blocker / today's work**?                     | **Brain**: today's session note (`session-start` / `log-blocker`)                  |
| Is this **vendor / third-party documentation**?                         | **Neither** — link to source instead; only mirror in brain if you've added value   |
| Is it **secrets / API keys / credentials**?                             | **Neither (in plain text)** — keychain / env / SecureStore on device               |

If multiple "yes" answers fire, the work is mixed. Split it:

- The decision/spec/knowledge piece → brain
- The code/config/test piece → code repo
- Cross-link them (brain spec links to commit; commit message references brain decision id)

## Process

### 1. Get the artifact / thought from the user

Ask: "What is the piece of work? Paste it, summarize it, or tell me where it currently lives."

### 2. Apply the rubric

Walk the questions in order. Stop at the first "yes" — that's the primary destination. Note any secondary destinations.

### 3. Recommend the concrete next step

Be specific. Not "this is brain knowledge", but:

```
This is a reusable pattern → atomic note.
→ workspaces/mr-brainer/knowledge/expo-symbols-vs-vector-icons.md

Next: I can create the note now (run brain-reflect to route it formally),
or you can write it yourself.
```

If the artifact is mixed, give a split:

```
This has two pieces:

1. The decision (Redis vs in-memory cache, picked Redis because…)
   → Brain: decisions/2026-04-18-redis-cache.md (workspace gotam)
   → Use: log-decision skill

2. The actual cache.ts implementation
   → Code: ~/code/smarttravel/src/cache.ts
   → Use: brain-dispatch to implement

Decision goes first; implementation gets dispatched referencing the decision.
```

### 4. Optionally invoke the next skill

If the user agrees, hand off to:

- `log-decision` for decisions
- `brain-reflect` for routing multiple takeaways at once
- `brain-new-feature` for spec/plan
- `brain-dispatch` for implementation

## Red Flags

- Putting **implementation details** (how a function works, line numbers, exact code) into brain — those rot the moment the code changes
- Putting **decisions / rationale** into commit messages alone — they get buried; future agents won't find them
- Refusing to split a mixed artifact — almost everything has both a brain piece and a code piece
- Treating "I'll write it down later" as a routing answer — that's a way to lose it
- Using the brain as a code-mirror (don't paste source files into brain knowledge notes)

## Notes

- The brain is for **decisions, knowledge, intent, and state**. The code repo is for **implementation, config, tests, and runtime artifacts**. When in doubt: brain holds *why*, code holds *how*.
- Cross-linking is cheap. Always link brain → code (path / commit hash) and code → brain (commit message references decision id or spec path).
- Related: `brain-reflect` (routes multiple items at once), `log-decision`, `brain-new-feature`, `brain-dispatch`.
