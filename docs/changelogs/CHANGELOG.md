# Changelog

## [v1.7.0] - 2026-10-20

Lessons from running the workflow with several parallel agents on one integration branch.

### New Features

- **Coordination check (`task`):** a task doc now lists the other `Planned` / `In Progress` tasks that change the same files, with an ordering rule. Three tasks that edited one shared module had no idea about each other.
- **Open decisions as stop points (`task`, `implement`):** user decisions go in an `Open Decisions` table, marked `BLOCKING` or `NON-BLOCKING`. `implement` refuses to start while a `BLOCKING` one is open; previously an agent claimed a task whose doc said "needs a decision" in prose.
- **Operator steps (`task`, `implement`, `ship`):** every task doc and PR states what merging alone doesn't apply (migrations, deploys, config). `ship` **computes** deploys from the diff, including deployables that only import a changed shared module, and reports whether each is already live when that can be checked read-only. A hand-written list had missed a worker that ran the changed code.
- **No-task-doc path (`simplify`):** small owner-requested changes get the same quality gate, recorded in the PR body.
- **Live verification (`test`):** distinguishes "tests pass" from "the change is live", requires output-based evidence in the running system, and marks deploy-dependent checks as Deferred instead of PASS.

- **Validation loops** ("never claim what you didn't observe"): acceptance criteria are now a table of **checks with expected results** (pre-merge or post-deploy), including at least one no-regression check; `implement` captures a **baseline** first, runs the narrowest check after each step, must change something between retries, and records failed attempts; `simplify` flags criteria with no failing check and tests that only assert source text; `test` compares against the baseline and grades each result's **evidence strength**; `ship` hands over the **post-deploy checks**; `release` keeps a task out of `Shipped` until they've been run, or the user waives them.

### Improvements

- **Templates bundled with the skills:** `task-document.md` and `tasks-tracking.md` moved to `skills/task/templates/`, and `test-report.md` to `skills/test/templates/`. `npx skills add` copies sibling files (verified by installing into a throwaway project), so the templates now arrive in target projects; before, `task` pointed at a `docs/templates/` path that only existed in this repo. A project's own `docs/templates/` still takes precedence.
- **A claimed task's scope is frozen** (`AGENTS.md`, `task`): post-claim findings become a new task, because edits to a doc an agent already read don't reach it.
- `implement` re-reads the task doc from the integration branch at start and before handoff, and follows a project `AGENTS.md` claim/tracker convention when it differs from the default moves.
- `task` matches an existing `TASKS.md` layout instead of assuming the template's sections, and records evidence (queries, IDs, `file:line`) for tasks that come from live investigation.

## [v1.6.0] - 2026-05-19

### New Features

- `ship` self-heals when intermediate stages (`simplify`, `test`, `document`) were skipped. It always advances the task row to `Ready To Ship` regardless of starting section, and discloses which stages ran in a new `Skipped Stages` line in the PR body and handoff text.
- `ship` detects skipped stages from durable artifacts (`Implementation Notes` and `Quality Gate Notes` sections in the task doc, `docs/testing/{ID}-*.md`, `docs/learnings/{ID}-*.md`), not from the tracker section — so a stale tracker cannot misreport which stages actually ran.

### Improvements

- `TASKS.md` tracker discipline promoted to a top-level rule in `AGENTS.md`. Every status-moving stage (`task`, `implement`, `test`, `document`, `ship`, `release`) now has the tracker update listed as an invariant and a final "re-read and verify" step before handoff, addressing stale rows that previously blocked downstream headless sessions.
- `simplify` acts as a cross-stage guard: if the task doc has `Implementation Notes` but `TASKS.md` still shows the row in `Planned` or `In Progress`, it reconciles the row to `Testing` rather than propagating the staleness. If `Implementation Notes` is absent, it does not reconcile.
- `document` is now idempotent for tracker moves: it reconciles the row to `Approved` from any earlier section if `test` missed the update, otherwise it is a no-op.

### Documentation

- README explicitly documents that stages between `implement` and `ship` are skippable and that `ship` self-heals — useful for headless sessions running small tasks that do not need the full chain.
