---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## How to Access Skills

**In Claude Code:** Use the `Skill` tool. When you invoke a skill, its content is loaded and presented to you—follow it directly. Never use the Read tool on skill files.

**In other environments:** Check your platform's documentation for how skills are loaded.

# Using Skills

## The Rule

**Invoke relevant or requested skills BEFORE any response or action.** Even a 1% chance a skill might apply means that you should invoke the skill to check. If an invoked skill turns out to be wrong for the situation, you don't need to use it.

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## Red Flags

These thoughts mean STOP—you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "I can check git/files quickly" | Files lack conversation context. Check for skills. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Skills prevent this. |
| "I know what that means" | Knowing the concept ≠ using the skill. Invoke it. |

## Skill Priority

When multiple skills could apply, use this order:

1. **Process skills first** (brainstorming, writing-plans, debugging) - these determine HOW to approach the task
2. **Role skills second** - consult domain experts for specialized decisions:
   - Architecture/design → `it-architect`
   - Frontend work → `frontend-engineer`
   - API/backend work → `backend-engineer`
   - Infrastructure/CI/CD → `devops-engineer`
   - Data modeling/queries → `database-architect`
   - ML/AI features → `ai-ml-engineer`
   - Test strategy → `qa-engineer`
   - Production issues → `sre-engineer`
   - Security concerns → `security-engineer`
   - PII/GDPR/ISO 27001 → `compliance-officer`
   - Code review/tech debt → `tech-lead`
   - Prioritization/MVP → `product-engineer`
3. **Implementation skills third** (frontend-design, mcp-builder) - these guide execution

"Let's build X" → brainstorming first, then role skills for domain guidance, then implementation.
"Fix this bug" → debugging first, then relevant role skill (e.g., `sre-engineer` for prod, `database-architect` for DB).
"Handle user data" → `compliance-officer` MUST be invoked for any PII/personal data work.

## Session Awareness

If a session is active (a session log exists at `docs/sessions/YYYY-MM/YYYY-MM-DD-<nick>.md`):
- **Log significant decisions, completions, and issues** into the session log
- Process skills (brainstorming, writing-plans, subagent-driven-development) do this automatically
- For ad-hoc work outside those flows, append a brief entry yourself

Session commands:
- `/zaciname` — start a work session (creates log, gathers context, sets daily plan)
- `/koncime` — end a session (summarize, commit, push, optional Slack notification)

## Compliance Gate

**Automatic trigger** — invoke `compliance-officer` skill when any of these apply:
- Task involves user/personal data (names, emails, IDs, addresses, health, financial)
- New data storage, collection, or processing is being designed
- Third-party integrations that receive or send personal data
- Logging, analytics, or tracking that could capture PII
- Data export, migration, or deletion features

This is not optional. GDPR fines are real. When in doubt, invoke it.

## Skill Types

**Rigid** (TDD, debugging): Follow exactly. Don't adapt away discipline.

**Flexible** (patterns, role skills): Adapt principles to context.

The skill itself tells you which.

## User Instructions

Instructions say WHAT, not HOW. "Add X" or "Fix Y" doesn't mean skip workflows.
