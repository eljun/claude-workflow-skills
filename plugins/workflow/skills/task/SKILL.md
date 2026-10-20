---
name: task
description: Use this skill when the user wants to plan a new feature, bug fix, enhancement, documentation update, or chore before implementation. Creates or updates a reviewed task document in docs/task/ and updates TASKS.md, then stops for human approval.
---

# task - Planning Stage

## Invariants

- Create or update the task document only.
- Do not implement code.
- Do not start automation from planning.
- Stop after updating `TASKS.md` and tell the user the task is ready for review.
- After adding the row to `Planned`, re-read `TASKS.md` to confirm the row is present before handoff.
- Never change the scope of a task that is already claimed or `In Progress` (a worker branch, an open PR, or the tracker says so). Put new findings in a **new** task that references it, and tell the user to message the agent working on the original. Editing a claimed doc does not reach an agent that already read it.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `task`.

## Workflow

1. Read `AGENTS.md`.
2. Read `LEARNINGS.md` if it exists.
3. Clarify the request only where missing information changes scope, risk, or acceptance criteria.
4. Perform targeted codebase research for files directly relevant to the task.
5. **Coordination check.** List the files this task will change, then read the other `Planned` and `In Progress` task docs (and open worker branches or PRs, if the project uses them). Record every overlap in `Coordination`, naming the shared file and the other task, with an ordering rule ("land after {ID}") when two tasks edit the same file.
6. **Open decisions.** Anything only the user can decide goes in `Open Decisions`, marked `BLOCKING` (implementation must not start) or `NON-BLOCKING` (a default is stated). Don't bury a decision in prose.
7. **Operator steps.** If merging the change does not by itself apply it (migrations, deploys of separately deployed components, config, data backfills), list those steps in `Operator Steps`. For deploys, name every affected deployable, including those that only **import** a changed shared module, and say how you found them.
8. Choose a recommended execution tier.
9. Create or update `docs/task/{ID}-{task-name}.md` from the template (see Templates).
10. Add or update the task in `TASKS.md` under `Planned`.
11. Stop for user review and approval.

## Templates

Use the project's own `docs/templates/task-document.md` and `docs/templates/tasks-tracking.md` when they exist. Otherwise use the copies bundled with this skill in `templates/` (next to this file). If the project's existing task docs follow a different shape, match the project, but keep the required sections below.

## Execution Tier Guidance

Use `Recommended Tier` in the task document:

| Tier | Use When |
|------|----------|
| `fast` | Small, low-risk changes that follow existing patterns. |
| `balanced` | Normal feature work, moderate refactors, or changes across several files. |
| `deep` | Architecture, security, data model, migration, high-risk refactor, or ambiguous requirements. |

The tier is guidance for downstream agents. Runtime adapters define how, or whether, tiers map to models or reasoning settings.

## Task Document Requirements

The task document must include:

- clear overview and requirements
- explicit out-of-scope or must-not-change boundaries
- proposed file changes
- focused code context for files the implementation stage should read
- implementation steps
- acceptance criteria
- verification commands or approach
- `Coordination` (overlapping tasks and files, or "None found")
- `Open Decisions` (each `BLOCKING` or `NON-BLOCKING`, or "None")
- `Operator Steps` (or "None: merge is enough")
- compatibility touchpoints for packaging, docs, adapters, or install surface when relevant

**Acceptance criteria are checks, not prose.** Each one names the command, query, API call or browser step that verifies it, the expected result, and whether it runs pre-merge or post-deploy. Include at least one **no-regression** check for behaviour that must not change. A criterion with no check that could fail can't be claimed later; if it can't be automated, mark it `manual` and say why.

When the task comes from a live investigation, include the evidence (the query, record IDs, counts, `file:line`) so the implementation agent can re-verify it instead of trusting prose.

Embed concise code excerpts or notes in `Code Context`; do not paste entire unrelated files.

## TASKS.md Update

If `TASKS.md` does not exist, create it from the tracker template (see Templates).

If it exists, **match its existing sections and columns**, even if they differ from the template. Add the task to `Planned` (or the project's equivalent), for example:

```markdown
| {ID} | {Task} | {Priority} | {Type} | docs/task/{ID}-{task-name}.md | {Date} |
```

If the task has a `BLOCKING` open decision, say so in the row ("⚠️ decision D1 needed before implement").

## Handoff

End with:

```text
Task planned: docs/task/{ID}-{task-name}.md
Status: awaiting user review
Blocking decisions: {list or "None"}
Coordinates with: {task IDs or "None"}
Next stage after approval: implement {ID}
```
