---
name: log-adr
description: Use when an architectural / technical / security decision has been made and needs to be recorded as a formal Architecture Decision Record. For light process / business / ops / product decisions, use log-decision instead.
---

# Log ADR

## Overview

Records architectural, technical, or security decisions in the brain repository as full ADRs — with numbered sub-decisions, alternatives rejected, optional supersedes, and an amendments trail. ADRs are the source of truth for cross-cutting technical contracts; light process or business decisions belong in `log-decision`.

## When to Use

Use this skill when:
- An architectural or technical contract is being committed across services / clients (e.g. an auth flow, a data model, a security posture).
- The decision is non-trivial enough that future readers need alternatives, rationale, and explicit out-of-scope.
- The decision will likely be amended over time (add the Amendments section on first amendment).
- **Not for light decisions.** For short process / business / ops / product decisions without an alternatives analysis, use `log-decision`. Both write into `decisions/`; the distinction is structural.

## Locate Brain Repo

Before anything else, find the `brain` repository:

1. Check if the current workspace IS the brain repo (look for `AGENTS.md` + `REPO_KNOWLEDGE.md` in root).
2. If not, check common sibling locations: `../brain`, `../../brain`.
3. If not found, ask the user for the path.

All file operations happen in the brain repo.

## Git Safety Protocol

Before reading or writing in the brain repo:

1. **Verify the branch** — `git -C <brain> branch --show-current` should return `main` (or a feature branch if the user is intentionally working on one). If you find a different branch, ask the user before continuing.
2. **Pull the latest** — `git -C <brain> pull --rebase origin <branch>` so the ADR lands against a fresh tip and the session file is current.
3. **Inspect uncommitted work** — `git -C <brain> status --short`. Today's session file should already be untracked from `session-start`; surprising modified files are not normal — confirm with the user.

## Read Template

Read the template from the brain repo (single source of truth):

```
Read <brain>/templates/log/adr.md
```

If the file is missing, fail loudly with this message and stop:

> `templates/log/adr.md not found in <brain-path>. Either restore from git or update brain.`

Do NOT fall back to an embedded copy — there is no embedded copy.

## Process

Ask the user, one section at a time:

1. **Title** (imperative, descriptive — e.g. "OAuth server-side flow and denormalized provider IDs"). Slug: kebab-case from the first 5 non-stopword tokens of the title; if collision in `decisions/`, append `-2`, `-3`, etc.
2. **Category** — `technical | architecture | security`.
3. **Participants** — list as `Name (Role)`. The `Role` MUST match `graph/entities/people/<id>.yaml`. If unsure, look it up; do not invent titles.
4. **Status** — usually `accepted` for newly-recorded decisions. Use `proposed` if the decision is being captured before final approval.
5. **Decision** — the one-paragraph opener, then the numbered list of interlocking sub-decisions. Ask the user to enumerate each sub-decision; each gets a short name and body.
6. **Context** — why this is being decided now. Reference predecessor ADRs if any.
7. **Rationale** — why this option vs the rejected ones.
8. **Alternatives rejected** — bullet each alternative considered and the reason for rejection. ADRs without an Alternatives section are incomplete.
9. **Consequences** — fill the sub-blocks: New services / New endpoints / Schema changes (migration names) / Deprecations / Open follow-ups. Omit any sub-block that is N/A.
10. **Supersedes** (optional) — if this ADR overrides specific sections of a predecessor, list them. Otherwise omit the section entirely.
11. **Out of scope** — what is explicitly NOT being decided here.
12. **Next steps** — first concrete implementation steps.
13. **References** — predecessor ADRs, related docs.

Do NOT add an Amendments section on first write — it is added later when the ADR is amended.

## Write the File

Write to `<brain>/decisions/YYYY-MM-DD-<slug>.md`. Use today's date.

## After Creation

- Add a reference to today's session diary under "## During the day":

```markdown
**[HH:MM] ADR:** {short title}
→ decisions/YYYY-MM-DD-<slug>.md
```

- **Do NOT commit.** See "Commit Policy" below.

## Commit Policy

This skill does NOT commit. It writes the ADR file and appends to today's session diary, but stages nothing.

ADRs are typically high-stakes — discuss with the team before merging if needed. The `session-end` skill bundles the day's writes into one commit on `main`.

If the ADR is so consequential it needs to land immediately (rare), tell the user explicitly and let them decide whether to commit by hand. Do not auto-commit from this skill.

## Amendments (post-write)

When an ADR needs to be amended later:

1. Open the ADR.
2. Add `**Amended:** YYYY-MM-DD — short summary` line after `**Status:**` in the frontmatter.
3. Make the in-section additions, marking each new paragraph with `(added YYYY-MM-DD)` inline.
4. At the bottom (before `## References`), add or extend the `## Amendments` section with a dated entry summarizing what changed and why.
5. Do NOT rewrite existing sections — amendments are additive.

## Red Flags

- Writing an ADR without enumerating alternatives. ADRs without Alternatives rejected are decisions without context — use `log-decision` instead if you can't articulate alternatives.
- Inventing a participant Role that does not match `graph/entities/people/<id>.yaml`. Reconcile against the entity, always.
- Committing from this skill. `session-end` bundles.
- Skipping the Git Safety Protocol.
- Writing one ADR that bundles multiple unrelated decisions. Split into separate ADRs.
- Modifying existing sections during an amendment. Amendments are additive — add a new dated paragraph, don't rewrite history.

## Notes

- ADRs are immutable in spirit. Amendments are additive. If a decision is fully reversed, write a new ADR that supersedes the old one.
- The `/brain-log-adr` slash command (if it exists) is a thin wrapper that loads this skill — they are not duplicates. This file is the spec.
