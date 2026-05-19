---
name: ship
description: Use this skill after documentation is complete and the task is approved. Runs pre-ship checks, prepares a branch or pull request, updates TASKS.md, and leaves release timing to the release stage.
---

# ship - Pull Request Stage

## Invariants

- Always advance the task to `Ready To Ship` in `TASKS.md` and re-read the tracker to confirm the row moved before handoff. A stale tracker blocks downstream and headless sessions.
- Self-heal when intermediate stages were skipped: never abort because the task is not in `Approved`. Detect the current section, record a `Skipped Stages` list, and disclose it in the PR body and handoff text.
- Run available pre-ship checks before creating a pull request.
- Do not auto-merge.
- Keep released-version work for the `release` stage.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `ship`.

## Workflow

1. Read `AGENTS.md`.
2. Resolve the task ID and read the task document.
3. Compute `Skipped Stages` by checking for durable artifacts (see table below). Do **not** infer skipped stages from the current `TASKS.md` section — the tracker is exactly what goes stale, so trusting it would re-introduce the bug.
4. Pre-ship checks are still useful; missing artifacts do not block ship, only get disclosed.
5. Run pre-ship checks documented by the project or task.
6. Review changed files for unintended edits and secrets.
7. Create or reuse an appropriate branch.
8. Create a pull request with links to the task document, test report, and the `Skipped Stages` disclosure.
9. Move the task to `Ready To Ship` in `TASKS.md`, removing the row from whichever prior section it currently sits in.
10. Re-read `TASKS.md` and verify the task row is now under `Ready To Ship` and absent from prior sections. If the row is not in the expected section, the move was missed — perform the edit now before handoff.

(This is a verification against the model forgetting the step, not against `Edit` failing. The check is cheap and catches the most common cause of stale trackers.)

## Skipped Stages Detection

Detect from durable artifacts, not from tracker section. The tracker may be stale; the artifacts are the ground truth.

| Stage | Counts as "ran" when this artifact exists |
|-------|-------------------------------------------|
| `implement` | task doc contains an `Implementation Notes` section |
| `simplify` | task doc contains a `Quality Gate Notes` section with `Result: PASS` or `FAIL` |
| `test` | `docs/testing/{ID}-{task-name}.md` exists |
| `document` | `docs/learnings/{ID}-{task-name}.md` exists, or feature/guide docs were updated for this task |

A stage with no artifact is "skipped". List skipped stages in the PR body and handoff.

## Pre-Ship Checks

Run available project checks, such as:

```bash
pnpm build
pnpm typecheck
pnpm lint
pnpm test
```

Use the commands that exist in the target project. If a command is unavailable, record that it was skipped and why.

## Pull Request Body

```markdown
## Summary

- {Change summary}

## Verification

- {Command or report}

## Skipped Stages

- {Comma-separated list, or "None"}

## Workflow References

- Task: docs/task/{ID}-{task-name}.md
- Test report: docs/testing/{ID}-{task-name}.md or N/A
- Feature docs: {path or N/A}
```

## Handoff

```text
Ship preparation complete for task {ID}
Task status: Ready To Ship (verified in TASKS.md)
Skipped stages: {list or "None"}
PR: {url}
Next stage after merge: release
```
